---
title:  "ethereum whitepaper - 笔记"
categories: 
  - ethereum
---
# Name Origin
- Aetherium/Volucite ([Crystal necklace](https://ghibli.fandom.com/wiki/Crystal_necklace) from `Laputa: Castle in the Sky`)
- [ether](https://markets.businessinsider.com/currencies/news/ethereum-name-origin-ether-vitalik-buterin-camilla-russo-infinite-machine-2021-5-1030477375)
  > Vitalik Buterin was inspired by a medieval scientific theory when he came up with the name for the ethereum network, according to Camilla Russo, former Bloomberg journalist and founder of DeFi news platform, The Defiant.
  >
  > In her 2020 book The Infinite Machine, Russo said that Buterin looked to science fiction terms for inspiration when he wrote the ethereum white paper in 2013.
  >
  > Buterin was scrolling through Wikipedia when he came across the word "ether," and remembered this term from a science book he had read as a child.
  >
  > Ether is the now disproved concept that there's a subtle material that fills space and carries all light waves. In the 19th century, scientists assumed ether was weightless, transparent, frictionless, undetectable chemically or physically, and permeated all matter and space, according to Britannica.
  >
  > "Vitalik wanted his platform to be the underlying and imperceptible medium for every application, just what medieval scientists thought ether was," Russo said. "Plus, it sounded nice." 
  > 
  > He landed on the word "ethereum," to describe the decentralized protocol.
  >
  > "Ether" refers to the cryptocurrency of the ethereum blockchain, a fuel that can be used for payments or to power decentralized apps built on the platform.
  >
  > The theory of ether was abandoned by scientists after Albert Einstein formed the special theory of relativity in 1905, per Britannica. But the use cases for the ethereum protocol are just getting started, crypto experts say.

# Basic Concepts
- Accounts
  > Ethereum account contains four fields:
  >
  > - The nonce, a counter used to make sure each transaction can only be processed once
  > - The account's current ether balance
  > - The account's contract code, if present
  > - The account's storage (empty by default)
  >
  > In general, there are two types of accounts: externally owned accounts, controlled by private keys, and contract accounts, controlled by their contract code. An externally owned account has no code, and one can send messages from an externally owned account by creating and signing a transaction; in a contract account, every time the contract account receives a message its code activates, allowing it to read and write to internal storage and send other messages or create contracts in turn.
- Messages and Transactions
  > The term "transaction" is used in Ethereum to refer to the signed data package that stores a message to be sent from an externally owned account. Transactions contain:
  >
  > - The recipient of the message
  > - A signature identifying the sender
  > - The amount of ether to transfer from the sender to the recipient
  > - An optional data field
  > - A STARTGAS value, representing the maximum number of computational steps the transaction execution is allowed to take
  > - A GASPRICE value, representing the fee the sender pays per computational step
  >
  > The fundamental unit of computation is "gas"; usually, a computational step costs 1 gas, but some operations cost higher amounts of gas because they are more computationally expensive, or increase the amount of data that must be stored as part of the state. There is also a fee of 5 gas for every byte in the transaction data.

  > Contracts have the ability to send "messages" to other contracts. Messages are virtual objects that are never serialized and exist only in the Ethereum execution environment. A message contains:
  >
  > - The sender of the message (implicit)
  > - The recipient of the message
  > - The amount of ether to transfer alongside the message
  > - An optional data field
  > - A STARTGAS value
- Code Execution
  > EVM's full computational state while running can be defined by the tuple (block_state, transaction, message, code, memory, stack, pc, gas)

# Blockchain and Mining
>  Ethereum blocks contain a copy of both the transaction list and the most recent state (In Ethereum, the state is made up of objects called "accounts")
- [Merkle Patricia Trie](https://ethereum.org/en/developers/docs/data-structures-and-encoding/patricia-merkle-trie/)
