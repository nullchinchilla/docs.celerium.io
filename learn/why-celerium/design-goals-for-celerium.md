# Design goals for Celerium

## A universal trust anchor

Instead of a specific decentralized application or a "kitchen-sink" runtime environment, Celerium provides a **universal trust anchor** to a large class of decentralized apps. We want Celerium to sit at the bottom of diverse and evolving protocol stacks, provide the magic ingredient of decentralized trust and consensus --- but not much else.

Towards this end, Celerium introduces a novel design combining the classical model of "UTXO-based" cryptocurrencies like Bitcoin --- namely, a globally consistent grow-only DAG of transactions --- with numerous innovations in areas such as consensus and cryptocurrency issuance to maximize robustness and performance. Before we take a dive into the details of Celerium's architecture though, let's first examine some key goals and non-goals we want to accomplish.

## Goals 

Our overarching goal of being a foundational root of trust leads our design to follow the following principles:

### Simple abstractions

Celerium should present simple, "non-leaking" abstractions. Average programmers without extensive experience in blockchain development should be able to easily understand the features and behavior of the Celerium blockchain. As far as possible, Celerium should never force users to consider subtle edge cases --- proof-of-work blockchains' unintuitive behavior in the presence of pathological networks \("confirmations", forks, reorganizations\) is an excellent example of what to avoid.

### Stable protocol

Though an initial period of rapid evolution is inevitable and indeed desirable, once mature, Celerium's internals should stay the same over time as much as possible. Celerium today should be the same system as Celerium tomorrow, allowing applications with very long development cycles, hardware embedded applications, etc. Moreover, protocol stability also means avoiding contentious consensus-breaking updates.

### High performance

Celerium's performance, measured both in transaction throughput and scalability in the number of fully secure clients, must be much higher than existing public blockchains. Decentralized apps can't take over the world if their security hinges on debilitatingly poor performance.

### Application neutrality

Celerium should not attempt to prevent or censor any categories of applications, and it does not really have an "intended use".

### Robust decentralization

The ideal public blockchain must simulate a universally trusted intermediary; thus, decentralization is a must. Celerium is designed to decentralize trust across as large a population of stakeholders as possible, with no unaccountable third parties, including network operators, able to subvert its security guarantees.

## Non-goals

However, our objective also leads us to explicity exclude as \emph{non-goals} many commonly sought-after properties:

### Evolving feature-set

Many blockchains envision constant protocol evolution, adding consensus-critical features over time to support new features. As we've mentioned before \mytodo{where}, however, this introduces difficult out-of-band coordination problems which easily lead to de-facto centralization, contentious forks, and "governance capture" by special interests. Celerium aims to eventually stabilize on a well-specified, "eternal" protocol, with consensus-breaking changes made only in exceptional, non-controversial circumstances, such as to fix critical design flaws.

### Cryptoasset value growth

Celerium's cryptocurrency, the cel, is designed to have very low price volatility, explicitly avoiding large price increases even with greatly increased adoption. A speculative asset exciting to "HODL" simply cannot be a good medium of exchange or store of value.

