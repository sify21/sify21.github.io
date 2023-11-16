---
title: "mastering ehtereum - 笔记"
categories: 
  - ethereum
---
## ch02 basics

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

- JSON-RPC

  https://www.jsonrpc.org/specification

## ch04 cryptography

- EOA: _externally owned accounts_
### private key

- nonzero number less than $2^{256}$ (a 78-digit number, roughly $1.158 * 10^{77}$, python `2 ** 256` to see)
- shares the first 38 digits with $2^{256}$ and is defined as the order of elliptic curve
- the visible universe is estimated to contain $10^{77}$ to $10^{80}$ atoms
### public key

- elliptic curve multiplication $K = k * G$. $k$: private key. $G$: constant point called the generator point
- the multiplication is called a "one-way" function by cryptographers
### elliptic curve cryptography

secp256k1, to reuse elliptic curve libraries and tools from Bitcoin
```math
y^2 = \big(x^3+7\big) over \big(\mathbb{F}_p\big)
```
$$y^2 \mod p = \big(x^3+7\big) \mod p$$

the curve is over a finite field of prime order $p$ (also written as $\mathbb{F}_p$), where $p = 2^{256} - 2^{32} - 2^9 - 2^8 - 2^7 - 2^6 - 2^4 -1$, which is a very large prime number.

The variable p is the prime order of the elliptic curve (the prime that is used for all the modulo operations). you can write the equation in python code as `(x**3 +7 - y**2) % p == 0`

### elliptic curve arithmetic operation
```math
P_1 + P_2 + P'_3 = 0, P'_3=(x,y), P_3=(x,-y)
```
https://bitcoin.stackexchange.com/questions/21907/what-does-the-curve-used-in-bitcoin-secp256k1-look-like
$$P_3 = P_1 + P_2$$

- point at infinity: ($P_1$ and $P_2$ have same x value but different y value, then $P_3$ is point at infinity)

- SECG (Standards for Efficient Cryptography Group) [SEC1](http://www.secg.org/sec1-v2.pdf)
  prefix | meaning | length(bytes counting prefix)
  --- | --- | ---
  0x00 | point at infinity | 1
  0x04 | uncompressed point | 65
  0x02 | compressed point with even y | 33
  0x03 | compressed point with odd y | 33

### cryptographic hash function: Keccak-256
- originally a SHA-3 candidate, NIST adjusted some parameters for efficiency, at the same time Edward Snowden revealed Dual_EC_DRBG backdoor. thus backlash against those changes and deplayed standardization of SHA-3, so Ethereum Foundation implement the original Keccak algorithm, rather than using the finalized FIPS-202 SHA-3 standard.

### Ethereum Address
- last 20 bytes of the Keccak-256 hash of the public key (without the prefix 0x04)
- higher layers should add checksums to those bytes
- ICAP (Inter exchange Client Address Protocol): similar to International Bank Account Number (IBAN) encoding, less support
- later, EIP-55 mixed-capitalization checksum

## ch05 wallets
HD wallet advantages
- organizational meaning
- create a sequence of public keys without private keys: watch-only or receive-only capacity

standards
- mnemonic code words, BIP-39
- HD wallets, BIP-32
  - extended keys (append a 256-bit chain code)
  - hardended child key derivation
    - normal derivation: index number from $0$ to $2^{31}-1$
    - hardened derivation: index number from $2^{31}$ to $2^{32}-1$ (displayed as $i^\prime$, meaning $2^{31}+i$
- Paths, BIP-43/44
  `m / purpose' / coin_type' / account' / change / address_index`

  forth level only "receive" path is used in ethereum

## ch06 transactions
Contracts don’t run on their own. Ethereum doesn’t run autonomously. Everything starts with a transaction.
- Nonce
  
  not stored, calculated dynamically(`getTransactionCount`), by counting the number of confirmed transactions that have originated from an address. vital for account-based protocol, in contrast to UTXO mechanism of Bitcoin protocol
  - tx being included by the order of creation(similar to JSON-RPC id of Request object)
  - replay (without nonce the two tx would be the same)
- Gas price
- Gas limit
- Recipient
- Value
- Data
  - when sent to EOA

    interpreted by wallet(ignored by most), not subject to Ethereum's consensus rules
  - when sent to ABI-compatible contract

    interpreted by EVM as contract invocation
    - A function selector: first 4 bytes of Keccak-256 hash of function's prototype, e.g. withdraw(uint256), and padded to 32 bytes
    - function arguments: encoded by rules for types in ABI specification
  - when sent to zero address (there is specially designated burn-address: 0x0..0dEaD)
  
    means contract creation, bytecode representation of the contract(optionally with value as starting balance)
- $v,r,s$
  
  The three components of an ECDSA digital signature of the originating EOA.
  $$Sig = F_{sig}\big(F_{keccak256}(m),k\big)$$
  - k signing private key
  - m RLP-encoded transaction(nonce, gasPrice, gasLimit, to, value, data, chainID, 0, 0)
  - $Sig = (r,s)$
    
  $r$ is $x$ coordinate of ephemeral public key $Q$ from ephemeral private key $q$, $s \equiv q^{-1}(Keccak256(m)+r*k) (\mod p)$

  $v$ indicates the chain ID and the recovery identifier (Public Key Recovery involves $r$, $s$ and message hash)

transaction serialized using RLP (Recursive Length Prefix), big-endian integers, no field delimiters or labels. EOA's public key is derived from $v,r,s$

## ch07 smart contract and solidity
- by Nick Szabo
- misnomer in Ethereum, neither smart nor legal contracts
- computer programs
- immutable (the code cannot change)
- deterministic
- EVM context
- world computer
- selfdestruct/suicide negative gas
### contract ABI(application binary interface)
- ABI defines how data structures and functions are accessed in machine code, thus primary way of encoding and decoding data into and out of machine code
- API(application programming interface) define this access in high level, often human-readable formats as source code
- in Ethereum, define functions in contract that can be invoked and describe function arguments and result
- specified as JSON array of function descriptions and events (solc --abi)

## ch08 vyper
- no function modifiers
- no inheritance
- no inline assembly
- no function overloading
- no implicit typecasting: use `convert` to perform explicit casts (raise exception when information lost;)
- deterministic number of iterations with `for`; no recursive calling
- preconditionsn and postconditions
  - condition
  - effects
  - interactions
- decorators
- one vyper file for one contract; function and variable declarations physically written in particular order
- overflow issue
  - solidity
    - SafeMath library
    - Mythril OSS security analysis tool
  - vyper
    - SafeMath equivalent
    - clamps

## ch09 smart contract security
- defensive programming
  - minimalism/simplicity
  - code reuse
  - code quality: unforgiving discipline
  - readability/auditability
  - test coverage

- security risks / antipatterns
  - reentrancy
    - use `transfer`
    - `checks-effects-interacctions` pattern
    - mutex
  - arithmetic over/underflows
    - SafeMath library by OpenZeppelin
  - unexpected ether

    ether added to contract without executing any code; or `this.balance` artificially manipulated
    - sendall/selfdestruct/suicide
    - pre-sent ether

      `contract_address = keccak256(rlp.encode([account_address,transaction_nonce]))`
  - delegatecall
    - context-reserving: state variables are referenced by slot index
    - library should be stateless
  - default visibilities
    - solidity: default is public
    - always specify visibility
  - entropy illusion
    - source of entropy must be external to the blockchain
      - commit-reveal(done among peers)
      - RandDAO
      - randomness oracle
  - external contract referencing
    
    author/owner/deployer of the contract(the attacker) wants to hide malicious code (from auditor)
    - `new` to create contracts
    - hardcode external contract addresses
    - external contral addresses should be public to allow users examnie referenced code
    - if external contract address can be changed, it should implement a time-lock/voting mechanism for auditing
  - short address/parameter attack

    performed on third-party applications that interact with contracts
    - EVM will add zeros to the end of the encoded parameters if they are shorter than the expected parameter length
    - input parameters in external applications should be validated
    - careful ordering of contract parameters can mitigate: padding(added zeros) only occurs at the end
  - unchecked call return values
    - `call`/`send` return `Boolean`, rather than revert
    - should use `transfer`(which reverts) to send ether
    - [_withdrawal_ pattern](https://docs.soliditylang.org/en/latest/common-patterns.html#withdrawal-from-contracts): isolated withdraw function
  - race conditions/front-running
    - attacker watch transaction pool, raise gasPrice
    - miners reorder transactions at their will
    - solution1: upper bound on gasPrice
    - solution2: commit-reveal scheme
    - solution3: submarine sends
  - dos
    - loop through externally manipulated mappings/arrays (exceeds block gas limit)
      - should use withdrawal pattern
    - owner privileged operations needed for the contract to proceed to the next state
      - can use multisig
      - can use time-lock
    - progressing state based on external calls
      - should add a time-based state progression
    - can add a `maintenanceUser` (centralized, trust-issue)
  - block timestamp manipulation
    - use block number instead
  - constructor name (before solidity v0.4.22)
  - uninitialized storage pointers
    - solidity by default puts local variables of  complex data types(e.g. structs) in storage
  - floating point and precision
  - tx.origin authentication
    - can only be used when: deny external contracts from calling current contract `require(tx.origin == msg.sender)`
