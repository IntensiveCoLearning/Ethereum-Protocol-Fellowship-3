---
timezone: UTC+8
---

> 请在上边的 timezone 添加你的当地时区(UTC)，这会有助于你的打卡状态的自动化更新，如果没有添加，默认为北京时间 UTC+8 时区


# 你的名字

1. 自我介绍

**Dex** 对这个世界保持好奇心->想深入了解以太坊

2. 你认为你会完成本次残酷学习吗？

会的

3. 你的联系方式（推荐 Telegram）

- **Telegram**：[t.me/dexhunt3r](https://t.me/dexhunt3r)  
- **Twitter/X**：[x.com/dexhunt3r](https://x.com/dexhunt3r)  
- **farcaster**: [warpcaster.com/dexhunter.eth](https://warpcast.com/dexhunter.eth)

## Notes

<!-- Content_START -->

### 2025.03.10

先把所有资源看了一下，比如

* [油管频道eth protocol fellows](https://www.youtube.com/@ethprotocolfellows/videos)
* [wiki efp.wiki](https://epf.wiki/#/eps/intro)
* [discord](https://discord.com/invite/epfsg)
* [github repo protocol studies](https://github.com/eth-protocol-fellows/protocol-studies)
* [ethereum quizzes](https://ethereum.org/en/quizzes/) 


今天好像已经是week4了，所以需要把之前的内容过一下
* [week1](https://epf.wiki/#/eps/week1)
    * 大概介绍了下protocol, 历史和哲学
    * 以太坊协议设计
        * [技术黄皮书](https://ethereum.github.io/yellowpaper/paper.pdf)
        * 实现
        * 测试
        * 合作
* [week2](https://epf.wiki/#/eps/week2)
    * week1的链接其实直接先到week3的，不确定为啥，week3对应的是lecture2 consensus，具体看[这里](https://github.com/eth-protocol-fellows/protocol-studies/blob/main/docs/eps/week3.md)
    * 执行层
        * 区块
        * 状态机
        * jrpc信息传递
* [week3](https://epf.wiki/#/eps/week3)
    * 共识层就是PoS，具体看Casper的[论文](https://arxiv.org/pdf/2003.03052)



- *做了一下quizzes，感觉主要是夸夸以太坊*
![](./assets/dexhunter/screenshot-0310.png)

### 2025.03.11

今天看了[week4](https://epf.wiki/#/eps/week4)的内容，主要是关于测试和安全
* 第一次知道retesteth还有个webui [这里](http://retesteth.ethdevops.io/web/)，可以测试各种不同的方法，教程在[这里](https://ethereum-tests.readthedocs.io/en/latest/retesteth-tutorial.html)，还蛮有意思的

```
Running tests using path: /data/tests
Running 1 test case...
Retesteth config path: /var/www/.retesteth
WARNING: Retesteth configs version is different (running: '0.3.2-legacy' vs config '0.3.2-cancun')!
WARNING: Update configs to the latest by deleting the folder `/var/www/.retesteth`!
Active client configurations: 't8ntool '
Checking test filler hashes for GeneralStateTests/stExample
Filter: 'accessListExample Shanghai'
Check `/data/tests/GeneralStateTests/stExample/accessListExample.json` hash
SrcFile `/data/tests/src/GeneralStateTestsFiller/stExample/accessListExampleFiller.yml`
Read json structure accessListExampleFiller.yml
Read json structure finish
Running tests for config 'Ethereum GO on StateTool' 2
Test Case "stExample": (1 of 1)
100%
Instantiated: "evm version 1.15.6-unstable-4cdd7c86-20250310"
Running accessListExample: (6995263567478450108)
Read json structure accessListExample.json
Read json finish
WARNING: Specified filter did not run a single transaction!  (GeneralStateTests/stExample/accessListExample, fork: Shanghai, TrInfo: d: -1, g: -1, v: -1)

*** No errors detected
*** Total Tests Run: 1


--------
*** TOTAL WARNINGS DETECTED: 3 warnings during all test execution!
--------
info: Retesteth configs version is different (running: '0.3.2-legacy' vs config '0.3.2-cancun')!
info: Update configs to the latest by deleting the folder `/var/www/.retesteth`!
info: Specified filter did not run a single transaction!  (GeneralStateTests/stExample/accessListExample, fork: Shanghai, TrInfo: d: -1, g: -1, v: -1) (GeneralStateTests/stExample/accessListExample, fork: Shanghai, TrInfo: d: -1, g: -1, v: -1)
```

### 2025.03.12

不知道为什么昨天的commit被overwritten了，然后根据读历史的启发今天开始带着问题来学习。
-> 今天想知道的问题是，为什么以太坊主链到2025年了还是这么贵和慢 why is ehtereum mainnet still costly and slow in 2025?


slowness:
> blocks produced every 12 seconds and each block containing about 185 transactions on average.

costliness:
> The costliness arises from gas fees, which users pay for computational resources. During high demand, these fees spike due to limited block space, and the market-driven gas price mechanism means users must bid higher to get transactions processed quickly. Even with changes like the London hard fork in 2021 introducing a base fee, network congestion still drives up costs.

根据 [EIP-1559](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1559.md)
```
Transaction/Gas fee = Gas used × (Base fee + Priority fee)
```

可能是要等pectra和sharding才能让主链交易变得更快或便宜

References:
1. [chatgpt deepresearch reference](https://chatgpt.com/share/67d0ced5-5e74-800d-89a3-7bd7fc4f8006)
2. [grok research](https://grok.com/share/bGVnYWN5_1aab2e22-5f73-48db-bf1f-718eed7559c6)


### 2025.03.13

继续啃PoS

* [eth pos](https://ethereum.org/en/developers/docs/consensus-mechanisms/pos/)
* [vitalik's post](https://vitalik.eth.limo/general/2017/12/31/pos_faq.html)
* [Casper FFG](https://arxiv.org/pdf/1710.09437)

--> 发现一个简单的python实现 [simple casper](https://github.com/omoindrot/simple_casper/tree/master)

不过好像没有测试finality和一些攻击的场景，感觉可以改写试试

原本的casper代码 [https://github.com/ethereum/casper](https://github.com/ethereum/casper)
新的代码  [https://github.com/ethereum/consensus-specs](https://github.com/ethereum/consensus-specs)

validator的条件
* 32 ETH

消减条约 slashing rules
* A validator must not publish two distinct votes for the same target height
* A validator must not vote within the span of its other votes.

### 2025.03.14

参考两本书
* [Understanding-Ethereum-Go-Version](https://github.com/ABCDELabs/Understanding-Ethereum-Go-version)
* [mastering ethereum](https://github.com/ethereumbook/ethereumbook)

以及开始看以太坊源码
* [reth](https://github.com/paradigmxyz/reth)
* [go-eth](https://github.com/ethereum/go-ethereum)

### 2025.03.15

关注一下mev
* 看到了siyuan大佬的[post](https://x.com/yzilabs/status/1899813012233752989)
* [mev book](https://www.monoceros.com/insights/maximal-extractable-value-book)
* [mev book on github](https://github.com/0xOsiris/Mev_Book)
* also take a look at flash bot's mev letters

了解了一些holesky的事情背景
[grok summary](https://grok.com/share/bGVnYWN5_a8cfe9ef-e843-4539-8aee-cfaf4cc181e9)


### 2025.03.16

看了这个黄皮书简化版的解释[1]和精通以太坊第一章[2]

总结下
以太坊合约是一个有确定性但实际上无限的状态机（unbounded state machine），主要有两个功能
1. 全局可访问的状态 singleton state (distributed single-state)
2. 可以改变这个状态的虚拟机

World State 世界状态是 地址 到 账户 的映射，这部分的信息用 Merkle Patricia Trees 存储在数据库后端

我觉得这篇关于以太坊的技术实现文章写的非常通俗易懂，如果有不懂的内容可以参考

[1]: https://github.com/chronaeon/beigepaper/blob/master/beigepaper.pdf
[2]: https://github.com/ethereumbook/ethereumbook/blob/develop/01what-is.asciidoc

### 2025.03.17

看精通以太坊第二章[1]

1 wei = 1e-18 ether
or 1 ether = 1e18 wei

wallet (personal opinion)
* ~~metamask~~ -> DO NOT USE (as for March 2025) unless you are lazy
-> rabby wallet (the safe one)

don't lose your priv key
-> use a cold wallet if possible

#### Networks

* main net
* test net
    * ropsten
    * kovan
    * rinkeby
* localhost/custom rpc

TODO: will read more tomorrow about smart contract and world computer concept


[1]: https://github.com/ethereumbook/ethereumbook/blob/develop/02intro.asciidoc

### 2025.03.18

精通以太坊第三章[1]

几个客户端的实现
1. Parity, written in Rust4
2. Geth, written in Go
3. cpp-ethereum, written in C++
4. pyethereum, written in Python
5. Mantis, written in Scala
6. Harmony, written in Java

[1]: https://github.com/ethereumbook/ethereumbook/blob/develop/03clients.asciidoc

### 2025.03.19

第四章，关于密码学，内容比较多，分几天来看完[1]

椭圆曲线密码学[2]

![](https://github.com/ethereumbook/ethereumbook/blob/develop/images/ec_over_small_prime_field.png)

* OpenSSL
* Bitcoin's libsecp256k1 -> secp256k1 elliptic curve 
* Hash Function


[1]: https://github.com/ethereumbook/ethereumbook/blob/develop/04keys-addresses.asciidoc
[2]: https://github.com/ethereumbook/ethereumbook/blob/develop/04keys-addresses.asciidoc#elliptic_curve


### 2025.03.22

* two types of accounts
    * externally owned accounts (EOAs) <- private key + address + signature
    * contracts <- smart contract code (executed by EVM)

#### Based on [chapter 5](https://github.com/ethereumbook/ethereumbook/blob/develop/05wallets.asciidoc)

Ethereum wallets serve as the primary user interface to Ethereum, with two main aspects:
- **For users**: Applications that manage keys, track balances, and create/sign transactions
- **For developers**: Systems for key management and storage

Despite the name, wallets don't actually contain ether or tokens – these assets exist on the blockchain. Wallets hold the keys that provide control over these assets.

Wallet Technology Types:

##### 1. Nondeterministic (Random) Wallets
- Each key is independently generated from a different random number
- Also known as "Just a Bunch of Keys" (JBOK) wallets
- More difficult to back up and manage
- Example: Keystore files (JSON-encoded, encrypted by passphrase)

##### 2. Deterministic (Seeded) Wallets
- All keys derived from a single master key (seed)
- A single backup at creation secures all funds
- Seed can be encoded as mnemonic code words for easier backup

##### 3. Hierarchical Deterministic (HD) Wallets
- Based on BIP-32 standard
- Keys derived in a tree structure (parent keys → child keys → grandchild keys)
- Provides organizational structure and improved security


Modern wallets typically implement several industry standards:

##### BIP-39: Mnemonic Code Words
- Creates a human-readable backup (12-24 words)
- Random entropy → checksum → encode as words → derive seed
- Optional passphrase adds security (but increases risk of fund loss if forgotten)

##### BIP-32: Hierarchical Deterministic Wallets
- Creates tree-like structure of keys
- Extended keys (xprv, xpub) can derive child keys
- Hardened derivation prevents chain code compromise

##### BIP-43/44: Standardized HD Wallet Structure
- Defines purpose and path for HD wallets
- Standard path: m/purpose'/coin_type'/account'/change/address_index
- For Ethereum: m/44'/60'/0'/0/x (where x is the address index)

#### Key Concepts

- **Extended Keys**: Parent keys that can derive child keys (with chain code)
- **Hardened Derivation**: Breaks parent-child relationship to improve security
- **HD Wallet Path**: Naming convention for identifying keys in the tree (e.g., m/44'/60'/0'/0/2)
- **Mnemonic Words vs. Brainwallets**: Mnemonics are generated randomly by the wallet, not chosen by the user

#### Best Practices

For implementing Ethereum wallets, build an HD wallet with a seed encoded as mnemonic code, following BIP-32, BIP-39, BIP-43, and BIP-44 standards for maximum security, flexibility, and compatibility.

#### Security Considerations

- Always back up your seed phrase (mnemonic words) in a secure, physical location
- Never store your seed phrase digitally
- Consider using hardware wallets for large amounts of funds
- If using a passphrase with your mnemonic, ensure it can be recovered by trusted parties if needed

### 2025.03.23

Transactions are signed messages originated by externally owned accounts (EOAs), transmitted through the Ethereum network, and recorded on the blockchain. They are the only mechanism that can:
- Trigger state changes in Ethereum
- Cause smart contracts to execute

A transaction contains the following data:

- **Nonce**: A sequence number issued by the originator to prevent message replay
- **Gas price**: Amount of ether (in wei) the originator is willing to pay per unit of gas
- **Gas limit**: Maximum amount of gas the originator is willing to buy
- **Recipient**: Destination Ethereum address
- **Value**: Amount of ether (in wei) to send
- **Data**: Variable-length binary data payload
- **v, r, s**: Components of the ECDSA digital signature

*All transactions are serialized using Recursive Length Prefix (RLP) encoding.*

The nonce is critical for two reasons:
1. **Transaction ordering**: Ensures transactions from the same account are processed in the order they were created
2. **Replay protection**: Makes each transaction unique, preventing duplicated payments

*Nonces are tracked sequentially for each account. The nonce for a new transaction must be exactly one higher than the previous transaction's nonce.*

Tracking Nonces:
- Use `web3.eth.getTransactionCount(address)` to find the current nonce
- For multiple rapid transactions, maintain your own nonce counter
- Parity offers `parity_nextNonce` for more reliable nonce tracking

Nonce Gaps and Concurrency Issues:
- Missing nonces will cause subsequent transactions to be held in the mempool
- Duplicate nonces will cause one transaction to be accepted and one rejected
- Concurrency can cause problems when multiple systems generate transactions from the same account

Gas is Ethereum's mechanism for:
- Measuring computational resource usage
- Preventing DoS attacks
- Allocating resources in a fair manner

Two important components:
- **Gas price**: How much ether per unit of gas (set by the transaction creator)
- **Gas limit**: Maximum gas units the transaction can consume

*Simple ether transfers between EOAs always cost exactly 21,000 gas. Contract interactions vary in cost depending on complexity.*

The recipient is specified in the `to` field:
- Can be an EOA or contract address
- No validation is performed on this field
- Sending to an invalid address effectively "burns" the ether

Transactions can contain:
- **Value only**: A payment
- **Data only**: A contract invocation
- **Both**: A payment and contract invocation
- **Neither**: Technically valid but not useful

For contract interactions, the data field typically contains:
- Function selector: First 4 bytes of the Keccak-256 hash of the function signature
- Function arguments: Encoded according to ABI specification

Special Transaction: Contract Creation

Contracts are created by sending transactions to the special "zero address" (0x0):
- The `to` field is set to 0x0
- The contract bytecode is included in the data field
- Optional ether can be included in the `value` field to fund the new contract

Digital Signatures

Transactions are signed using the Elliptic Curve Digital Signature Algorithm (ECDSA). Signatures serve three purposes:

1. **Authorization**: Proves the owner of the private key authorized the transaction
2. **Non-repudiation**: The proof of authorization is undeniable
3. **Integrity**: The transaction data cannot be modified after signing

The signature includes three values:
- **r, s**: The ECDSA signature components
- **v**: Recovery identifier that helps derive the public key from the signature

ECDSA Mathematics

The ECDSA signature creation process:
1. Generate an ephemeral (temporary) private key
2. Derive the corresponding ephemeral public key
3. The r value is the x coordinate of the ephemeral public key
4. The s value is calculated using the private key, transaction hash, and ephemeral key

note: *Signature verification can recover the public key without knowing the private key.*

Transaction Signing in Practice

To sign a transaction:
1. Create a transaction data structure with all fields
2. RLP-encode the transaction data
3. Calculate the Keccak-256 hash of the serialized message
4. Sign the hash with the private key using ECDSA
5. Append the signature values to the transaction

Offline Signing

For security, transaction signing can be separated from transaction transmission:
1. Create unsigned transaction on an online system
2. Transfer to an offline system for signing
3. Transfer the signed transaction back to an online system for broadcasting

note: *This process, called "offline signing," helps protect private keys by keeping them off internet-connected computers.*

Transaction Propagation

Transactions propagate through the Ethereum network via a flood routing protocol:
1. The node with the signed transaction validates it
2. It then transmits the transaction to all its neighbors
3. Each neighbor validates and retransmits to their neighbors
4. Within seconds, the transaction reaches all nodes in the network

Recording on the Blockchain

Valid transactions are eventually:
1. Selected by miners for inclusion in a block
2. Recorded permanently on the blockchain
3. Execute their effect on the Ethereum state

Multiple-Signature (Multisig) Transactions

While Ethereum's basic transactions don't directly support multiple signatures, smart contracts can implement multisig functionality:
- Funds are sent to a contract that enforces signature requirements
- Multisig contracts can have arbitrary signing conditions
- The contract only releases funds when conditions are satisfied

EIP-155: Transaction Replay Protection

EIP-155 prevents transaction replay attacks between different Ethereum networks by:
- Including a chain identifier in the transaction data
- Modifying the signing algorithm to encode the chain ID in the v value
- Making it impossible to replay a transaction meant for one network on another

### 2025.03.24

Based on [Chapter 7](https://github.com/ethereumbook/ethereumbook/blob/develop/07smart-contracts-solidity.asciidoc)

What Is a Smart Contract?

Smart contracts in Ethereum are immutable computer programs that run deterministically in the Ethereum Virtual Machine. Despite the name, they aren't "smart" in the AI sense, nor are they legal contracts. They're simply programs deployed on the blockchain that execute exactly as programmed without possibility of downtime, censorship, or third-party interference.

Life Cycle of a Smart Contract

Smart contracts are written in high-level languages (most commonly Solidity), compiled to EVM bytecode, and deployed through special creation transactions. Each contract has a unique Ethereum address but no associated private key. Contracts only execute when called by a transaction, either directly from an EOA or indirectly via another contract. They remain dormant until triggered, and execution is atomic—either fully successful or completely reverted.

Ethereum High-Level Languages

While EVM bytecode is the ultimate language of execution, developers typically write smart contracts in higher-level languages. Solidity is the most widely used, but others include LLL, Serpent, Vyper, and Bamboo. These languages vary in their approach, with some being more declarative (functional) and others more imperative (procedural).

Building Smart Contracts with Solidity

Solidity is an object-oriented language with syntax similar to JavaScript or C++. It includes features specific to blockchain development, such as address types, gas optimization patterns, and built-in functions for Ethereum operations. The chapter walked through building a simple "Faucet" contract that can receive and dispense ether.

The Ethereum Contract ABI

The Application Binary Interface (ABI) defines how to encode function calls and data for the EVM. It allows applications to interact with contracts by specifying the format of functions, arguments, and return values. The ABI is typically output as a JSON structure during compilation.

## Solidity Programming Features

The chapter covered many Solidity programming features:

- **Data types**: Booleans, integers, addresses, arrays, mappings, structs, etc.
- **Variables and functions**: Global variables like `msg.sender`, `block.number`, and built-in functions
- **Contract definition**: How to define contracts, interfaces, and libraries
- **Function modifiers**: For adding preconditions to functions
- **Inheritance**: How contracts can inherit from other contracts
- **Error handling**: Using assert, require, and revert
- **Events**: For logging information and providing return values to the front end
- **Contract interaction**: How to call other contracts using various methods

Gas Considerations

Gas is a critical constraint in smart contract development. The chapter discussed strategies for minimizing gas consumption, such as avoiding dynamically sized arrays and unnecessary external calls. It also showed how to estimate the gas cost of contract functions.

Smart contract development with Solidity requires understanding not just the language, but also the execution environment, security considerations, and economic implications of your code. This chapter provided the foundation for building effective and efficient smart contracts on Ethereum.

<!-- ### 2025.03.25

TODO: read [chapter 8](https://github.com/ethereumbook/ethereumbook/blob/develop/08smart-contracts-vyper.asciidoc) -->


<!-- Content_END -->
