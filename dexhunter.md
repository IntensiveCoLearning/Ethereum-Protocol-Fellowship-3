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




<!-- Content_END -->
