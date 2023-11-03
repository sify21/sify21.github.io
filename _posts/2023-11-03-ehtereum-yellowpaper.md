---
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

## transaction state
symbol | meaning
--- | ---
$T_x$ | type
$T_p$ | gasPrice
$T_g$ | gasLimit
$T_t$ | to
$T_v$ | value(Wei)
$T_r$, $T_s$ | signature (r, s)
$T_A$(type 1) | accessList
$T_c$(type 1) | chainId $\beta$
$T_y$(type 1) | yParity
$T_w$(type 0 legacy) | $T_w=27+T_y$ or $T_w=2\beta+35+T_y$
$T_i$ | init
