---
mathjax: true
title:  "ethereum yellowpaper - 笔记"
categories: 
  - ethereum
---

## World State

symbol | meaning
--- | ---
$\sigma$ | world state
$\sigma [a]$ | account state
$\sigma [a]_n$ | nounce
$\sigma [a]_b$ | balance(Wei)
$\sigma [a]_s$ | storageRoot
$\sigma [a]_c$ | codeHash

$$\forall a : \sigma [a] = \varnothing \vee (a \in \mathbb{B}_{20} \wedge v(\sigma[a]))$$

$$v(x) \equiv x_n \in \mathbb{N}_{256} \wedge x_b \in \mathbb{N}_{256} \wedge x_s \in \mathbb{B}_{32} \wedge x_c \in \mathbb{B}_{32}$$

$$EMPTY(\sigma, a) \equiv \sigma[a]_c=KEC\big(()\big) \wedge \sigma[a]_n=0 \wedge \sigma[a]_b=0$$

$$DEAD(\sigma, a) \equiv \sigma[a]=\varnothing \vee EMPTY(\sigma, a)$$


## Transaction

symbol | meaning
--- | ---
$T_x$ | type
$T_p$ | gasPrice
$T_g$ | gasLimit
$T_t$ | to
$T_v$ | value(Wei)
$T_r$, $T_s$ | signature (r, s)
$T_A$ (type 1) | accessList
$T_c$ (type 1) | chainId $\beta$
$T_y$ (type 1) | yParity
$T_w$ (type 0 legacy) | $T_w=27+T_y$ or $T_w=2\beta+35+T_y$
$T_i$ ($T_t=\varnothing$)| init (contract creation)
$T_d$ ($T_t\ne\varnothing$)| data (message call)

## Block

symbol | meaning
--- | ---
B | block
$B_H$ | block header
$B_T$ | transaction list
$B_U$ | ommer block headers

$$B \equiv (B_H, B_T, B_U)$$

block header field | meaning
--- | ---
$H_p$ | parentHash
$H_o$ | ommersHash
$H_c$ | beneficiary
$H_r$ | stateRoot
$H_t$ | transactionsRoot
$H_e$ | receiptsRoot
$H_b$ | logsBloom
$H_d$ | difficulty
$H_i$ | number
$H_l$ | gasLimit
$H_g$ | gasUsed
$H_s$ | timestamp
$H_x$ | extraData
$H_m$ | mixHash
$H_n$ | nonce

### Transaction Receipt

symbol | meaning
--- | ---
$B_R$ | transaction receipt trie of the block, $B_R[i]$ for the $i$ th transaction
R | transaction receipt
$R_x$ | type of the transaction
$R_z$ | status code of the transaction
$R_u$ | the cumulative gas used in the block containing the transaction receipt as of immediately after the transaction has happened
$R_l$ | the set of logs created through execution of the transaction
$R_b$ |  the Bloom filter composed from information in those logs

$$R \equiv (R_x, R_z, R_u, R_b, R_l)$$

log entry field | meaning
--- | ---
O | log entry
$O_a$ | logger's address
$O_t$ | a possibly empty series of 32-byte log topics
$O_d$ | log data

$$O \equiv (O_a, (O_{t0}, O_{t1}, ...), O_d)$$
