---
timezone: UTC+8
---

> 请在上边的 timezone 添加你的当地时区(UTC)，这会有助于你的打卡状态的自动化更新，如果没有添加，默认为北京时间 UTC+8 时区


# RockyFang

1. 自我介绍: 我叫rocky，來自深圳，目前在一家知名交易所从事后端开发工作，平时的工作也会跟链打交道，希望能在这里学到更多链上相关的知识。感谢提供学习机会。
笔记按内容整理链接：https://github.com/rockyfang2024/Web3Learn/tree/main/ETH%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0
2. 你认为你会完成本次残酷学习吗？   会
3. 你的联系方式（推荐 Telegram）  https://t.me/rockyfang2024
## Notes

<!-- Content_START -->

### 2025.03.10

笔记整理：https://github.com/rockyfang2024/Web3Learn/tree/main/ETH%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0

- 尝试了水龙头的使用
- 了解了ETH节点类型的区别
- 初步使用了Remix创建智能合约
- 初步学习了Solidity开发智能合约的语法

### 2025.03.11

笔记整理：https://github.com/rockyfang2024/Web3Learn/tree/main/ETH%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0

- 申请了MetaMask API的账户，尝试使用ethers.js的API实现获取最新区块号、获取钱包余额、 使用测试网络发起交易的操作
- 找到ETH的测试网的浏览器
- 对比了web3.js 和 ethers.js的区别


### 2025.03.12

笔记整理：https://github.com/rockyfang2024/Web3Learn/tree/main/ETH%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0

- 初步阅读《精通以太坊》书籍的前两章，了解了一些概念：分叉、钱包、图灵完备、提案ERC和EIP的区别、ETH和ETC的关系
  - 计划本周完成第一个智能合约的Demo：按书中的例子，实现一个水龙头的智能合约
- 比较有收获的是，纠正了之前对钱包的理解
  - 一直以为币是存在钱包里的，事实上钱包不存储代币，而是存储公钥和私钥，进而控制链上的资产
  - 由此想到了一个类比：区块链钱包和手机银行APP很相似，本身都不存储钱，而是通过私钥（账号、密码）进而管理钱。
- 学习了助记词、种子、私钥的概念和关系

### 2025.03.13

笔记整理：https://github.com/rockyfang2024/Web3Learn/tree/main/ETH%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0

- 尝试了部署水龙头合约到ETH测试网，并且使用remix调用领取测试币方法，向合约地址转账
- 阅读《精通以太坊》第三、四章，学习了公钥和私钥的关系，以及公钥是私钥通过椭圆曲线算法生成的。

### 2025.03.14

笔记整理：https://github.com/rockyfang2024/Web3Learn/tree/main/ETH%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0

- 阅读了《精通以太坊》第五、六章，学习了交易相关的知识
  - 了解到了交易是如何广播的，以及可以从交易的value和data来判断是转账还是智能合约。
  - 如何销毁ETH

### 2025.03.15

笔记整理：https://github.com/rockyfang2024/Web3Learn/tree/main/ETH%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0

- 阅读了《精通以太坊》第七章，学习智能合约和Solidity

### 2025.03.16

笔记整理：https://github.com/rockyfang2024/Web3Learn/tree/main/ETH%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0

- 快速浏览《精通以太坊》第九到十二章，了解了DAPP、智能合约的安全性、代币

### 2025.03.17

笔记整理：https://github.com/rockyfang2024/Web3Learn/tree/main/ETH%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0

- 学习Gas 是如何计算的，为什么 Gas 会波动，如何创建 Gas 高效的智能合约
- 开始研究EVM官方文档 https://ethereum.org/zh/developers/docs/evm/

### 2025.03.18

笔记整理：https://github.com/rockyfang2024/Web3Learn/tree/main/ETH%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0

- 学习EVM的世界状态
- 了解到了EVM每次执行一个合约或交易都会创建一个虚拟机实例，并且EVM是基于栈的虚拟机，最大深度为1024。若一个合约的调用深度超过1024，会执行失败。


### 2025.03.19

笔记整理：https://github.com/rockyfang2024/Web3Learn/tree/main/ETH%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0

- 学习到了交易可能失败的场景
- 了解了GasLimit

### 2025.03.20

笔记整理：https://github.com/rockyfang2024/Web3Learn/tree/main/ETH%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0

- 学习了ETH的共识层和执行层的概念及作用
- 了解信标链的概念
- 了解一币多链和跨链操作

### 2025.03.21

笔记整理：https://github.com/rockyfang2024/Web3Learn/tree/main/ETH%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0

- 继续学习智能合约solidity
- 学习ETH和layer2

### 2025.03.22

笔记整理：https://github.com/rockyfang2024/Web3Learn/tree/main/ETH%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0

- 继续学习智能合约solidity

### 2025.03.23

笔记整理：https://github.com/rockyfang2024/Web3Learn/tree/main/ETH%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0

- 学习智能合约solidity的一些入门程序


### 2025.03.24

笔记整理：https://github.com/rockyfang2024/Web3Learn/tree/main/ETH%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0

- 学习solidity高级数据结构和一些函数的用法

### 2025.03.25

笔记整理：https://github.com/rockyfang2024/Web3Learn/tree/main/ETH%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0

- 了解了多签钱包，和Bybit被盗事件的过程
- 学习了 CALL 和 DELEGATECALL的区别

### 2025.03.26

笔记整理：https://github.com/rockyfang2024/Web3Learn/tree/main/ETH%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0

- 学习go-ethereum中core包下的代码
  - blockchain.go
  - rawdb.go

### 2025.03.27

笔记整理：https://github.com/rockyfang2024/Web3Learn/tree/main/ETH%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0

- 学习go-ethereum中core包下的代码
  - transaction_pool.go

<!-- Content_END -->
