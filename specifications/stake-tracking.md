# Stake tracking

## Overview

Staking information is tracked according to the following principles:

* Lents trade freely on the chain with no limits.
* Lents can be staked for a fixed, long period of time with special staking transactions. 
* Voting power stays constant for medium-length periods of time to reduce "catching-up" time for thin clients.

## How are stakes managed?

### Stake epochs

Time is divided into _stake epochs_ lasting 500,000 blocks each. The first epoch is from block 0 to 499,999; the second from 500,000 to 999,999, etc. Thus, each epoch is about half a year in length.

At the start of each epoch, the effects of all the stake-related transactions in the previous epoch are applied for the next epoch. A staking transaction estabilishes voting rights for exactly one epoch, and imposes a waiting period of at least another epoch before the funds are returned, during which slashing may happen. This is done by simply enforcing that a stake deposited in block $$n$$ stays deposited until block $$n+1000000$$.

The state of the stakes within an epoch is described by a _stake document_.

### Stake documents

Each epoch has a stake document associated with it, summarizing the detailed information of stake bonds. The stake document is a Merkle binary radix-tree mapping between `SigningKey` and the following structure, RLP-encoded:

```go
// StakeDocEntry is an entry in a stake document.
type StakeDocEntry struct {
    SigningKey  []byte // first byte is algorithm, rest is PK, not public key hash
    StartEpoch  uint
    UnlockEpoch uint // epoch after the last voting epoch
    Staked      uint // lents staked
}
```

### Block header values

The block header includes hashes of a trie `StakeDoc`. It's a mapping from staking transaction ID to `StakeDocEntry`. When a new epoch is started, the trie is cleaned of all entries where `UnlockEpoch` is in the past.

Both of these documents contain information about stakeholders who no longer have voting power, but still have deposited stakes. This is in accordance with the overall principle that the block header commits to all the data that's needed to validate new blocks and transactions.

### Staking transaction

All staking transactions must:

* Have kind `KindStake`
* First output:
  * Has cointype `'L'`
  * Has value at least 1000
* Data:
  * Valid `StakeDocEntry`
  * `StartEpoch` greater than current epoch
  * `UnlockEpoch` greater than `StartEpoch`
  * `Staked` exactly equal to value of first output

### Thin-client catching up

Thin clients must be able to securely derive the stake document of the current epoch in order to properly validate block signatures. This is done by the client requesting the following information:

* All the pre-epoch-start block headers since the last one the client knows about, together with their quorum signatures
* The stake documents of all the epochs the client missed

The client then verifies the information by incrementally checking the validity of the pre-epoch-start block header using a stake document it has already verified, using the header to verify another stake document, etc.

## Design rationale

### Why such long epochs?

The long epochs batch changes in the stake document into a much smaller number of stake document updates: twice a year. This means that thin clients can catch up much faster than if they must track every change in stake since forever. Even if a client is offline for 10 years, it would only need to download 20 stake documents. Assuming the worst case \(1000 stakeholders\), this is just 960 KB of data, most likely much smaller after compression.

### Why fixed bond periods?

Most bonded proof-of-stake systems, like Casper, allow stakeholders to withdraw any time, but subject them to a waiting time during withdrawal. This is essentially equivalent to our system, considering that stakeholders can use a strategy analogous to [CD laddering](https://en.wikipedia.org/wiki/Laddering) to keep stake deposited indefinitely.

The advantage of fixed bond periods is simply ease of implementation and understanding, especially considering we use a UTXO-based model and wish to reduce special cases like separate withdrawal initiation and completion transactions. It also nicely unifies with the epoch length.

### Weak subjectivity?

Like all proof-of-stake systems, we suffer from a security problem known as **weak subjectivity**. Basically, clients that have been off for a very long time \(in our case, more than two epochs\) cannot defend against attacks by a quorum of _past_ stakeholders, as they can collude to sign a series of stake documents inconsistent with the main chain, and more importantly, they cannot be punished by slashing or consensus alarm since on the main chain their funds have already been unstaked, and even if the clients are not eclipsed and can see the main chain, they have no way of deciding.

Vitalik Buterin already has a [pretty good blog post](https://blog.ethereum.org/2014/11/25/proof-stake-learned-love-weak-subjectivity/) arguing that this is not as bad as it sounds in the case of Casper. Given Celerium's properties, though, there are some even more important arguments against the idea that weak subjectivity is really bad:

* Under a security model of "passive altruism", as used in Algorand etc, where due to transaction costs in planning an attack, most stakeholders stay honest unless doing so significantly damages their self-interest, weak subjectivity for Celerium disappears, as old-epoch quorums can then be trusted not to double-sign. This "passive altruism" can be as simple as deleting keys when bonds are redeemed.
* In a security model where clients can get eclipsed by a bad network, proof of work suffers from a similar problem: an arbitrarily bad, arbitrarily low-difficulty fork can fool any client. The main assumption behind the "subjectivities", that the client can see all previous messages and must decide the correct fork, does not work when the network is adversarial. Celerium, however, assumes that clients' networks may be entirely adversarial for an unbounded time, in which case Nakamoto-style consensus completely fails, while the stake-document chain still works perfectly.

### Can slashed stakeholders vote?

Slashed stakeholders can, unfortunately, vote until the end of an epoch. This is not really an problem, since under normal circumstances slashed stakeholders cannot possibly be too numerous to cause security failure \(in that case a consensus alarm will happen and the whole issue is moot\). The only problem is that they might collect rewards, but such rewards will only be a small percentage of the stake they lost.

