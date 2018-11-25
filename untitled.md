# BFT consensus

## Design principles

Celerium's consensus protocol is fairly similar to a simplified version of that of Tendermint. One notable difference is that direct communication is used instead of a random gossip protocol due to the desire for lower latency over the Internet --- using a gossip protocol generally means multiplying the latency by a large constant unless highly sophisticated and possibly fragile relay protocols are used. Furthermore, unlike PBFT and similar systems, all communication happens to and from the proposer in each round. This does place the network communication overhead more heavily on the proposer, but not by much, and in return it actually significantly reduces the number of messages sent in total during a round.

{% hint style="info" %}
In the future, tree-shaped communication patterns, combined with something like BLS signatures, may be used. The adaptation is fairly simple, and trees can be made Byzantine-resistant using the same sort of randomization technique as proposer selection. This would allow extremely massive scalability of coordinator nodes.
{% endhint %}

The consensus protocol relies on partial synchronization for liveness, and is round based, where each round consists of three steps: _Propose_, _Prevote_, and _Precommit_, with a special step _Commit_. Each of the three steps takes a third of the time allocated to the round; the time per round starts at 5 seconds and increase by 50% for every subsequent round, with at most a 1-minute round time.

## Algorithm

### Propose phase

Each round has a designated _proposer_, chosen by a deterministic pseudo-randomized method. More precisely, for each round, we calculate a _seed_: the seed of round 0 is the previous block hash, while each seed is the hash of the previous seed. Then, for each round, we divide the space between 0 to 2^256-1 into portions in proportion to the relative voting power of each node, ordered from biggest to smallest voting power. The seed is then interpreted as a big-endian number and whoever's section it lands on gets to be the proposer for that round. 

At the beginning of the Propose phase, everybody downloads a proposal from the proposer:

```yaml
Round: roundNum
Proposal: proposedBlock
ProofOfLock: TODO!!!
Signature: Sign(Hash(roundNum, proposedBlock, "propose"))
```

If the proposer is locked on a block from some prior round, it proposes the locked block and includes a proof-of-lock in the proposal. Otherwise, it builds a block out of its mempool and proposes it instead.

For other coordinators, the proposal is accepted if it is both valid, and either not conflict with any locked blocks or contain a proof of lock that will unlock any already-locked block. Otherwise, the proposal is rejected, and **the coordinator does not do anything for the rest of the phases** --- all of them assume a good `proposedBlock` has been downloaded from the proposer.

{% hint style="info" %}
**What about proof of lock?**
{% endhint %}

### Prevote phase

The next step is _Prevote_. In this phase, everybody uploads prevotes to the proposer:

```yaml
Round: roundNum
Prevote: Hash(proposedBlock)
Signature: Sign(Hash(roundNum, Hash(proposedBlock), "prevote"))
```

### Precommit phase

The next step is _Precommit_. 

At the beginning of this step, each coordinator downloads a package of all the prevote messages from the proposer.

If the coordinator sees more than 2/3 of prevotes for the proposer's block, it will lock onto that block and release any prior locks, packaging the prevotes into a _proof-of-lock_. It will then upload a precommit for that block. Otherwise it doesn't do anything.

At the end of _Precommit_, the coordinator downloads all the precommits from the proposer. If the node has received more than 2/3 of the precommits, it jumps into _Commit_; otherwise it loops back to _Propose_ for the next round.

### Commit phase

The final, special phase is _Commit_. Once a node enters this phase, it knows exactly what the next block is going to be. proposedBlock is added to the block database, with the collection of precommit messages as a _proof of commit_,  and the next BFT run is scheduled for 30 seconds later. 

To ensure liveness, every coordinator also probes a random other coordinator every second, trying to "catch up" through the usual fastsync mechanism, aborting the BFT process if fastsync instead works.



