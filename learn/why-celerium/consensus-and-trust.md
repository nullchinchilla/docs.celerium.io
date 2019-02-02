# Consensus and trust

We start our exploration of Celerium's features by looking at how nodes in Celerium come to agreement on the status of the network --- decentralized **consensus**, the foundation of any public blockchain's security. Celerium uses a variation of **bonded proof-of-stake** found in systems such as Tendermint, augmented with "auditors" which further decentralize trust.

## Oligarchy with a free press

Participants in Celerium are divided into three categories by their roles:

* **Stakeholders** record transactions into new blocks and confirm them using a Byzantine fault-tolerant consensus algorithm between themselves. They communicate with each other through a broadcast protocol which other nodes in the network never participate in. The set of stakeholders is determined on the blockchain by "staking" a cryptoasset; the staking process will be discussed in TODO. Stakeholders correspond to miners or validators in other systems.
* **Auditors** download newly created blocks from the stakeholders and gossip them between themselves, while storing a local copy of the blockchain. Anybody can join the network as a auditor, and they verify the new blocks decided by the stakeholders and check that the stakeholders never equivocate on the content of a given block height. Importantly, Auditors roughly correspond to full nodes in other systems, although they have a more important role in Celerium's security.
* **Clients** are lightweight participants that query the network of auditors to access specific information in the blockchain, yet do not trust any particular auditor.

From this overview we can already see that the trust model of Celerium differs significantly both from that of traditional proof-of-work public blockchains like Bitcoin and from that of typical private blockchains, and is one of its major innovations. Celerium's trust can be summarized succinctly as an **"oligarchy with a free press"**, in contrast to private blockchains' "closed oligarchy/monarchy" or traditional public blockchains' "direct democracy".

## Stakeholders: the oligarchy

### Bonded proof-of-stake

In Celerium, a Byzantine-resistant fault tolerant algorithm, similar to that used in Tendermint, is used between the stakeholders to establish consensus on the content of the blockchain. The stakeholders form an "oligarchy", as the majority of participants are not stakeholders, yet they decide the authoritative state of the network. The list of stakeholders is known to all participants in the network, dynamically determined through ownership of a scarce staking token, the **lent**.

{% hint style="info" %}
**Lent** comes from _lentus_, the Latin word for "slow", a reference to how lents must be frozen by stakeholders to establish voting rights.
{% endhint %}

Lents are an on-chain cryptocurrency traded freely alongside cels, the main cryptocurrency of Celerium, with a fixed supply of 1 million lents. Holders of lents can **stake** them, locking them up for a fixed period of time \(500,000 blocks, or approximately 6 months\) to obtain voting rights in the consensus algorithm in proportion to the amount of lents staked. The central security assumption Celerium uses is that **at least 2/3 of the staked lents are in the hands of honest stakeholders**, which derives from a fundamental property of asynchronous Byzantine-fault-tolerant consensus.

### Rewards and slashing

How do we incentivize stakeholders to behave honestly? We use a carrot-and-stick approach commonly found in systems using bonded proof of stake: honest stakeholders earn **rewards** over time in proportion to the amount of lents they stake, while misbehaving stakeholders can have their entire stake **slashed** given evidence of misbehavior or **kicked** from being a stakeholder.

Rewards to stakeholders come from two sources: lent demurrage and transaction fees. Lent demurrage is essentially a tax on lents that are not staked: unstaked lents are confiscated from holders and credited to a **reward pool** at a rate of about 5% a year. When stakeholders' 6-month terms expire, they can then withdraw a portion of the reward pool proportional to the lents they staked; this on average transfers the "taxes" to the stakeholders proportionally to their stakes. Lent demurrage also serves a double purpose of encouraging staking of lents rather than holding them passively.

Transaction fees, denominated in cels, are imposed on every transaction on the network, and these fees also go entirely to the stakeholders as an additional source of income. The details of exactly how much transaction fees are charged and exactly how they are paid out to stakeholders are a little complicated, as Celerium uses an idiosyncratic transaction fee model, different from that of other cryptocurrencies, in order to make it easier for clients to calculate appropriate fees. We discuss transaction fees more in the ~~standalone document on fees~~; for now a reasonable approximation is that stakeholders receive fees in proportion to the volume of transactions on the network.

In addition to rewards that incentivize contributing to the network as a stakeholder, we disincentivize misbehavior by two separate mechanisms: slashing and kicking.

Slashing is used to punish cryptographically provable misbehavior. The last step of our Byzantine-fault-tolerant consensus protocol has all stakeholders **commit** to a particular block by signing it cryptographically; the protocol is designed such that all honest stakeholders will commit the same block, as long as more than 2/3 of the stakeholders are honest. Thus, we have two **slashing conditions** which leave cryptographic proof that a certain stakeholder is dishonest:

* **Equivocation**, where a stakeholder signs two different blocks with the same block height
* **Invalid block**, where a stakeholder signs an invalid block

In either of these cases, anybody can submit cryptographic evidence \(two conflicting signatures, or a signature on an invalid block\) as a specially-formatted transaction on the blockchain. This **slashing transaction** removes the offending stakeholder, moving all of the staked funds into the reward pool and leaving the offender with nothing. We put slashed funds into the reward pool to incentivize stakeholders --- who already run have the resources to do so --- to audit the behavior of their peers, rather than waiting for random altruistic strangers to do so.

Kicking punishes cryptographically unprovable misbehavior, such as having a very unreliable network, being offline, proposing empty blocks, etc. ~~The exact procedure is rather involved to avoid abuse~~, but the gist of it is that stakeholders holding more than 2/3 of the lents can vote to remove a stakeholder from the list, forcing it to unstake without any reward. This may sound like an easy way for a majority of stakeholders to cartelize and block "competition", but note that such a malicious majority can already do so by simply censoring all staking transactions, so kicking should not pose any additional problem.

### Why bonded proof-of-stake?



## Auditors: the free press

### Making failure catastrophic

The "free press" in Celerium consists of **auditors**. Auditors are "full nodes" in usual terminology, replicating the entire blockchain --- each block of which is published and cryptographically signed by the stakeholders --- while checking that each transaction is valid with respect to previous ones in the blockchain. They form a random **gossip** network among themselves, similar to that used by Bitcoin full nodes, through which information about new blocks is disseminated. This gossip network reduces load on the stakeholders and makes it difficult for malicious networks to censor the blockchain, since as long as some auditors can connect to the stakeholders and the auditors form a connected graph, new blocks will quickly be visible to every auditor.

The more important role of auditors, though, is to _make consensus failure catastrophic,_ playing a crucial role in keeping the oligarchy of coordinators honest. Auditors utilize their position as relayers of new blocks to continually monitor for evidence that the consensus of the coordinators is corrupt --- for example, invalid blocks or two different blocks at the same height signed by a quorum of coordinators would be proof that the coordinators are no longer trustworthy. Notably, these proofs are cryptographically undeniable, as invalid states signed by a quorum never have "innocent explanations". Any auditor that sees such proof immediately broadcasts it to all auditors it knows in the gossip network, while permanently activating a \`\`kill switch'' and refusing to operate normally. Thus, the presence of auditors ensures that even if an adversary somehow gains control over a quorum of coordinators, any attempt at forking or appending invalid transactions to the blockchain would cause a catastrophic collapse of the entire network instead of silent, arbitrary corruption of the blockchain.

```text
Why make failure catastrophic? Disabling the network prevents damage and forces
manual recovery through a hard fork. If 2/3 of stakeholders are bad the network
is pretty much useless anyway. This also makes attacks very very risky and
impossible to conceal.
```

### Why fail hard?

## Clients: thin yet fully secure

```text
Current thin clients have bad tradeoffs
- Not so thin
- Often trusts too much

Extremely thin by keeping track of only latest header

Fast, safe stakeholder tracking with essentially no "weak subjectivity"
```



