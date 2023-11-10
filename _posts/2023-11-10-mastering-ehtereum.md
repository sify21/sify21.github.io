---
title: "mastering ehtereum - 笔记"
categories: 
  - ethereum
---
## ch02 intro

Value (in wei) | Exponent | Common name | SI name
-- | -- | -- | --
1 | 1 | wei | Wei
1,000 | $10^3$ | Babbage | Kilowei or femtoether
1,000,000 | $10^6$ | Lovelace | Megawei or picoether
1,000,000,000 | $10^9$ | Shannon | Gigawei or nanoether
1,000,000,000,000 | $10^{12}$ | Szabo | Microether or micro
1,000,000,000,000,000 | $10^{15}$ | Finney | Milliether or milli
1,000,000,000,000,000,000 | $10^{18}$ | Ether | Ether
1,000,000,000,000,000,000,000 | $10^{21}$ | Grand | Kiloether
1,000,000,000,000,000,000,000,000 | $10^{24}$ | | Megaether

- `zero address`
  
  Registering a contract on the blockchain involves creating a special transaction whose destination is the address 0x0000000000000000000000000000000000000000, also known as the zero address

## ch03 clients
- DoS attack

  block 2,283,397 (2016-09-18) to block 2,700,031 (2016-11-26). validation time: 1min/block. hard fork cleaned up 20 million empty accounts created by spam transactions. fast synchronization skips the full validation of transactions until it has synced to the tip of the blockchain.

- `Remote Client` $\ne$ `Light Client`

  `Light Client` is analogous to a Simplified Payment Verification client in Bitcoin. Light clients are in development and not in general use for Ethereum
