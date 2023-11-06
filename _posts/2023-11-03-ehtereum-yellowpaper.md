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

$$\forall a : \sigma [a] = \varnothing \vee (a \in \mathbb{B}_{20} \wedge v(\sigma[a]))$$

$$\mathbb{B}_{256} \mathbb{B}_{256}$$

$$v(x) \equiv x_n \in \mathbb{N}_{256} \wedge x_b \in \mathbb{N}_{256} \wedge x_s \in \mathbb{B}_{32} \wedge x_c \in \mathbb{B}_{32}$$

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
