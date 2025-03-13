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


### 2025.03.12

今天主要分析了联盟链和以太坊共识机制的区别，联盟链在实际应用中做出了一种权衡：牺牲部分完全去中心化的特性，换来了更高的效率、合规性和业务适用性。在这种场景下，外部机制确保的信任实际上与区块链的技术特性相辅相成，共同实现了一个高效、可信和多方协同的数据网络。

### 2025.03.13

**燃料费** 是以太坊执行层根据网络需求设定的计算成本单位，而 **Gwei** 是以太币（ETH）的最小单位之一（$1 Gwei = 10^{−9}\space ETH$）。

当用户在以太坊上执行交易或智能合约时，所需的 **Gas 量** 取决于操作的计算复杂度和存储占用。这部分 Gas 需求是 **固定的**，但 **每单位 Gas 的价格（Gas Price）** 以 Gwei 计价，并会随网络的繁忙程度波动。

最终的 **燃料费（交易费用）** 计算方式如下：

$$交易费用=Gas 消耗量×Gas Price (Gwei)$$

因此，在网络拥堵时，Gas Price 上升，交易费用变高；而在网络空闲时，Gas Price 下降，交易费用变低。在理解这两个概念时要万分注意。



<!-- Content_END -->
