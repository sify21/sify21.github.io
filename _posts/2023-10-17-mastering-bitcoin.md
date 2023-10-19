---
title:  "mastering bitcoin - 笔记"
categories: 
  - bitcoin
---
# ch02
Alice to Bob's rawtx(P2PKH)
```
0100000001186f9f998a5aa6f048e51dd8419a14d8a0f1a8a2836dd734d2804fe65fa35779000000008b483045022100884d142d86652a3f47ba4746ec719bbfbd040a570b1deccbb6498c75c4ae24cb02204b9f039ff08df09cbe9f6addac960298cad530a863ea8f53982c09db8f6e381301410484ecc0d46f1918b30928fa0e4ed99f16a0fb4fde0735e7ade8416ab9fe423cc5412336376789d172787ec3457eee41c04f4938de5cc17b4a10fa336a8d752adfffffffff0260e31600000000001976a914ab68025513c3dbd2f7b92a94e0581f5d50f654e788acd0ef8000000000001976a9147f9b1a7fb68d60c536c2fd8aeaa53a8f3cc025a888ac00000000
```
its parent rawtx
```
0100000001524d288f25cada331c298e21995ad070e1d1a0793e818f2f7cfb5f6122ef3e71000000008c493046022100a59e516883459706ac2e6ed6a97ef9788942d3c96a0108f2699fa48d9a5725d1022100f9bb4434943e87901c0c96b5f3af4e7ba7b83e12c69b1edbfe6965f933fcd17d014104e5a0b4de6c09bd9d3f730ce56ff42657da3a7ec4798c0ace2459fb007236bc3249f70170509ed663da0300023a5de700998bfec49d4da4c66288a58374626c8dffffffff0180969800000000001976a9147f9b1a7fb68d60c536c2fd8aeaa53a8f3cc025a888ac00000000
```
# ch04
```
K=kG, 0<k<=n-1
n(1.1578 * 1077) is slightly less than 2^256(order of elliptic curve)
```
```
A=RIPEMD160(SHA256(K)), or A=HASH160(K)
```
```
checksum = first 4 bytes of: SHA256(SHA256(prefix + data))
Base58Check-encoded Payload = Base58(prefix + data + checksum)
```
Base58Check version prefix and encoded result examples

|Type| Version prefix (hex)| Base58 result prefix|
|---|---|---|
| Bitcoin Address | 0x00 | 1|
| Pay-to-Script-Hash Address | 0x05 | 3|
| Bitcoin Testnet Address | 0x6F | m or n|
| Private Key WIF |  0x80 | 5, K, or L|
| BIP-38 Encrypted Private Key | 0x0142 | 6P|
| BIP-32 Extended Public Key | 0x0488B21E | xpub|

```
uncompressed public key = 04 x y
compressed public key = 02 x(y is even) or 03 x(y is odd)
compressed private key (for wallet import format) = uncompressed private key 01
```
# ch05
`mnemonic(128-256 bits)` -> (optinaly with salt) key-stretching function PBKDF2(2048 rounds of HMAC-SHA512) -> `seed(512-bit)` -> HMAC-SHA512 -> `master pri key(left 256 bits) + master chain code(right 256 bits)`
- Mnemonic code words, based on BIP-39
- HD(hierarchical deterministic) wallets, based on BIP-32
- Multipurpose HD wallet structure, based on BIP-43
- Multicurrency and multiaccount wallets, based on BIP-44: `m / purpose'(always 44’) / coin_type' / account' / change(0 for receiving address or signing prikey, 1 for change address) / address_index (usable addresses)`

# ch07
## p2sh(BIP-16)
- execute scriptSig
- the stack gets copied (contains \<redeem script\>, which is the original scripPub of p2pbk or p2ms)
- execute scriptPub
- pop and deserialize \<redeem script\> from the copied stack, which becomes the new locking script

2-of-3 multisig script using p2sh adds an extra 25 bytes to the overall script, compared to 2-of-3 multisig script using the simple P2MS pattern.
## segwit
- BIP-141: The main definition of Segregated Witness.
- BIP-143: Transaction Signature Verification for Version 0 Witness Program
- BIP-144: Peer Services—New network messages and serialization formats
- BIP-145: getblocktemplate Updates for Segregated Witness (for mining)
- BIP-173: Base32 address format for native v0-16 witness outputs