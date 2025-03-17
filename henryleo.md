---
timezone: UTC+8
---

> 请在上边的 timezone 添加你的当地时区(UTC)，这会有助于你的打卡状态的自动化更新，如果没有添加，默认为北京时间 UTC+8 时区


# 你的名字

1. 自我介绍 hi，我是henryleo，生物信息工程师，开放科学/DeSci爱好者，同时也对代币经济学、隐私和区块链在社会治理中的作用很感兴趣。目前专注合成生物学工作。
2. 你认为你会完成本次残酷学习吗？ 我想我可以！
3. 你的联系方式 https://github.com/henrycyberbio/

## Notes

<!-- Content_START -->

### 2025.03.10

#### 用户转账ETH的燃料费
燃料费=基础费用+优先费，这个费用不能低于燃料限额，燃料限额和基础费用由以太坊决定，优先费是给验证者的小费，由用户决定。这意味着用户只需要考虑优先费即可。
##### 第一种方式：完全手动设定

例如，Gas限额=21000Gwei，基础费用为100Gwei，优先费为10Gwei，则用户实际需要支付的费用为：
$$
Gas=GasLimit*(Base+Priority)=21000*(100+10)Gwei=2310000Gwei=0.00231ETH
$$
##### 第二种方式：设定最高收费
maxFeePerGas 最高收费

当用户设定的最高收费大于基础费用与优先费之和，则会退回多余部分：
$$
repay=maxFeePerGas-(BaseFee+Priority)
$$
当最高收费小于基础费用与优先费之和，则需要排队

##### 第三种方式：设定最高优先费
maxPriorityFeePerGas 最高优先费

当使用这种设置时，假设不需要排队，则收费肯定大于需要的费用：
$$maxFeePerGas>(BaseFee+Priority)$$
移项，即，当优先费小于最大费用与基础费用之差时，必然上链：
$$Priority<maxFeePerGas-BaseFee$$
那么为了节约Gas费或避免巨额Gas费的情况，可以设定最高优先费 maxPriorityFeePerGas：
$$Priority=min(maxPriorityFeePerGas, (maxFeePerGas-BaseFee))$$
这个公式意味着当网络不繁忙时（`maxFeePerGas-BaseFee`较小），选择立即成交的优先费用；当网络繁忙时，可以选择预期的最高优先费，开始排队。


### 2025.03.11
上文提及基础费用是以太坊网络决定的，实际上有一个简单的公式，一个区块当前的基础费用是**由上一个区块是否超出了网络最大设定的最大燃料费的一半**来决定，即：
$$
BaseFee\equiv
\left \{
\begin{array}{l}
10^{9}, if\space伦敦升级之前 \\
Gas_{PrevBlock}, if\space Gas_{PrevBlock}=\frac{GasLimit}{2} \\
Gas_{PrevBlock}*(1+A), if \space Gas_{PrevBlock}>\frac{GasLimit}{2} \\
Gas_{PrevBlock}*(1-B), if \space Gas_{PrevBlock}<\frac{GasLimit}{2} \\
\end{array}
\right.
$$
这个方案的动态在于A是可变的，A的范围是`0 ~ 12.5%`、B是`-10% ~ 0`

**已勘误至2025.03.14笔记**


### 2025.03.12

今天主要分析了联盟链和以太坊共识机制的区别，联盟链在实际应用中做出了一种权衡：牺牲部分完全去中心化的特性，换来了更高的效率、合规性和业务适用性。在这种场景下，外部机制确保的信任实际上与区块链的技术特性相辅相成，共同实现了一个高效、可信和多方协同的数据网络。

### 2025.03.13

**燃料费** 是以太坊执行层根据网络需求设定的计算成本单位，而 **Gwei** 是以太币（ETH）的最小单位之一（$1 Gwei = 10^{−9}\space ETH$）。

当用户在以太坊上执行交易或智能合约时，所需的 **Gas 量** 取决于操作的计算复杂度和存储占用。这部分 Gas 需求是 **固定的**，但 **每单位 Gas 的价格（Gas Price）** 以 Gwei 计价，并会随网络的繁忙程度波动。

最终的 **燃料费（交易费用）** 计算方式如下：

$$交易费用=Gas 消耗量×Gas Price (Gwei)$$

因此，在网络拥堵时，Gas Price 上升，交易费用变高；而在网络空闲时，Gas Price 下降，交易费用变低。在理解这两个概念时要万分注意。

### 2025.03.14
在学习了Gas和Gwei的区别后对上述笔记进行修改：

基础费用的单位是Gwei，因此基础费用的变化影响的是燃料费和ETH的“汇率”，而并不影响交互本身需要多少燃料完成。

上文提及基础费用是以太坊网络决定的，实际上有一个简单的公式，一个区块当前的基础费用是**由上一个区块是否超出目标燃料费**来决定，目标燃料费在EIP1559设定为燃料限额的一半，即：
$$
BaseFeePerGas\equiv
\left \{
\begin{array}{l}
10^{9}, if\space伦敦升级之前 \\
BaseFeePerGas_{PrevBlock}, if\space BaseFeePerGas_{PrevBlock}=\frac{GasLimit}{2} \\
BaseFeePerGas_{PrevBlock}*(1+A), if \space BaseFeePerGas_{PrevBlock}>\frac{GasLimit}{2} \\
BaseFeePerGas_{PrevBlock}*(1-A), if \space BaseFeePerGas_{PrevBlock}<\frac{GasLimit}{2} \\
\end{array}
\right.
$$
这个方案的动态在于A是可变的，A的范围是`-12.5 ~ 12.5%``

而A的算法是：
$$
A = \frac{GasUse_{PrevBlock}-Gas_{target}}{Gas_{target}}*12.5\%
$$
A也叫做基础费用的变化幅度。

这个里面有许多可以用于调整以太坊经济模型的参数，如：
- 燃料费限额
- 燃料费目标或弹性系数（$\frac{GasLimit}{2}$，分母也叫弹性系数 Elasticity multiplier，现在是2）
- 基本费用最大分母（$\frac{1}{8}=12.5\%$，这是最极端的情况下，当前区块燃料费为零，那么分子为（燃料费限额-0）/燃料费限额=1。这个分母也可以调整）

#### refs:
- [EIP-1559](https://eips.ethereum.org/EIPS/eip-1559)
- [以太坊Gas费(Ethereum Gas Fees)详细介绍：gas费计算、gas费实时查询、gas 费预测、ethereum gas tracker、ethereum gas charts | 币圈小林子](https://www.youtube.com/watch?v=eMX85xai2-U)
- [燃料和费用|EF](https://ethereum.org/zh/developers/docs/gas/#base-fee)



### 2025.03.16
#### 基本交易成本

常见的基本交易燃料 Intrinsic Gas 分两类：
- **普通转账（ETH 发送）**：21,000 Gas，用于交易验证和签名验证
- **合约调用**：21,000 Gas + 额外计算消耗
- **合约创建**：21,000 Gas + 32000 Gas，用于存储合约代码和初始化状态

其中，转账基础费用记为 $G_{transaction}​=21000Gas$ ，合约创建费用记为 $G_{txCreate}=32000Gas$ 


### 2025.03.17
#### 合约数据的燃料成本
Calldata 是指随交易发送到智能合约的输入数据，Calldata需要占用链上的储存和运输，其每个字节都要消耗燃料。根据其字节的类型消耗量不同：
- 零字节（0x00）：4 Gas
- 非零字节：16 Gas


### 2025.03.18
#### 计算合约执行成本

智能合约的底层操作是通过EVM 操作码执行，每个操作码代表一种特定的操作，各操作码所需燃料费请见：[EVM操作码|EF](https://ethereum.org/zh/developers/docs/evm/opcodes/)

在以太坊中，有两类操作：**计算（Computation）** 和 **存储变更（Storage Modification）** 但它们的 Gas 成本计算方式和对网络的影响大不相同。
##### 计算 Computation
计算指的是 EVM 运行智能合约时执行的算术、逻辑、数据操作等。
- 主要包括：`ADD`、`MUL`、`DIV`、`LT`、`EQ`、`SHA3`、`CALL`、`RETURN` 等指令。
- 这些指令的 Gas 成本通常是 **固定的** 或 **动态计算的（如 EXP）**，但不会对区块链的长期状态产生影响。

##### 存储变更 Storage Modification
主要涉及对合约 storage 存储 变量的读写，即 `SLOAD`读取存储槽 和 `SSTORE`写入存储槽

**存储比计算昂贵得多**，因为存储变更会永久写入以太坊的状态数据库，消耗较多的磁盘 I/O 和同步成本

###### SLOAD
根据EIP-2929，`SLOAD`操作的燃料消耗分为两种：
- Cold Storage Slot 冷储存槽：如果这是本次交易中首次访问的存储槽，SLOAD 操作会消耗 2100 Gas
- Warm Storage Slot 温储存槽：如果该存储槽在本次交易中已被访问过，SLOAD 操作仅消耗 100 Gas

[SLOAD | EVM操作码 | Github](https://github.com/wolflo/evm-opcodes/blob/main/gas.md#a6-sload)
###### SSTORE
同样的，`SSTORE`也会根据冷、温储存槽增减燃料费，如果是冷储存槽则增加2100Gas。[SSTORE | EVM操作码 | Github](https://github.com/wolflo/evm-opcodes/blob/main/gas.md#a7-sstore)

根据储存值，燃料费有以下机制：
- 从 0 改为 非 0：
    - 消耗 20,000 Gas：将存储槽从零值修改为非零值时，需要消耗 20,000 Gas。
- 从 非 0 改为 0：
    - 消耗 5,000 Gas，且在交易执行成功后可退款 15,000 Gas：将存储槽从非零值修改为零值时，初始消耗 5,000 Gas，但由于释放了存储空间，协议设计允许在交易成功执行后返还 15,000 Gas。
- 从 非 0 改为 另一个 非 0：
    - 消耗 5,000 Gas：将存储槽从一个非零值修改为另一个非零值时，消耗 5,000 Gas。

<!-- Content_END -->
