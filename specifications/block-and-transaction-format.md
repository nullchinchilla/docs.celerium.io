# Block and transaction format

## Overview

This document is a full specification of all the data structures used in Celerium.

We represent structures in Go syntax, which should be fairly straightforward. Serialization is done with Ethereum's RLP syntax, with the Go-to-RLP mapping used by geth's `rlp` package.

## Block headers

Block headers in Celerium are designed to maximize the utility of ultrathin clients who only keep track of a reasonably recent, _single_ block header.

Each Celerium block header is as follows:

```text
type BlockHeader struct {
    PrevHash   celcrypt.Bytes32
    HistoryRH  celcrypt.Bytes32
    StateRH    celcrypt.Bytes32
    TxHashesRH celcrypt.Bytes32
    StakeH     celcrypt.Bytes32
    FeePoolC   uint
    FeePoolL   uint
    Height     uint
    Timestamp  uint
}
```

The parent hash points to the previous block header. The history tree maps _little-endian_ block numbers to block header hashes.

### State tree

The state tree of each block is an MBPT storing mappings generally related to state useful for light clients. The purpose of lumping it into one tree is to support future features while keeping backwards compatibility with clients \(though not auditors\). This _currently_ is the following:

* A mapping from `["utxo.value", txInput]`, for every UTXO represented as a TxInput structure, to a TxOutput structure. Both these structures are defined in the subsequent section on transactions.d
* A mapping from `["utxo.coincount", consHash]` for every constraint with active coins \(the double hashing is to prevent DoS vectors from really huge constraints\), to the \(RLP-encoded\) number of unspent UTXOs with that constraint. Of course, constraints with no active coin would not have an entry here.

These mappings allow the basic function of a "SPV" client to be entirely trust-free, unlike the situation in Bitcoin where hiding coins or misrepresenting spent status of coins is possible.

### Transaction tree

The transaction tree is an MBPT mapping transaction hashes of new transactions in the block to the transaction itself.

### Block number

The block number is included, unlike, say, Bitcoin, for a client to quickly check whether or not its copy of the latest block header is outdated or not.

### Transaction time

The timestamp is simply whatever happens to be the clock of the successful proposer of the block, and is not checked when blocks are validated. Later blocks are not even guaranteed to have higher timestamps than earlier blocks. Thus, timestamps should never be trusted alone; instead, for trusted timestamping applications using the median of, say, the surrounding 100 blocks is strongly recommended.

## Transactions

### General format

The general format of a transaction is as follows:

```go
type Transaction struct {
    Kind    uint8
    Inputs  []TxInput
    Outputs []TxOutput
    Fee     uint
    Data    []byte
    Sigs    []TxSig
}

// TxInput is a transaction input.
type TxInput struct {
    TxHash celcrypt.Bytes32
    Index  uint
}

// TxOutput is a transaction output.
type TxOutput struct {
    Constraint ConsScript
    Value      uint
    CoinType   []byte
}

// TxSig is an algorithm-agnostic signature structure.
type TxSig struct {
    Algorithm byte
    PubKey    []byte
    Signature []byte
}
```

As in all classic UTXO-based blockchains, each transaction is at its core collection of inputs and outputs, each input spending a previous transaction's output. Every output includes a **constraint script**, which constrains what sort of transaction may spend it --- for example, by requiring the transaction spending it to have a signature from a particular public key. An output also indicates the **coin type**; it's either "C" \(for cels\) or "L" \(for lents\) or a custom user-created token, as we will describe later.

Unlike Bitcoin, however, we do not supply input scripts to satisfy individual output constraints; the entire transaction is fed as the input to each output constraint it attempts to spend. Another difference is that the **fee**, which is always denominated in cels, is explicit; this mostly simplifies implementation.

The **kind** of a transaction determines the interpretation of the other fields. The default kind, used in all usual fund-transferring contexts, is `0x00`; other kinds are used for special transactions, such as minting new currency and staking.

Some notes:

* The signature part can be malleable and thus is removed when txids are computed. This is for the same rationale as in SegWit etc.
* Sigs is intended only for cryptographic signatures indicating "who" sent the transaction for satisfying some input constraint, as its contents are not included in cryptographic hashes and cannot be relied on. Non-signature constraint-satisfaction data, or just random metadata, should be placed in Data instead.
* The maximum size of a transaction is 1 MB.

### Minting new cels

As described in the Celmint document, minting in Celerium is a two-step process involving **registering** and **solving** puzzles.

#### Registering puzzles

To register a puzzle, a transaction with `Kind == 0x10` is broadcast, with `Data` looking like:

```go
type PuzzleRegister struct {
    Constraint ConsScript
    Difficulty uint
}
```

`Constraint` poses an additional constraint on what sort of transaction is allowed to solve the puzzle. Typically it checks a signature to make sure the person solving the puzzle is the same minter that posed the puzzle.

Registration puzzles' inputs and outputs are validated as usual.

#### Solving puzzles

To solve a puzzle, a transaction with `Kind == 0x11` is broadcast, with `Data` looking like:

```go
type PuzzleSolve struct {
    Solution []byte
}
```

The transaction inputs and outputs are validated as usual, except the input number of cels is incremented by the reward determined by the difficulty of the puzzle. The result is a transaction with more output than input, thus minting new cels.

### Staking and unstaking

#### Staking

To stake a bond, a transaction with `Kind == 0x51` is broadcast, with `Data` looking like:

```go
type StakeBond struct {
    Host string
    Deposit uint
}
```

`Host` specifies the consensus-protocol host \(generally a public key\), while  `Deposit` is the size of the bond in microlents, which must be at least 1000 lents \($$10^9$$ microlents\), all of which must "disappear" from the transaction inputs.

The bond will last for exactly 500,000 blocks. That is, if the staking transaction happens in block $$n$$, then the stake will be active from block $$n+1$$ to $$n+500000$$, inclusive.

#### Unstaking

When bonds 

## Constraint scripting

### Overview

**Constraints** in Celerium correspond roughly to scripts in Bitcoin, except with a larger feature set and with no whitelist of formats that can be relayed. They are scripts written in a stack-based language that take a transaction \(and some other state such as the last block\) as an input and return a boolean; transactions must **satisfy** all their input TXO's constraints in order to validate. A constraint is satisfied if after running a transaction through the script, the stack has one element, `[]`.

Constraints are represented as a , but usually written in a fairly readable yet easily hand-translatable format.

### List of opcodes

| Opcode     | Encoding | Input          | Output       | Description                                        |
|:---------- |:-------- |:-------------- |:------------ |:-------------------------------------------------- |
| `pushbts`  | 0x10     | \(none\)       | data         | next 8 bytes is length of data to push             |
| `checksig` | 0x50     | sigalg, pubkey | \[1\], \[0\] | checks whether or not the tx has a valid signature |
|            |          |                |              |                                                    |

### Examples

\(Note: push is a macro\)

#### Check ed25519 signature

```text
push 0xdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeefdeadbeef
push "E"
checksig
```
