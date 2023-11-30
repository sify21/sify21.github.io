---
title:  "mastering bitcoin - 笔记"
categories: 
  - bitcoin
---
# ch02 How Bitcoin Works
Alice to Bob's rawtx(P2PKH)
```
0100000001186f9f998a5aa6f048e51dd8419a14d8a0f1a8a2836dd734d2804fe65fa35779000000008b483045022100884d142d86652a3f47ba4746ec719bbfbd040a570b1deccbb6498c75c4ae24cb02204b9f039ff08df09cbe9f6addac960298cad530a863ea8f53982c09db8f6e381301410484ecc0d46f1918b30928fa0e4ed99f16a0fb4fde0735e7ade8416ab9fe423cc5412336376789d172787ec3457eee41c04f4938de5cc17b4a10fa336a8d752adfffffffff0260e31600000000001976a914ab68025513c3dbd2f7b92a94e0581f5d50f654e788acd0ef8000000000001976a9147f9b1a7fb68d60c536c2fd8aeaa53a8f3cc025a888ac00000000
```
its parent rawtx
```
0100000001524d288f25cada331c298e21995ad070e1d1a0793e818f2f7cfb5f6122ef3e71000000008c493046022100a59e516883459706ac2e6ed6a97ef9788942d3c96a0108f2699fa48d9a5725d1022100f9bb4434943e87901c0c96b5f3af4e7ba7b83e12c69b1edbfe6965f933fcd17d014104e5a0b4de6c09bd9d3f730ce56ff42657da3a7ec4798c0ace2459fb007236bc3249f70170509ed663da0300023a5de700998bfec49d4da4c66288a58374626c8dffffffff0180969800000000001976a9147f9b1a7fb68d60c536c2fd8aeaa53a8f3cc025a888ac00000000
```
# ch04 Keys, Addresses
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
# ch05 Wallets
`mnemonic(128-256 bits)` -> (optinaly with salt) key-stretching function PBKDF2(2048 rounds of HMAC-SHA512) -> `seed(512-bit)` -> HMAC-SHA512 -> `master pri key(left 256 bits) + master chain code(right 256 bits)`

CKD function visualized. from [a nice blog](https://medium.com/@blainemalone01/hd-wallets-why-hardened-derivation-matters-89efcdc71671) of @blainemalone01 on medium.

![child key derivation function](/assets/images/bip32.webp)

- Mnemonic code words, based on [BIP-39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)
- HD(hierarchical deterministic) wallets, based on [BIP-32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)
- Multipurpose HD wallet structure, based on [BIP-43](https://github.com/bitcoin/bips/blob/master/bip-0043.mediawiki)
- Multicurrency and multiaccount wallets, based on [BIP-44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki): `m / purpose'(always 44’) / coin_type' / account' / change(0 for receiving address or signing prikey, 1 for change address) / address_index (usable addresses)`

# ch07 Advanced Transactions and Scripting
## p2sh([BIP-16](https://github.com/bitcoin/bips/blob/master/bip-0016.mediawiki))
- execute scriptSig
- the stack gets copied (contains \<redeem script\>, which is the original scripPub of p2pbk or p2ms)
- execute scriptPub
- pop and deserialize \<redeem script\> from the copied stack, which becomes the new locking script

2-of-3 multisig script using p2sh adds an extra 25 bytes to the overall script, compared to 2-of-3 multisig script using the simple P2MS pattern.
## segwit
- [BIP-141](https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki) The main definition of Segregated Witness.
- [BIP-143](https://github.com/bitcoin/bips/blob/master/bip-0143.mediawiki) Transaction Signature Verification for Version 0 Witness Program
- [BIP-144](https://github.com/bitcoin/bips/blob/master/bip-0144.mediawiki) Peer Services—New network messages and serialization formats
- [BIP-145](https://github.com/bitcoin/bips/blob/master/bip-0145.mediawiki) getblocktemplate Updates for Segregated Witness (for mining)
- [BIP-173](https://github.com/bitcoin/bips/blob/master/bip-0173.mediawiki) Base32 address format for native v0-16 witness outputs

A scriptPubKey (or redeemScript as defined in BIP16/P2SH) that consists of a 1-byte push opcode (for 0 to 16) followed by a data push between 2 and 40 bytes gets a new special meaning, which is interpreted as  "version byte" + "witness program".

two cases in which witness validation logic are triggered:
- native witness program: above pattern is in scriptPubKey (scriptSig must be empty or validaition fails)
- P2SH witness program: above pattern is in redeemScript (scriptSig must be exactly a push of the BIP16 redeemScript or validation fails)

`version byte` = `0` && `witness program` = `20 bytes`: interpreted as P2WPKH program, the witness must consist of exactly 2 items (≤ 520 bytes each). The first one a signature, and the second one a public key.

`version byte` = `0` && `witness program` = `32 bytes`: interpreted as P2WSH program, The witness must consist of an input stack to feed to the script, followed by a serialized script(witness script, ≤ 10,000 bytes).

# ch08 The Bitcoin Network
- P2P network
- FIBRE(fast internet bitcoin relay engine) 挖矿用
## full node

## SPV node
只存储Block头。SPV verifies transactions by reference to their depth in the blockchain instead of their height。节点等待6个新block堆叠在包含tx的block上，以此证明tx被认证了。

消息(会过滤peer连接中的block和tx消息)：
- getheaders(headers,包含2000个)
- getdata(merkleblock,tx)
- filterload, filteradd, filterclear([BIP-37](https://github.com/bitcoin/bips/blob/master/bip-0037.mediawiki) 请求特定tx会暴露钱包管理的地址，所以引入Bloom Filters提高私密性)

通过消息加密来提升SPV节点的隐私性
- Tor
- P2P层面 : Peer Authentication(BIP-150), Peer-to-Peer Communication Encryption(BIP-151）

# ch09 The Blockchain
## Block
|Size| Field | Description |
| -- | -- | -- |
| 4 bytes | Block Size | The size of the block, in bytes, following this field |
| 80 bytes | Block Header | Several fields form the block header |
| 1&#x2013;9 bytes (VarInt) | Transaction Counter | How many transactions follow |
| Variable | Transactions | The transactions recorded in this block |
## Block Header(40 bytes)
|Size| Field | Description |
| -- | -- | -- |
| 4 bytes | Version | A version number to track software/protocol upgrades |
| 32 bytes | Previous Block Hash | A reference to the hash of the previous (parent) block in the chain |
| 32 bytes | Merkle Root | A hash of the root of the merkle tree of this block's transactions |
| 4 bytes | Timestamp | The approximate creation time of this block (in seconds elapsed since Unix Epoch) |
| 4 bytes | Difficulty Target | The Proof-of-Work algorithm difficulty target for this block |
| 4 bytes | Nonce | A counter used for the Proof-of-Work algorithm |

`authentication path`, or `merkle path`: nodes in the merkle tree to make hash with, bottom-up.

The merkleblock message contains the block header as well as a merkle path that links the transaction of interest to the merkle root in the block

# ch12 Blockchain Applications
## payment channel / state channel
funding transaction / anchor transaction (on-chain) ----> commitment transactions (off-chain, each state invalidates previous state) ----> settlement transaction (on-chain)

various mechanisms that can be used to invalidate prior state :
- transaction-level timelocks (nLocktime)

    The refund transaction acts as the first commitment transaction and its timelock establishes the upper bound for the channel’s life.

    Each commitment sets a shorter timelock, allowing it to be spent before the previous commitments become valid

- Asymmetric revocable commitments with relative time locks (CSV CheckSequenceVerify)

    `time delay` + `revocation key` as a punishment/disadvantage/penalty

    我构造的（让对方签的）tx包含如下输出
    ```
    Output 0 <5 bitcoin>:
        <对方的 Public Key> CHECKSIG

    Output 1 <5 bitcoin>:
        IF
            # Revocation penalty output
            <Revocation Public Key>
        ELSE
            <1000 blocks>
            CHECKSEQUENCEVERIFY
            DROP
            <我的 Public Key>
        ENDIF
        CHECKSIG
    ```
    进入下一个state，我要revoke我自己的tx，做法是：把我的half of the revoke secret发送给对方。
  
    在每个阶段，对方也持有类似的一个tx。

## routed payment channels (lightning network)
HTLC (hash time lock contract)
- hash: redeem immediately
- time lock: refund after timeout(if the secret is not revealed)

*疑问1* 在channel末端，返回private key后，为什么channel中的每个节点必须把key传给上家？当前节点有了key就可以直接得到钱了啊

