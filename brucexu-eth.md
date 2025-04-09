---
timezone: UTC+13
---

# 你的名字

## 自我介绍

**Bruce**，以太坊狂战士，密码朋克传销专家，极度残酷无情地人行 LLM 建设者。

## 你认为你会完成本次残酷学习吗？

✅ **那必须的！**

## 你的联系方式（推荐 Telegram）

- 📬 **Telegram**：[t.me/brucexu_eth](https://t.me/brucexu_eth)
- 🐦 **Twitter/X**：[x.com/brucexu_eth](https://x.com/brucexu_eth)

---

# Notes

<!-- Content_START -->

# 2025.03.11

## Test https://epf.wiki/#/eps/week4

测试还是很关键的，如果几个客户端输出的不一致，就会出现分叉了。一般测试需要构造一些参数和内容，比如：

- Pre-state： accounts 和 balances、nonces、code 和 storage
- Environment：timestamp、previous RANDAO、block number 等等
- Txs：具体的 tx messages
- Post-state：预期的 state or accounts

Fuzzy state testing，可以创建随机的 smart contract code 来测试可能出现的意外情况。

上面主要是 EVM 测试。Blockchain 的测试就是多加一些 blocks，然后检查 chain head 和 post-state。需要添加一些 invalid block 然后测试失败的情况。

通用的测试用例在这里：https://github.com/ethereum/tests 测试的工具是：https://github.com/ethereum/retesteth。

一些客户端会引用这些测试用例到自己的仓库进行测试，例如：`go-ethereum (Go): Docs, Test location: tests/testdata`。

![image](https://github.com/user-attachments/assets/a78bb472-d1b7-4242-a7bc-04ad589cc451)

基本上就是一些 data 相关的，可能是从真实区块链上面拿到的，然后放在这里用于做单元测试使用。

TODO 看到 ethereum/execution-spec-tests 了 Week.4.EPFsg.Test.Security.Overview.pdf

# 2025.03.12

## 深度看看 Pectra 升级 https://gist.github.com/Thegaram/64b5d43144d5740f01907b48d986b8e7

一般通过类似这样的 EIP-7600 来整理和汇总包含进来的升级，比如这个是在 2024-01-18 创建的，等待了一年 https://eips.ethereum.org/EIPS/eip-7600 准备 last call 了。

### EVM

- EIP-7702: Set EOA account code (https://eips.ethereum.org/EIPS/eip-7702)
  - Add a new tx type that sets the code for an EOA 支持通过 EOA 发送
- EIP-2537: Precompile for BLS12-381 curve operations (https://eips.ethereum.org/EIPS/eip-2537)
- EIP-2935: Save historical block hashes in state (https://eips.ethereum.org/EIPS/eip-2935)

### Blobs & Calldata

- EIP-7691: Blob throughput increase (https://eips.ethereum.org/EIPS/eip-7691)
- EIP-7840: Add blob schedule to EL config files (https://eips.ethereum.org/EIPS/eip-7840)
- EIP-7623: Increase calldata cost (https://eips.ethereum.org/EIPS/eip-7623)

### CL & Staking

- EIP-7685: General purpose execution layer requests (https://eips.ethereum.org/EIPS/eip-7685)
- EIP-6110: Supply validator deposits on chain (https://eips.ethereum.org/EIPS/eip-6110)
- EIP-7002: Execution layer triggerable exits (https://eips.ethereum.org/EIPS/eip-7002)
- EIP-7251: Increase the MAX_EFFECTIVE_BALANCE (https://eips.ethereum.org/EIPS/eip-7251)
- EIP-7549: Move committee index outside Attestation (https://eips.ethereum.org/EIPS/eip-7549)

### EIP-7702 Notes

```
# At first, account is an EOA with no code
> cast code "$alice"
0x

# Set delegated code
> cast send --from "$alice" --private-key "$alice_pk" --auth "$multicall" $(cast az)

# Account now has a special code delegation, stored persistently in state
> cast code "$alice"
0xef0100ca11bde05977b3631167028862be2a173976ca11
```

0xef0100 这是一个特殊前缀，用来表示"给 EOA 地址写入 delegated code"的操作。EIP-7702 让包含这一前缀的 code 被解释成"这段地址在执行时要将调用委托给后面指定的合约"。

MultiCall 是一种智能合约功能，允许在一个交易中执行多个合约调用。

```
> calls=()

> calldata=$(cast calldata "approve(address,uint256)" "$erc20_gateway" "$amount")
> calls+=("($usdt, false, 0, $calldata)")

> calldata=$(cast calldata "depositERC20(address,uint256,uint256)" "$usdt" "$amount" "$gas")
> calls+=("($erc20_gateway, false, $fee, $calldata)")

# ...

> cast send "$alice" "aggregate3Value((address,bool,uint256,bytes)[] calldata calls)" \
    "[${calls[1]}, ${calls[2]}}]" \
     --value $fee \
    --from "$alice" --unlocked
```

之后就可以合并成一个 call 去触发。不然需要分开多个 EOA 签名和 call。

执行以下相关命令进一步测试（https://gist.github.com/Thegaram/51a49a829b2a13ebc88990894a30912b）：

```
# set to an L1 node RPC
eth_rpc="https://eth.llamarpc.com"

# run 7702-enabled local forked network
anvil --fork-url "$eth_rpc" --auto-impersonate --odyssey
```

新开一个窗口：

```
# Anvil dev account
alice="0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266"
alice_pk="0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"

# contracts
usdt="0xdac17f958d2ee523a2206206994597c13d831ec7"
multicall="0xcA11bde05977b3631167028862bE2a173976CA11"
scroll_erc20_gateway="0xd8a791fe2be73eb6e6cf1eb0cb3f36adc9b3f8f9"
scroll_message_queue="0x0d7E906BD9cAFa154b048cFa766Cc1E54E39AF9B"

# message configs
amount="1"
gas_limit="200000"

# estimate cross-layer message fee
fee=$(cast call "$scroll_message_queue" "estimateCrossDomainMessageFee(uint256 _gasLimit)(uint256)" "$gas_limit" --json | jq -r '.[]')

# pre-fund account with ETH
cast rpc anvil_setBalance "$alice" 10000000000000000000

# pre-fund account with USDT
export issuer=$(cast call "$usdt" "owner()(address)")
cast send --from "$issuer" --unlocked "$usdt" "transfer(address to, uint value)" "$alice" "$(( 10 * $amount ))"


```

输出：

```
blockHash            0xab049cfa97ad8326a73c54794562b309ee87d6f7642f9c907c9eaa197665b23d
blockNumber          22028120
contractAddress
cumulativeGasUsed    63173
effectiveGasPrice    573246114
from                 0xC6CDE7C39eB2f0F0095F41570af89eFC2C1Ea828
gasUsed              63173
logs                 [{"address":"0xdac17f958d2ee523a2206206994597c13d831ec7","topics":["0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef","0x000000000000000000000000c6cde7c39eb2f0f0095f41570af89efc2c1ea828","0x000000000000000000000000f39fd6e51aad88f6f4ce6ab8827279cfffb92266"],"data":"0x000000000000000000000000000000000000000000000000000000000000000a","blockHash":"0xab049cfa97ad8326a73c54794562b309ee87d6f7642f9c907c9eaa197665b23d","blockNumber":"0x1501f58","blockTimestamp":"0x67d100e1","transactionHash":"0x47c0198c5beae7286e3ee98e9f8cc39c0768b66e79ec16e689ec65ea2ba1828b","transactionIndex":"0x0","logIndex":"0x0","removed":false}]
logsBloom            0x0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001000000000000000000000400000000000000000000000000000000000800000000000004000000000000000000000000000000000020000000000000010000000000000000000000000000010000000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000080000000000000000000000000000000000000000000000002000000200000000000000000000000002000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
root
status               1 (success)
transactionHash      0x47c0198c5beae7286e3ee98e9f8cc39c0768b66e79ec16e689ec65ea2ba1828b
transactionIndex     0
type                 2
blobGasPrice         5663462761
blobGasUsed
authorizationList
to                   0xdAC17F958D2ee523a2206206994597C13D831ec7
```

TODO 跑到 Deposit without EIP-7702 https://gist.github.com/Thegaram/51a49a829b2a13ebc88990894a30912b https://gist.github.com/Thegaram/64b5d43144d5740f01907b48d986b8e7

# 2025.03.13

Deposit without EIP-7702 这是默认的方式，需要两笔 tx

```
# approve token spending
cast send --from "$alice" --unlocked "$usdt" "approve(address spender, uint256 value)" "$scroll_erc20_gateway" "$amount"

# deposit USDT to Scroll
cast send --from "$alice" --unlocked "$scroll_erc20_gateway" "depositERC20(address _token,uint256 _amount,uint256 _gasLimit)" "$usdt" "$amount" "$gas_limit" --value $fee

# (optional) reset token approval
cast send --from "$alice" --unlocked "$usdt" "approve(address spender, uint256 value)" "$scroll_erc20_gateway" "0"
```

deposit 返回：

```
blockHash            0xa617f41cbc6a9e22a8e7d21c82774847ac5fef0a284d85f55ba64f98350e5332
blockNumber          22037056
contractAddress
如果是创建合约交易，交易完成后会在这里放上新合约地址
cumulativeGasUsed    199263
effectiveGasPrice    518744790
from                 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
gasUsed              199263
本次交易的实际 ETH 消耗：gasUsed * effectiveGasPrice
logs                 [{
    "address": "0xdac17f958d2ee523a2206206994597c13d831ec7",
    "topics": [
      "0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef",
      "0x000000000000000000000000f39fd6e51aad88f6f4ce6ab8827279cfffb92266",
      "0x000000000000000000000000d8a791fe2be73eb6e6cf1eb0cb3f36adc9b3f8f9"
    ],
    "data": "0x0000000000000000000000000000000000000000000000000000000000000001",
    "blockHash": "0xa617f41cbc6a9e22a8e7d21c82774847ac5fef0a284d85f55ba64f98350e5332",
    "blockNumber": "0x1504240",
    "blockTimestamp": "0x67d2a5a5",
    "transactionHash": "0x86daea8ed7145a3abebc843303485e4be5317ab1d6e372840abaab9dd3bfe369",
    "transactionIndex": "0x0",
    "logIndex": "0x0",
    "removed": false
  }]
交易过程中 emit 的 log，每条字段包括：
address：触发该日志的合约地址。
topics：主题数组（最多 4 个元素）。第一个主题常常是事件的哈希签名，后面可以是根据 indexed 关键字索引的参数。
data：非索引的数据（事件里没有 indexed 修饰的参数会放在这里）。
blockHash / blockNumber / transactionHash / transactionIndex / logIndex：表明该 Log 是在哪个区块、哪笔交易、第几条日志。
removed：是否由于链回滚（reorg）导致该日志被移除，一般情况是 false。
用途：
DApp 前端可以通过过滤日志来获取智能合约事件（如 ERC20 的 Transfer 事件）。
事件是链上与外界交互（例如前端监听）的重要手段。

logsBloom            0x000000000010004000000000080000000400000000000000000000000000000000000000000000800200000000000100000000200000000000000000000000000000000000000000000000080021000000800000000004400000000000200001400000000000001000000001000000010000002000000000008004102000000000000000000000000000000000000000000020000000000000000000021000001040000000000800000000c0000000000000060000000000080000000000004000000042000004200000021001004000000000122400000000004000800000000000000000000000000000810000000000000000000020000000000000000000
根据 log 生成的 256 字节布隆过滤器，方便节点快速判断一个区块是否有包含想要的日志，先粗筛，然后再查看
root                 不用了，以前用于判断交易执行结果
status               1 (success)
transactionHash      0x86daea8ed7145a3abebc843303485e4be5317ab1d6e372840abaab9dd3bfe369
transactionIndex     0
type                 2
blobGasPrice         860283508
EIP-4844 新增的 blob 数据的 Gas 价格
blobGasUsed
存储 blob 花费的资源量，如果不包括 blob 就是 0
authorizationList
to                   0xD8A791fE2bE73eb6E6cF1eb0cb3F36adC9B3F8f9
```

Deposit with EIP-7702:

```
# check code without delegation, should return 0x
cast code "$alice"

# execute 7702 delegation
cast send --from "$alice" --private-key "$alice_pk" --auth "$multicall" $(cast az)

这里 --auth "$multicall" 的合约是一个通用合约，是可以执行 multicall 的一个合约，上面这个意思是让 alice 地址支持使用这个 multicall 合约？但是为什么有了 code？而且合约还是 usdt 的？

# check code again, should return 0xef0100dac17f958d2ee523a2206206994597c13d831ec7
cast code "$alice"

# construct sequence of calls
calls=()

calldata=$(cast calldata "approve(address spender, uint256 value)" "$scroll_erc20_gateway" "$amount")
calls+=("($usdt, false, 0, $calldata)")

calldata=$(cast calldata "depositERC20(address _token,uint256 _amount,uint256 _gasLimit)" "$usdt" "$amount" "$gas_limit")
calls+=("($scroll_erc20_gateway, false, $fee, $calldata)")

calldata=$(cast calldata "approve(address spender, uint256 value)" "$scroll_erc20_gateway" "0")
calls+=("($usdt, false, 0, $calldata)")

calls 参数：
(0xdac17f958d2ee523a2206206994597c13d831ec7, false, 0, 0x095ea7b3000000000000000000000000d8a791fe2be73eb6e6cf1eb0cb3f36adc9b3f8f90000000000000000000000000000000000000000000000000000000000000001) (0xd8a791fe2be73eb6e6cf1eb0cb3f36adc9b3f8f9, false, 8106926800000, 0x21425ee0000000000000000000000000dac17f958d2ee523a2206206994597c13d831ec700000000000000000000000000000000000000000000000000000000000000010000000000000000000000000000000000000000000000000000000000030d40) (0xdac17f958d2ee523a2206206994597c13d831ec7, false, 0, 0x095ea7b3000000000000000000000000d8a791fe2be73eb6e6cf1eb0cb3f36adc9b3f8f90000000000000000000000000000000000000000000000000000000000000000)

# send in a single tx
cast send --from "$alice" --unlocked "$alice" --value $fee "aggregate3Value((address,bool,uint256,bytes)[] calldata calls)" "[${calls[1]}, ${calls[2]}, ${calls[3]}]"

一个 tx 就搞定了

blockHash            0x144c52392c4b5b392c40e6905347c28b05fe1bd4ea0c852d7db38b83f1687cd6
blockNumber          22037059
contractAddress
cumulativeGasUsed    226286
effectiveGasPrice    362684203
from                 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
gasUsed              226286
logs                 [{"address":"0xdac17f958d2ee523a2206206994597c13d831ec7","topics":["0x8c5be1e5ebec7d5bd14f71427d1e84f3dd0314c0f7b2291e5b200ac8c7c3b925","0x000000000000000000000000f39fd6e51aad88f6f4ce6ab8827279cfffb92266","0x000000000000000000000000d8a791fe2be73eb6e6cf1eb0cb3f36adc9b3f8f9"],"data":"0x0000000000000000000000000000000000000000000000000000000000000001","blockHash":"0x144c52392c4b5b392c40e6905347c28b05fe1bd4ea0c852d7db38b83f1687cd6","blockNumber":"0x1504243","blockTimestamp":"0x67d2aa5b","transactionHash":"0x7e6baa246f9515abe8e9934291b12aab7e030a04e12a632fe054aaf9d61832b3","transactionIndex":"0x0","logIndex":"0x0","removed":false},{"address":"0xdac17f958d2ee523a2206206994597c13d831ec7","topics":["0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef","0x000000000000000000000000f39fd6e51aad88f6f4ce6ab8827279cfffb92266","0x000000000000000000000000d8a791fe2be73eb6e6cf1eb0cb3f36adc9b3f8f9"],"data":"0x0000000000000000000000000000000000000000000000000000000000000001","blockHash":"0x144c52392c4b5b392c40e6905347c28b05fe1bd4ea0c852d7db38b83f1687cd6","blockNumber":"0x1504243","blockTimestamp":"0x67d2aa5b","transactionHash":"0x7e6baa246f9515abe8e9934291b12aab7e030a04e12a632fe054aaf9d61832b3","transactionIndex":"0x0","logIndex":"0x1","removed":false},{"address":"0x8fa3b4570b4c96f8036c13b64971ba65867eeb48","topics":["0x3d0ce9bfc3ed7d6862dbb28b2dea94561fe714a1b4d019aa8af39730d1ad7c3d","0x0000000000000000000000006774bcbd5cecef1336b5300fb5186a12ddd8b367"],"data":"0x0000000000000000000000000000000000000000000000000000075f8a7dfc80","blockHash":"0x144c52392c4b5b392c40e6905347c28b05fe1bd4ea0c852d7db38b83f1687cd6","blockNumber":"0x1504243","blockTimestamp":"0x67d2aa5b","transactionHash":"0x7e6baa246f9515abe8e9934291b12aab7e030a04e12a632fe054aaf9d61832b3","transactionIndex":"0x0","logIndex":"0x2","removed":false},{"address":"0x0d7e906bd9cafa154b048cfa766cc1e54e39af9b","topics":["0x69cfcb8e6d4192b8aba9902243912587f37e550d75c1fa801491fce26717f37e","0x0000000000000000000000007885bcbd5cecef1336b5300fb5186a12ddd8c478","0x000000000000000000000000781e90f1c8fc4611c9b7497c3b47f99ef6969cbc"],"data":"0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000e87b5000000000000000000000000000000000000000000000000000000000030d40000000000000000000000000000000000000000000000000000000000000008000000000000000000000000000000000000000000000000000000000000002248ef1332e000000000000000000000000d8a791fe2be73eb6e6cf1eb0cb3f36adc9b3f8f9000000000000000000000000e2b4795039517653c5ae8c2a9bfdd783b48f447a000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000e87b500000000000000000000000000000000000000000000000000000000000000a000000000000000000000000000000000000000000000000000000000000001448431f5c1000000000000000000000000dac17f958d2ee523a2206206994597c13d831ec7000000000000000000000000f55bec9cafdbe8730f096aa55dad6d22d44099df000000000000000000000000f39fd6e51aad88f6f4ce6ab8827279cfffb92266000000000000000000000000f39fd6e51aad88f6f4ce6ab8827279cfffb92266000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000c000000000000000000000000000000000000000000000000000000000000000600000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000","blockHash":"0x144c52392c4b5b392c40e6905347c28b05fe1bd4ea0c852d7db38b83f1687cd6","blockNumber":"0x1504243","blockTimestamp":"0x67d2aa5b","transactionHash":"0x7e6baa246f9515abe8e9934291b12aab7e030a04e12a632fe054aaf9d61832b3","transactionIndex":"0x0","logIndex":"0x3","removed":false},{"address":"0x6774bcbd5cecef1336b5300fb5186a12ddd8b367","topics":["0x104371f3b442861a2a7b82a070afbbaab748bb13757bf47769e170e37809ec1e","0x000000000000000000000000d8a791fe2be73eb6e6cf1eb0cb3f36adc9b3f8f9","0x000000000000000000000000e2b4795039517653c5ae8c2a9bfdd783b48f447a"],"data":"0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000e87b5000000000000000000000000000000000000000000000000000000000030d40000000000000000000000000000000000000000000000000000000000000008000000000000000000000000000000000000000000000000000000000000001448431f5c1000000000000000000000000dac17f958d2ee523a2206206994597c13d831ec7000000000000000000000000f55bec9cafdbe8730f096aa55dad6d22d44099df000000000000000000000000f39fd6e51aad88f6f4ce6ab8827279cfffb92266000000000000000000000000f39fd6e51aad88f6f4ce6ab8827279cfffb92266000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000c0000000000000000000000000000000000000000000000000000000000000006000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000040000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000","blockHash":"0x144c52392c4b5b392c40e6905347c28b05fe1bd4ea0c852d7db38b83f1687cd6","blockNumber":"0x1504243","blockTimestamp":"0x67d2aa5b","transactionHash":"0x7e6baa246f9515abe8e9934291b12aab7e030a04e12a632fe054aaf9d61832b3","transactionIndex":"0x0","logIndex":"0x4","removed":false},{"address":"0xd8a791fe2be73eb6e6cf1eb0cb3f36adc9b3f8f9","topics":["0x31cd3b976e4d654022bf95c68a2ce53f1d5d94afabe0454d2832208eeb40af25","0x000000000000000000000000dac17f958d2ee523a2206206994597c13d831ec7","0x000000000000000000000000f55bec9cafdbe8730f096aa55dad6d22d44099df","0x000000000000000000000000f39fd6e51aad88f6f4ce6ab8827279cfffb92266"],"data":"0x000000000000000000000000f39fd6e51aad88f6f4ce6ab8827279cfffb92266000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000600000000000000000000000000000000000000000000000000000000000000000","blockHash":"0x144c52392c4b5b392c40e6905347c28b05fe1bd4ea0c852d7db38b83f1687cd6","blockNumber":"0x1504243","blockTimestamp":"0x67d2aa5b","transactionHash":"0x7e6baa246f9515abe8e9934291b12aab7e030a04e12a632fe054aaf9d61832b3","transactionIndex":"0x0","logIndex":"0x5","removed":false},{"address":"0xdac17f958d2ee523a2206206994597c13d831ec7","topics":["0x8c5be1e5ebec7d5bd14f71427d1e84f3dd0314c0f7b2291e5b200ac8c7c3b925","0x000000000000000000000000f39fd6e51aad88f6f4ce6ab8827279cfffb92266","0x000000000000000000000000d8a791fe2be73eb6e6cf1eb0cb3f36adc9b3f8f9"],"data":"0x0000000000000000000000000000000000000000000000000000000000000000","blockHash":"0x144c52392c4b5b392c40e6905347c28b05fe1bd4ea0c852d7db38b83f1687cd6","blockNumber":"0x1504243","blockTimestamp":"0x67d2aa5b","transactionHash":"0x7e6baa246f9515abe8e9934291b12aab7e030a04e12a632fe054aaf9d61832b3","transactionIndex":"0x0","logIndex":"0x6","removed":false}]
logsBloom            0x000000000010004000000000080000000400000000000000000000000000000000000000000000800200000000000100000000200000000000000000002000000000000000000000000000080021000000800000000004400000000000200001400000000000001000000001000000010000002000000000008004102000000000000000000000000000000000000000000020000000000000000000021000001240000000000800000000c0000000000000060000000000080000000000004000000042000004200000021001004000000000122400000000004000800000000010000000000000000000810000000000000000000020000000000000000000
root
status               1 (success)
transactionHash      0x7e6baa246f9515abe8e9934291b12aab7e030a04e12a632fe054aaf9d61832b3
transactionIndex     0
type                 2
blobGasPrice         604204580
blobGasUsed
authorizationList
to                   0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
```

# 2025.03.14

## Deep Dive into Pectra EIPs https://gist.github.com/Thegaram/64b5d43144d5740f01907b48d986b8e7

EIP-7702 defines a new transaction type:

```
type SetCodeTx struct {
    ChainID    uint64
    Nonce      uint64
    GasTipCap  *uint256.Int
    GasFeeCap  *uint256.Int
    Gas        uint64
    To         common.Address
    Value      *uint256.Int
    Data       []byte
    AccessList AccessList
    AuthList   []SetCodeAuthorization // <-- this is new

    V *uint256.Int `json:"v" gencodec:"required"`
    R *uint256.Int `json:"r" gencodec:"required"`
    S *uint256.Int `json:"s" gencodec:"required"`
}
```

通过这种 tx type 可以将 SetCodeAuthorization 加入到 EOA 地址上面。

Debugging in go-ethereum and checking how EIP 7702 work.

但是生成的测试用例有点问题，所以没有完成。

# 2025.03.15

## 继续 deep dive 7702 go-ethereum implementation

```
aa: { // The address 0xAAAA calls into addr2
				Code:    program.New().Call(nil, addr2, 1, 0, 0, 0, 0).Bytes(),
				Nonce:   0,
				Balance: big.NewInt(0),
			},
```

program.New().Call is a convenience function to make a call, and convert to the bytecode:

```
program.New().Call(nil, addr2, 1, 0, 0, 0, 0).Bytes() 实际输出：
"`\x00`\x00`\x00`\x00`\x01sp<K+\xd7\f\x16\x9fW\x17\x10\x1c\xae\xe5C)\x9f\xc9F\xc7Z\xf1"
```

You could install ethereum on mac for accessing evm cli to run some evm code:

```
brew tap ethereum/ethereum
brew install ethereum
```

It will install https://geth.ethereum.org/docs but haven't check how to use it.

```
key1, _ = crypto.HexToECDSA("b71c71a67e1177ad4e901695e1b4b9ee17ae16c6668d313eac2f96dbcda3f291")

Output:
*crypto/ecdsa.PrivateKey {
    // PublicKey is using secp256k1 Curve algorithm with X and Y
    // addr := crypto.PubkeyToAddress(key1.PublicKey) -> common.Address [113,86,43,113,153,152,115,219,91,40,109,249,87,175,25,158,201,70,23,247]
    PublicKey: crypto/ecdsa.PublicKey {
        Curve: crypto/elliptic.Curve(*github.com/ethereum/go-ethereum/crypto/secp256k1.BitCurve),
        X: *(*"math/big.Int")(0x1400030ada0),
        Y: *(*"math/big.Int")(0x1400030adc0),
    },
    // decimal format of private key
    D: *math/big.Int {
        neg: false,
        abs: math/big.nat len: 4, cap: 8,
        [12407301369221083793,1706326350199861566,5661049564597369326,13194545968182294445]
    }
}
```

TODO Debug func SignSetCode(prv \*ecdsa.PrivateKey, auth SetCodeAuthorization) (SetCodeAuthorization, error) {

# 2025.03.16

## How does ECDSA work?

https://www.instructables.com/Understanding-how-ECDSA-protects-your-data/

ECDSA sthands for Elliptic Curve Digital Signature Algorithm.

ECDSA is used for generating a signature for data and no one can fake the signature. The receiver can verify the data with the signature. It is not an encryption algorithm like AES.

Curve and Algorithm are two important parts of ECDSA.

The process of generating a private key and public key: Select a predefined elliptic curve -> pick a random point on the curve (base point) -> generate a random number (private key) -> magical_equation(pk, base point) = public key.

The process of signing a file: sign(private key, hash of the file) = signature -> The signature consists of two components: R and S

The process of verifying a file: verify(public key, S) = R' -> compare if R' = R to verify if the signature is correct -> to verify the signer owns private key

# 2025.03.18

https://www.instructables.com/Understanding-how-ECDSA-protects-your-data/

ECDSA uses only integer mathematics, there are no floating points.

Computers use 'bits' to represent data, 4 bits can represent values 0 to 15. ECDSA will use 160 bits total, a very huge number. The underlaying mathematical construct is modulus.

ECDSA is used with a SHA1 cryptographic hash, with SHA1 hash algorithm, it will always be 20 bytes (160 bits). For a given file, SHA1 hash it and get 20 bytes, ECDSA signs the hash, if the data changes, the hash changes, and the signature isn't valid anymore.

The Elliptic Curve cryptography is based on an equation of the form: y^2 = (x^3 + a \* x + b) mod p

The ECDSA equation gives us a curve with a finite number of valid points on it (N) because the Y axis is bound by the modulus (p) and needs to be a perfect square (y^2) with a symmetry on the X axis. We have a total of N/2 possible, valid x coordinates without forgetting that N < p.

![image](https://github.com/user-attachments/assets/ee6b687b-6df1-4b76-a617-96183411e95e)

TODO Step 7: Point Addition

# 2025.03.19

Point Addition 的定义中，曲线上任选 P 和 Q 划线，可以得到跟曲线相交的 R 点，R 的 x 轴对称点是 -S，然后将定义表示为：P + Q = -S，这个就是 Point Addition。注意这里的 + 并不是坐标的相加。

Point multiplication 就是将多个 P 相加，例如 P + P + P 就是 k\*P，然后计算规则 P 点跟曲线的相交点根据 X 轴反转，然后继续。

![image](https://github.com/user-attachments/assets/9aae3d5c-1d84-4e1a-89f6-0aedaf392c18)

这个曲线的斜率和相交点的定义是怎么来的？同一个 P 点，为什么会落在这个地方？是最近的吗？还是？

如果 P 和 Q 不一样，则是连起来。如果 P = Q，则是切线，就是点垂直于线的切线。

The Trap Door Function。如果有一个点 R = k\*P，你可能知道 R 和 P，但是你不可能知道 k，因为是不可逆的。k 的选值通常是一个 256 位或者更大的值，如果你知道 k，那么可以通过高效算法快速生成 R，如果你不知道，遍历爆破需要的时间可能会很长。由此保护 k 的安全性。

看来基本上密码学都是靠不可逆的算法来保护某一个关键信息和变量。

```
// 简单模拟的点加法，群单位设为 {x: 0, y: 0}
function addPoints(P, Q) {
  return { x: P.x + Q.x, y: P.y + Q.y };
}

// 暴力法：依次加 k 次 P
function bruteForceMultiply(P, k) {
  let R = { x: 0, y: 0 }; // 单位元素
  for (let i = 0; i < k; i++) {
    R = addPoints(R, P);
  }
  return R;
}

// 双倍加法算法：使用二进制展开法
function doubleAndAdd(P, k) {
  let R = { x: 0, y: 0 }; // 定义单位元素
  let Q = P;
  let n = k;  // 使用局部变量保存 k 的值，避免修改原始 k
  while (n > 0) {
    if (n & 1) {
      R = addPoints(R, Q);
    }
    Q = addPoints(Q, Q); // 点翻倍
    n = n >> 1;
  }
  return R;
}

// 测试参数
const P = { x: 1, y: 1 };
const k = 100000000;

// 测试暴力方法
console.time("BruteForce");
const resultBruteForce = bruteForceMultiply(P, k);
console.timeEnd("BruteForce");

// 测试双倍加法方法
console.time("DoubleAndAdd");
const resultDoubleAndAdd = doubleAndAdd(P, k);
console.timeEnd("DoubleAndAdd");

console.log("BruteForce Result:", resultBruteForce);
console.log("DoubleAndAdd Result:", resultDoubleAndAdd);
```

上面是一段模拟的椭圆曲线，然后设置 k = 100000000，暴力计算和双倍加法的时间差异巨大：

![image](https://github.com/user-attachments/assets/536012b5-9d85-42aa-bb74-a689ac690a2a)

现实情况中 k 的值通常为 256 位以上！刚刚运算的只有 9 位。这种即使知道原始点和目标点，也无法找到被乘数的情况，是 ECDSA 算法安全性的基础，这一原理被称为"单向陷门函数"。

# 2025.03.20

ECDSA 的几个主要参数有 a, b, p, N and G。N 是曲线上所有的点，然后 G 是某一个随机点，作为 reference point 或者基点。

私钥是一个随机生成的数字，160 位的数字。

160 bit 指的是二进制下 160 位长度，也就意味着数字的范围是 0 ≤ X ≤ 2 ^ 160 − 1 之间。然后为了简单，通常使用十六进制表示。

bit、byte、KB、kb、字符串长度、二进制、十六进制、ASCII、UTF-8 编码，傻傻分不清？我帮你重新捋一下，彻底搞清楚：

- bit 是计算机系统的最小存储单位，表示二进制中的 0 或 1，1 bit = 0 or 1，无法存储太多信息
- byte 是字节，由于 0 和 1 无法存储更多信息，使用比较低效，所以计算机系统使用 byte 来存储更大的数据，所以 byte 将合并多位 bits 来表示更多信息，1 byte = 8 bits。这样就意味着 1 byte 可以表示 2^8 = 256 中不同的数值，从 0000 0000 = 0 到 1111 1111 = 255。比如存储 202 这个数字，只需要 1 byte 存储单元，存储 11001010 这 8 位 bits 即可。

好了，以上是计算机存储领域的概念，其他概念是不同 level 的。下面是二进制和十六进制：

- 计算机最底层存储是二进制，但是二进制表示数据很长，为了更短的展示，一般使用十六进制进行压缩。十六进制的一位数字 0-F，按照十进制是 0 - 15，按照二进制是 0000 - 1111，也就是说 1 位十六进制，需要 4 位二进制表示。按照存储的概念，就是 1 位十六进制数据 = 4 bits 存储，2 位十六进制数据 = 1 bytes。

下面进入编码和字符串 level 的概念：

- 已知人类的文字不仅仅包含数字，还有 A、B、C... 字母和海量的特殊字符和中文汉字，所以设计出来了编码系统来存储汉字，基本原理就是设计一个映射表，编写内容时按照统一的映射表写入存储，解析读取的时候，按照映射表渲染和展示。因此编码系统必须一致，否则会看到乱码。
- ASCII 是比较早期的编码，美国那边搞得，一开始觉得全世界用于计算机的字符只有数字 + 字母 a-zA-Z 和标点等，发现所有这些字符累加都不会超过 128 个，所以就使用了 7 bits 来表示 1 位字符，后来扩展成立 1 byte。也就是说，在 ASCII 编码下面一个字母 `a` 存储使用的编码 `0110 0001`，是 1 byte 8 bits。
- 很明显 ASCII 的编码是不够用的，光常用汉字都有几千个。所以目前主要使用 UTF-8 编码，一个可变长度字符编码，对于 ASCII 还是 8 bits，对于汉字就是 3 bytes = 24 bits，最长是 4 bytes。这样既可以保证尽量小的存储空间，又可以扩展到非常多的字符、emoji、语言。
- 字符串是基于编码的，然后存储在计算机系统里面，字符串的长度跟 Bytes 是两个维度的事情。例如："Hello 世界" 这个字符串长度为 7，但在 UTF-8 编码下面，一个英文是 1 byte 一个汉字是 3 bytes，一共是 11 bytes，所以在计算机存储上占用 11 bytes 的存储空间

计算机的概念 + 翻译，有时候会导致重名或者类似，理解时需要注意区分范围，KB 和 kb 就是两个不一样的单位，重点在于 B（Byte 大）和 b（bit 小）：

- KB 是存储方面的单位，是 Kilobyte，1 KB = 1024 Bytes
- kb 则是网络传输的单位，是 Kilobit，1 kb = 1000 bit

所以我们在装宽带的时候，网速的表示方法是 Mbps 这里面的 b 是小写的，就是 million bits per second。然后下载文件的时候，可能会使用 xx MB/s，这里的会用大 B，因为表示的是 Data 传输的内容，变成了存储领域的 Byte 了，因此最高文件下载速度大概是 Mbps / 8，因为 1 byte = 8 bits。

应该搞清楚了吧，下面看几个实际 cases 加深理解：

1. 160 bit 二进制使用十六进制表示会有多少位数字？因为 1 位十六进制可以表示 4 位二进制，所以 160 bit 二进制会有 160 位数字，可以浓缩成 40 位十六进制数据，所以字符长度是 40 位。然后底层存储空间是 160 bits / 8 = 20 bytes。
2. Solidity 的数据结构有一种定长字节数组 bytes1 到 bytes32，在存储英文、地址、Hash 的时候，内容字符串长度就等同于 bytes 长度。如果内容包括复杂字符，则无法对应。bytes 越小，实际占用的存储空间就小，降低 Gas fee。

# 2025.03.21

We set 'dA' as the private key (random number) and 'Qa' as the public key (a point), so we have : Qa = dA \* G (where G is the point of reference in the curve parameters).

公钥是 G 基点在某个椭圆曲线参数下进行点相加 private key 次数之后得到的结果。private key 其实是一个比较大的数字，是一个次数。

在以太坊下，使用的是 secp256k1 曲线，y² = x³ + 7：

```
p = FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEFFFFFC2F // 非常大的素数，做模数使用
a = 0000000000000000000000000000000000000000000000000000000000000000
b = 0000000000000000000000000000000000000000000000000000000000000007
G = (0x79BE667EF9DCBBAC55A06295CE870B07029BFCDB2DCE28D959F2815B16F81798,0x483ADA7726A3C4655DA4FBFC0E1108A8FD17B448A68554199C47D08FFB10D4B8) // 基点坐标
n = FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141 // 是椭圆曲线的阶，表示曲线上所有点的总数，定义了私钥可以选择的范围 1 到 N-1，基于 G 开始，不断进行点加法运算，在进行 N 次之后会回到起点
```

为什么选择这个参数的原因就是因为经过了大量的密码学研究和验证确保了安全性 然后这些参数确保了不同实现之间的兼容性，效率也不错。

生成的签名是 64 个 bytes 然后有两部分 value 组成 每部分是 32 个 byte 然后第一部分是 R 第二部分是 S 这个 R，S 共同组成了 ECDSA 的签名。

1. 先生成随机数 k 20 bytes
2. 计算 P = k \* G，得到了 P 在 curve 上的点，然后得到 (x, y) 坐标
3. x 的值就是 R
4. 使用 SHA-256 计算输出 256 bits 的 hash，是 32 bytes，使用 z 表示
5. S = k^-1 (z + dA \* R) mod p 可以获得 S 的值

## 关于乘法逆元和欧几里得算法

在 ECDSA 签名中，我们需要计算 k^-1（k 的乘法逆元）。乘法逆元在模运算中是一个重要的概念。

### 什么是模 p 下的乘法逆元？

在普通数学中：

- 如果 a \* b = 1，我们说 b 是 a 的乘法逆元
- 例如：2 \* 0.5 = 1，所以 0.5 是 2 的乘法逆元

但在模运算中（比如模 p）：

- 我们只处理整数
- 结果必须小于 p
- 乘法逆元的定义变成：如果 a \* b ≡ 1 (mod p)，则 b 是 a 在模 p 下的乘法逆元

### 扩展欧几里得算法计算乘法逆元

让我们用一个具体的例子来说明扩展欧几里得算法如何计算乘法逆元。

假设我们要计算 5 在模 7 下的乘法逆元（即找到 x，使得 5 \* x ≡ 1 (mod 7)）

扩展欧几里得算法的步骤：

1. 首先，我们有两个方程：

   ```
   7 = 1 * 5 + 2    (7除以5，商1余2)
   5 = 2 * 2 + 1    (5除以2，商2余1)
   2 = 2 * 1 + 0    (2除以1，商2余0)
   ```

2. 然后，从最后一个非零余数开始，往回推导：

   ```
   1 = 5 - 2 * 2
   ```

3. 将 2 用第一个方程表示：

   ```
   2 = 7 - 1 * 5
   ```

4. 代入上式：

   ```
   1 = 5 - 2 * (7 - 1 * 5)
   1 = 5 - 2 * 7 + 2 * 5
   1 = 3 * 5 - 2 * 7
   ```

5. 对 7 取模：
   ```
   3 * 5 ≡ 1 (mod 7)
   ```

所以，5 在模 7 下的乘法逆元是 3。

验证：

- 5 \* 3 = 15
- 15 mod 7 = 1
- 因此 5 \* 3 ≡ 1 (mod 7)

在 secp256k1 曲线中：

- p 是一个 256 位的大素数
- 计算过程类似，但数字更大
- 算法的时间复杂度是 O(log p)，比暴力尝试 O(p)要高效得多

这就是为什么扩展欧几里得算法能够高效地计算乘法逆元。它利用了辗转相除的思想，通过递归的方式快速找到结果。

# 2025.03.22

TODO 学习 7720 https://hackmd.io/@colinlyguo/SyAZWMmr1x https://mp.weixin.qq.com/s/WjpPNKEVlxlCSz1WyHH4tw

TODO 看看 EVM 的代码实现 https://docs.google.com/presentation/d/1_6tKfzWexxCe9og0c-_Qv0BfgSfqdlusXAgkStq3oY8/edit#slide=id.g33f2d85414d_4_0

# 2025.03.24

## Step 12: Verifying the Signature

如果想要 verify signature，需要 public key 就可以了：

P = S^-1*z*G + S^-1 _ R _ Qa

TOOD 需要推导一下公式，暂时跳过。简单的说，通过 S 和 Public Key 可以计算出 P 点，判断 P 点是否等于 R（也就是在椭圆曲线上的 x 坐标），如果相等，则说明签名有效，因为：

1. 只有知道 private key 的人才能生成正确的 S
2. 任何对消息 hash z、R 和 S 篡改，都会导致等式不成立

使用 random number k 和 dA private key 来计算出 S，然后使用 R 和 Qa Public key 可以 verify S。由于 trap door function，不能通过 Qa 和 R 反推计算出 dA 和 k，这样保护了 dA 的安全性。

## https://zh.epf.wiki/#/wiki/Cryptography/ecdsa

椭圆曲线之所以有趣，是因为曲线上的点构成一个群，即两个点 "相加" 的结果仍然在曲线上。

在生成 S 签名的时候，会使用一个随机数，在这个文档上面被称为临时秘钥 eK，生成临时公钥 eP。

根据公式得到 r = x 坐标 和 s = 签名。

验证就是：

- 对消息 Hash
- 计算临时公钥 R 然后对比 r，不一致就是有问题的

下面是一个 Solidity 的工作代码，使用场景是由服务器后端生成签名，然后放在合约里面进行校验并且开始某些权限，例如 NFT 白名单，在后端完成链下信息的检查，然后一起提交 mint：

1. 后端签名的生成逻辑：

```
async genSignature(types: string[], values: any[]) {
  const hash = ethers.utils.keccak256(
    ethers.utils.defaultAbiCoder.encode(types, values),
  );
  const signerWallet = new ethers.Wallet(
    process.env.SIGNER_WALLET_PRIVATE_KEY,
  );
  const message = ethers.utils.arrayify(hash);
  return await signerWallet.signMessage(message);
}

const signature = await genSignature(
  ['address'],
  [address],
);
```

可以获得类似：
0x4bddeedd03e68dcd87150421a61d4801d8250aa12c0b873a5585873b2bd3e98c67814616c9cb0c9652cfa98f218b5078093d6376d9d8fc56d130dc60266718ad1c 这样的 signature，实际上是三部分的合并：

- r = 0x4bddeedd03e68dcd87150421a61d4801d8250aa12c0b873a5585873b2bd3e98c
- s = 0x67814616c9cb0c9652cfa98f218b5078093d6376d9d8fc56d130dc60266718ad
- v = 0x1c

这里需要一个 signer，就是一个 private key，对应也有一个 public key。通过 signer 的 private key + random k + ECDSA 算法，可以生成上面的这个签名。目前是在中心化服务器后端生成。

2. 在智能合约端进行验证：

```
// 合约验证过程
function mint(bytes32 hashedMsg, bytes memory signature) external {
    // 1. 重建以太坊签名消息
    bytes32 ethSignedHash = hashedMsg.toEthSignedMessageHash();

    // 2. 从签名恢复地址
    address recoveredSigner = ethSignedHash.recover(signature);

    // 3. 验证签名者是否为授权地址
    require(recoveredSigner == signer, "Invalid signature");

    // 4. 执行铸造
    _safeMint(msg.sender, 1);
}
```

- toEthSignedMessageHash 是 <https://eip.fun/eips/eip-191> 规范里面的，额外增加了一个 message 前缀 `\x19Ethereum Signed Message:` 防止被攻击
- recover 使用 ecrecover 预编译合约进行恢复，可以得到一个 public key，就是 signer 的 public key
- 判断签名是否由 signer 签署，来验证当前签名是否被篡改或者有权限，因此 signer 的 private key 不能泄露

所以这个流程，需要保护好后端的 private key，然后将 signer public key 放在合约里面就可以进行白名单验证了，在后端服务进行判断并且生成 signature。

## secp256k1 曲线公钥恢复详解

secp256k1 曲线的基本参数:

```
曲线方程: y² = x³ + 7
p = FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEFFFFFC2F  // 模数
a = 0000000000000000000000000000000000000000000000000000000000000000  // 系数a
b = 0000000000000000000000000000000000000000000000000000000000000007  // 系数b
G = (
  x: 79BE667EF9DCBBAC55A06295CE870B07029BFCDB2DCE28D959F2815B16F81798,
  y: 483ADA7726A3C4655DA4FBFC0E1108A8FD17B448A68554199C47D08FFB10D4B8
)  // 基点坐标
n = FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFEBAAEDCE6AF48A03BBFD25E8CD0364141  // 曲线阶
```

## 输入参数

恢复公钥需要以下输入:

1. 消息哈希(hash): 32 字节
2. 签名数据(signature): 65 字节
   - R: 前 32 字节,表示临时公钥的 x 坐标
   - S: 中间 32 字节,实际签名值
   - V: 最后 1 字节,恢复标识符(recovery ID)

## 恢复过程

### 1. 恢复临时公钥 R 点

```javascript
// 1.1 从签名中提取R值(x坐标)
const x = BigInt(signature.slice(0, 32));

// 1.2 计算y坐标的可能值
const y2 = (x ** 3n + 7n) % p; // secp256k1曲线方程
const y = sqrt(y2, p); // 模平方根运算

// 1.3 根据recovery ID选择正确的y值
const v = signature[64]; // 最后一个字节是recovery ID
const R = {
  x: x,
  y: v === 27 ? y : p - y, // v决定使用哪个y值
};
```

### 2. 恢复原始公钥

```javascript
// 2.1 准备所需值
const r = x; // R点的x坐标
const s = BigInt(signature.slice(32, 64)); // 签名值S
const z = BigInt(hash); // 消息哈希

// 2.2 计算r的模乘法逆元
const r_inv = modInv(r, n); // 在群的阶n下求逆

// 2.3 计算公钥
// Qa = r^(-1) * (s*R - z*G)
const Qa = pointAdd(
  pointMul(R, s), // s*R
  pointMul(G, -z) // -z*G
).multiply(r_inv); // 整体乘以r^(-1)
```

# 2025.03.25

TODO 学习 7720 和相关实现 https://hackmd.io/@colinlyguo/SyAZWMmr1x https://mp.weixin.qq.com/s/WjpPNKEVlxlCSz1WyHH4tw

## https://mp.weixin.qq.com/s/WjpPNKEVlxlCSz1WyHH4tw

从以太坊的路线图来看，目前 ERC-4337 阶段进化到 Voluntary EOA Conversion 阶段了。ERC-4337 的进展可能有点慢，从数据角度来看几乎没有实质性的发展，没有被广泛使用，所以 7702 大幅提前。

EIP4337 的核心效果，就是在交易字段里增加了 Sender Address 字段，从而能让私钥与被操作的地址分离开。

EOA 的设计固有缺陷：

- 私钥一旦泄露就废了
- 只能使用 ECDSA 签名和校验算法
- 签名权限高
- 只能使用 ETH 支付手续费，不支持批量交易

破局之道在于实现账户抽象，将所有权（Owner）和签名权（Signer）解耦（Decoupling），从而才能逐个解决上述问题。

问题的解法看似有很多的 EIP 提案，但归根究底，就是两种核心思路，所以过往每一个没有被通过的 EIP，其考虑的问题也就汇聚成了现在方案的破局之道。

第一种路线是让 EOA 地址变为 CA 地址，让每个账户地址都拥有自己的“代码”逻辑。

第二种路线是让 EOA 地址驱动 CA 地址。EIP-3074 在 EVM 中加入两个新的 OpCodes AUTH 和 AUTHCALL ，让 EOA 能透过这两个 opcode 授权合约代替 EOA 的身份去呼叫其他合约。

EIP-7702 是什么
它通过新的交易类型来区分，可以允许 EOA 在单笔交易中临时具备智能合约的功能，进而支持业务上进行批量交易、无 Gas 交易和自定义权限管理等，且无需引入新的 EVM opCode（影响往前兼容性）。

定义了新的交易类型 0x04，其中新增了 authorization_list 对象，存储签名者希望在其 EOA 中执行的代码，用户签署交易的同时也签署要执行的合约代码，他作为二维列表存在，说明可以批量存放多个操作信息，执行批量操作。

要执行的合约代码以及操作指令在哪里？
“新”版本仅更改了代码部署方面的行为。
它不再将帐户代码设置为 contract_code，而是从 authorization_list 中检索代码 address 并将该代码设置为帐户代码。
所以，当需要执行授权代码时，从 authorization_list 的 address 字段指定的地址加载代码，并在签名者账户的上下文中执行。
这意味着用户的合约代码实际上是存储在链上的某个特定地址，而不是直接包含在交易中。
而操作指令和相关参数则存储在交易负载的 data 字段中。

打破了 tx.origin 和 msg.sender 两个比对的防护逻辑，很多过往的合约有风险了。

```
// 传统的安全检查模式
contract TraditionalSecurity {
    function someFunction() external {
        // 方式1: 检查调用者是否为EOA
        require(msg.sender == tx.origin, "Only EOA can call");

        // 方式2: 检查调用者是否为特定EOA
        require(msg.sender == owner, "Only owner can call");

        // 方式3: 检查调用者是否为合约
        require(msg.sender.code.length > 0, "Only contract can call");
    }
}
```

原本 msg.sender 和 tx.origin 在 EOA 下，是一样的，所以拿来判断必须通过 EOA 对合约方法进行调用。

7702 之后，使用 msg.sender == tx.origin 仍然可以判断当前是一个 EOA，但是当前 EOA 设置了代码，是一个合约账户。如果通过 7702 授权其他合约调用，那么 msg.sender = 被授权的合约，就会导致这个判断失效。

EIP-7702 的缺点是属于软分叉升级，需要大家共识推动，并且改动巨大，对链上生态影响太广：

1. 自由度极高，难以被审计，用户会更需求靠谱的钱包承担安全防护的保护。
2. 对原架构变化过大，虽然用不同交易类型区分，但是很多基建尤其链上不可改合约都无法直接适配。
3. 对 EOA 地址提供了合约能力，但对应的存储空间无法留存。看来就是每次发送 tx 的时候 setCode 进行调用。
4. 单独交易的成本稍微提高，因为会大量增加 Calldata 的部分，估算调用的总成本将是 16（gas） _ 15（字节） = 240（gas）calldata 成本，加上 EIP-3860 的成本 2 _ 15 = 30，再加上大约 的运行时成本 150。因此，仅仅准备账户哪怕什么都不做，就要增加 500 的 Gas 了。
5. “如果接收者签署了没有接收功能的代码，发送者在尝试发送资产时可能会面临 DoS。” 见案例。这个问题其实是 EOA A 签署了它不应该签署的东西——一个设置了错误实现（没有 receive()）的可重放文件。
6. 链上 冲提逻辑可能不一致，比如当转移 ERC-20 代币时，如果接收方账户有代码，则代币合约将调用 onERC20Received 接收方账户。如果 onERC20Received 还原或返回错误的值，则令牌传输将还原。
7. 另外如果 EOA 可以发出事件，会不会有什么问题？一些基础设施可能需要注意。

用户可以轻松用 EOA 做到多交易并行，比如授权代扣和执行代扣两种合一，这样对用户交易成本本身就低了，而对于 Dapp 而言，尤其是需要做链上企业级管理的项目方，比如交易所等更是颠覆性的优化，批量归集一旦原生态实现，基本交易所成本可以瞬间减少一半以上，最终也可以惠及用户。

TODO 使用 7702 做相关并行交易的酷炫 demo，让大家可以直观的体验到，找一个测试网。

# 2025.03.26

## https://eips.ethereum.org/EIPS/eip-7702

Add a new transaction type that adds a list of [chain_id, address, nonce, y_parity, r, s] authorization tuples. For each tuple, write a delegation designator (0xef0100 || address) to the signing account’s code. All code executing operations must load the code pointed to by the designator.

本质上就是新增了一种新的交易类型，增加了 authorization tuples，然后每个 tuple 指向了一个 delegation designator，执行的时候需要加载这些代码。参数：

- chain_id：链 ID，可能一个是避免分叉重放攻击
- address：授权的合约地址
- nonce：防止重放
- y_partiy, r, s：ECDSA 里面 v, r, s 的值，用于校验签名

```
// 委托标识符的格式
bytes32 designator = 0xef0100 || address

// 例如，如果授权地址是 0x1234...
// 委托标识符就是 0xef01001234...
```

大概工作流程就是：

1. 创建一个授权地址组，然后加入标记符 0xef0100 组装成 authList
2. 发送这个特殊的新交易
3. 执行这个交易代码，会循环将 authList 的 designator 指向的合约加载：codeAddress.call(msg.data)

主要的三类使用场景：

- Batching：一个 atomic 交易中，包含多个 operations。比如 ERC-20 先需要 approval 然后才能继续，需要两个 tx。基于 7702 可以实现更高级的用例：第一个操作的输出、输入第二个操作。
- Sponsorship：账号 X 为账号 Y 支付 gas fee，还可以支付其他的 ERC-20 token。
  - TODO 做个 demo 搞清楚原理 https://viem.sh/experimental/eip7702/contract-writes#6-optional-use-a-sponsor
  - ERC-20 代付的流程其实还是需要通过预言机进行换算，然后进行 token 的转移来实现
- Privilege de-escalation：签署 sub-keys 实现更加定制化的权限，例如 ERC-20 仅能与特定应用交互。
  - TODO 做个 demo 搞清楚原理

添加 authList 的实现逻辑：

1. 验证 chain id 是不是 0 或者当前的 ID
2. 做一些 nonce 之类的参数验证，包括 authority 数组里面的
3. 如果 address = 0x0000... 空地址，这个是 reset 的意思
4. 增加相关字段的 nonce

# 2025.03.27

## https://eips.ethereum.org/EIPS/eip-7702

### Delegation Designation

The delegation designation uses the banned opcode 0xef from EIP-3541 to designate the code has a special purpose.

# 2025.03.28

## https://eip.fun/eips/eip-7702

The following reading instructions are impacted: EXTCODESIZE, EXTCODECOPY, EXTCODEHASH, and the following executing instructions are impacted: CALL, CALLCODE, STATICCALL, DELEGATECALL, as well as transactions with destination targeting the code with delegation designation.

例如 EXTCODESIZE 会返回 23（0xef0100 的长度）而不是 0，所以用这种方式判断是否是 eoa 是不太好的。

看到 application 这一部分 https://eip.fun/eips/eip-7702#self-sponsoring:-allowing-tx.origin-to-set-code

# 2025.03.30

## https://eip.fun/eips/eip-7702

### Self-sponsoring: allowing tx.origin to set code

允许 tx.origin 来 set code 可以实现一笔交易完成 ERC20 的 approve-then-transfer 模式，让黑客钓鱼更开心。

TODO 做一个 demo。

之前有通过判断 msg.sender == tx.origin 来判断当前调用者是否为 EOA 钱包的逻辑，有了 7702 之后，这个判断可能会不成立。分析一下这种判断大概有三种 cases：

1. 确保 msg.sender 是一个 EOA。当 EOA 使用 EIP-7702 发送交易时，msg.sender 仍然是 EOA 的地址值。

# 2025.03.31

## https://eip.fun/eips/eip-7702

### Self-sponsoring: allowing tx.origin to set code

2. EOA 检测还会用于避免 atomic sandwich 攻击，例如 flash loans。通过判断是否是 EOA 会避免将多笔交易放在一个 atomic transaction 里面执行。这种情况下会受到影响。

TODO 做一个闪电贷的 demo。

3. 预防 reentrancy 重入攻击。因为 EOA 没有 receive call back 函数，所以没法被循环调用。7702 之后，EOA 也可以有 code，就是有 receive callback 函数，然后实现重入攻击。

TODO 做一个 EOA 重入攻击的 demo。

但是 EIP 坐着没有搜索到这样的重入攻击的防护逻辑，所以感觉问题还好？这个可以这样做吗？

EIP-7702 的执行流程：

1. Alice 是一个 EOA，发起一笔 tx，使用了 EIP-7702 的结构和类型。因此需要额外包括：

- 要执行的合约代码
- 相关签名
- 要调用的合约地址

2. tx 被客户端执行，Alice 的 EOA 变成一个临时合约账户，代码被部署，然后相当于合约：

- extcodesize(Alice) != 0 所以这种合约的判断就失效了

3. Alice 的这边 tx 开始被执行，然后开始通过合约调用合约
4. 交易执行完成，Alice 的账户的合约代码会被清除消失

---

为什么说 “tx.origin == msg.sender” 这种检查一直不安全？因为：

它判断的是调用链最开始是不是同一个人，不代表调用者的意图

攻击者可以诱导用户发交易，伪造出这种看似合法的调用结构

本质上是一种 权限外包 的思维漏洞，用户被迫替攻击者执行危险操作

而真正可靠的权限机制是：

- 检查 msg.sender 是不是你信任的地址
- 使用 ECDSA 签名验证（比如 OpenZeppelin 的 isValidSignature）
- 或者用 ERC-1271 判断智能合约钱包的合法签名

使用 tx.origin == msg.sender 做判断的合约一开始就有安全风险。

topmost context 是指 EVM 开始执行的最高层 context，每一层合约调用，都是增加了一层 execution layer。

---

7702 是被设计为高度兼容最终的 Account Abstraction，算是一个过渡方案吧，但是不关联过多的 ERC-4337 和 RIP-7560 的细节。

- 可以将 code 指向 ERC-4337 wallet code 来调用。TODO 做一个 demo
- 可以兼容和组合目前的种种合约，不会创建两套系统，同时可以快速的、方便的接入使用 ERC-4337

---

需要考虑的一些安全注意事项，主要在签名验证之类的地方，后面再详细看看吧。

# 2025.04.01

## event logs

在做 FS 的贡献内容存储方案设计，目前看下来使用 Event Logs 是比较方便的，因为需求是：

1. 永久存储
2. 直接通过合约读写，不需要额外的步骤（例如之前的 EAS）
3. 较低的成本和 gas fee

目前看下来 L2 的 Event logs gas fee 对于一个推特长度 140 bytes 几乎也没多少钱，理论计算是可行的。

但是目前 L2 的 Event logs 是不存在 L1 的，仅仅在 L2 上面，可靠性可能存疑。此外 L1 的 Event logs 也仅仅存储在 full node 上面，因为跟合约不相关，内容量又比较大，所以在很多客户端或者轻量的可能不会存储。

此外，由于一个 block gas 的限制，似乎是建议将内容切割成 1-2 kb 长度的 chunk 进行循环上传。

https://docs.alchemy.com/docs/deep-dive-into-eth_getlogs

# 2025.04.03

## https://hackmd.io/@colinlyguo/SyAZWMmr1x

EIP-7702 introduces a new transaction type that allows an Externally Owned Account (EOA) to specify an address as the pointer to its implementation. For example, this address could be a generic proxy or minimal proxy contract that forwards call messages to an upgradable wallet implementation.

这个描述还是比较形象的。一种新的 tx type，然后支持 EAO 指向一个 address 表示的是它的实现。这样就有点像 proxy 了，将 call messages 传递到那个 address。这样的话，有点像一个可升级的智能合约钱包。

提供了一些 features：
TODO 休闲黑客松 EIP 7702 主题，拉上 EF，等 Pectra 上线之后，一周学习，周末打。

- Social recovery：防止 private keys 丢失 ？？ 如何实现？？
- Transaction batching：合并交易，签名一次即可。FS 2.0 可以使用这种方式投票 + dispatch，还可以 sponsor gas
- Transaction Sponsorship：找人代付
- Arbitrary signing keys：支持 WebAuth、P256、BLS 等签名类型？？
- Session Keys：Users 可以生成一个 delegate session keys 带有 lifecycle 和 scoped permissions 给到服务商。
- 自定义 gas token、多签验证等等功能

这简直是提供了特别大的能力。跟 ERC-4337 结合，可以很好的集成在一起。Once a user authorizes an address with a smart contract wallet supporting ERC-4337, the EOA address can serve as the sender field of UserOperation in ERC-4337. 不用单独部署一个 AA 钱包，就可以使用相关的功能。

Smart EOAs 不能解决 private key 被别人知道的问题，private key 还是最高权限的。例如 EIP-7377 可以将 EOAs 迁移到 AA 钱包，还有其他的 EIP 可以在未来解决。

有意思，钱包可以要求用户在 authorizing EOA as a contract wallet 之后，删除 private key 以防止私钥泄漏。可以避免 local cache 或者供应链攻击。TODO 按照我的理解，授权是在 tx 的过程中进行的？还是需要 EOA sign 对应的 tx 才可以实现 authorizing？如何签名呢？

# 2025.04.05

## https://hackmd.io/@colinlyguo/SyAZWMmr1x

Smart EOA vs SC 优势：

1. 开启 SC 高级功能的同时，保持地址不变，有助于持有 SBT。
2. 简化 stealth addresses 的实现？？TODO
3. 兼容目前的系统

### Protocol Overview

新增一个 tx type，SET_CODE_TX_TYPE 0x04，然后 payload 增加了一些字段，最重要的是 authorization_list 字段：

\text{authorization_list} = [ \, [ \, \text{chain_id}, \, \text{address}, \, \text{nonce}, \, \text{y_parity}, \, \text{r}, \, \text{s} \, ], \, \dots \, ]

address 是目标代理 code 的地址。

while the EOA's address can be recovered from the payload (chain_id, address,nonce) and the signature (y_parity, r, s). 

oh shit！只要有 signature 就可以代替 private key 来执行相关命令，所以这里需要限定授权的 chain id 和 address，只能让这个地址进行工作，这样就实现了无需反复签名或者必须依赖 private key 了呀！TODO FairSharing 投票授权加一下这个功能。

TODO 需要 7702 的安全风险 reviewers，类似 revoke cash，展示当前 eoa 可能存在的 7702 风险，并且提供风险监测服务。可以提供一个通用 API，返回当前操作的详细解读，进行确认。

### 一些细节的 topics 分析

authorization_list 必须是非空的。客户端需要验证里面的每个参数，确保准确无误。

In each tuple of the authorization_list, the address field represents the authorized address, while the signer’s address is derived from signature and payload. 这个 Signer 地址还是会变的吗？

这个设计可以实现 sponsored transactions？如何实现 todo？？然后即便是一个 fails，其他的 authorization 不受到影响。

EIP-7702 introduces a new mechanism that increments the nonce of an EOA after each successful authorization. 

This means that if a transaction includes an authorization list and the authorization signer is the same as the transaction signer, you must set tx.nonce to account.nonce and authorization.nonce to account.nonce + 1.

With EIP-7702, from the perspective of an EOA, ETH balances can now decrease not only through signed transactions but also through transactions triggering contract executions. From the view of smart contracts, EOAs can directly send transactions to change account states. TODO 做个 Demo 吧，直接修改数值那 eth 去哪里了？

之前比较简单，因为 EOAs 只能通过 signed txs 才能发送 ETH，所以 client 可以验证 tx pools 里面 pending txs 就可以判断出是否有足够余额。但是现在需要更多判断，因为 7702 delegated code 可以任何时候修改 EOA 余额。

# 2025.04.06

## https://hackmd.io/@colinlyguo/SyAZWMmr1x

Revocation 取消，可以通过设置 address field 为 0x0000000000000000000000000000000000000000 取消所有的授权。这样会把 account's code 清空，code hash 设置为 empty hash。

这个有点违背我先前的理解，这个写入到底是永久的，还是发送交易的时候临时写入的？如果是临时写入的，如何实现 social recovery 呢？如果没有 private key，如何发送这个特殊的交易呢？

Re-delegating an account requires careful storage management to avoid collisions, which can otherwise lead to undefined behavior. 

TODO 做一个 EIP Fun extension，可以识别页面的 EIP、ERC 然后到 eip.fun 官网。然后可以在后面加个括号，使用一句话来解释这个 RIP、EIP、ERC 的效果，方便直接理解上下文。

### Core Protocol Implementations

TODO 全部的 geth 7702 实现在这里，学习下看看未来如何增加 https://github.com/ethereum/go-ethereum/pull/30078

Sending an EIP-7702 Transaction

To send an EIP-7702 transaction, the user first needs to fill in all non-signature fields outside of the authorization_list (definition of SetCodeTx here). Then, the user constructs the authorization_list here. For each authorization tuple, the user fills in the non-signature fields first, and then signs it using the SignAuth function.

SignAuth 是什么？

The user's EOA private key is required for signing. Signing involves hashing the authorization details, which include a magic number 0x05, a chain ID, the delegated code address, and the current nonce of the EOA.

还是需要 EOA 私钥签名，内容包括 auth details 的 hash。

Notably, setting the chain ID to 0 allows the authorization to be replayed across all EVM-compatible chains supporting EIP-7702, provided the nonce matches.

可以实现多个 evm 链重放，但是需要 nonce match，看来 nonce 这里的逻辑还是需要搞清楚一些。实际上，每个 evm 的 nonce 可能不一样，所以可能也很难在不同的链重放。

# 2025.04.07

## 技术实现

Once all fields except the signature fields of transaction are prepared, the user initializes the transaction (instructions here) and a "prague signer" (which adds support for EIP-7702 transaction hashing). The user then signs the transaction using SignTx and serializes it into a byte array using RLP encoding via MarshalBinary. Finally, the RLP byte array of transaction is hex-encoded and sent via the eth_sendRawTransaction RPC method.

TODO 需要看看原文和对应的代码，目前在飞机上，看不到。

简单的说：

1. 组装用户的请求，包括 delegated code address 等
2. 签名和序列化压缩
3. 发送 RPC
4. 节点相互同步
5. block proposer 检查：1. 不是一个合约创建 tx 2. 包括至少一个 authorization operation
6. 计算 auth 部分的 gas 消耗
7. 开始挨个执行 auth tuple，做出相应的操作
  1. 判断 ChainID = 0？
  2. Nonce 没有超过最大
  3. 验证 signature values，还原 signer address（其实是 public key）
  4. 没有 code 或者 exisiting delegation，排除 SC

TODO 授权 MCP service provider？实现无私钥自动授权执行命令。

## Nonce Increment Logic

因为不像以前那样，一次执行一个 tx，所以 nonce 的逻辑改变了。会有两个 check：

1. Transaction Nonce Validation and Increment: The transaction nonce is first checked to ensure it matches the account's current nonce in the state (validation here). Once validated, the transaction nonce is incremented (increment here).
2. Authorization Nonce Validation and Increment: For each authorization tuple in the authorization_list, the auth.Nonce is validated to ensure it matches the current nonce of the signing account (validation here). After successful validation, the nonce associated with the authorization is incremented (increment here).

新增了一个 authorization nonce，跟 tx nonce 分开处理了看来是。TODO 代码验证一下。

Users can use eth_getTransactionByHash and eth_getTransactionReceipt to query a transaction's status. 

有一些 Instruction Modifications：

- EXTCODECOPY：return 0xef01
- EXTCODESIZE：2 the byte length of 0xef01
- EXTCODEHASH：Keccak256 hash of 0xef01
- CALL, CALLCODE, DELEGATECALL, and STATICCALL will load the code from the delegated code address and execute it in the context of the smart EOA. 有点像 proxy。不合法的 delegations 会被跳过。
- CALL、CALLCODE、DELEGATEDCALL、STATICCALL 的 gas 有所调整

# 2025.04.08

## 调用 Smart EOAs

类似 call 一个 smart contract。就是在 tx 的 to 字段（7702 tx 中的 destination）设置为 Smart EOA address，就可以去调用这个 EOA 上面的参数了。

原来是通过这种方式实现无私钥的调用，但是还是需要构造一个 tx，可能需要一个随机或者 sponsor 的 EOA？TODO 做个 demo 试试看。

Nodes 仅仅处理 one level of delegation，避免复杂化和递归死循环。

## 功能实现方案

### 创建钱包

TODO 实际跑一下 demo。

一个 tx 合并多个 txs 的功能，这也就意味着 authorization 的 tuple 具备链式调用的可能？但是这样的话，有一些由于问题被跳过，难道不会有问题？反正签名一次，就可以完成多笔操作，避免重复签名。

由于 tx sender 没有设置，所以可以先签名生成 payload，之后被 third-party EOA 来提交上去。这样就实现了 sponsor tx service。比如使用代付服务的时候，可以通过 gitcoin passport scorer 的检测来避免女巫攻击。

TODO 跟 gas station 的区别？感觉这里可以创建一个 gas 代付以及跨链代付的基础服务，抹平 L2 之间 gas fee 和操作的问题。实现 Arb 消费 OP 的 gas，进行操作。

Preventing signing key tampering 这个没看懂 TODO

### Executing Calls

The user uses the WebAuthn key to sign the contract call, and extract the needed fields in the signature based on the specific wallet implementation.

### Recovery from Private Key Loss

When the private key is lost, the user can simply use their registered signing key to transfer assets from the EOA account.

TODO 授权转移资产合约是不是很危险？别人难道不会去调用？或许可以提供这种找回服务，授权转移到指定的钱包地址，这样 eoa pk 丢失了可以找回资产。需要增加一些 social recovery 或者 multiple guardinas 的机制来 move the balance。

### ZK Recovery

EIP-7702 enables wallet recovery by Zero-Knowledge Proof (ZKP), through delegating to a contract that supports these functionalities. One advantage of ZKP is verifying user information without compromising privacy. For instance, the authentication can leverage OpenID providers like Google or Facebook to issue JSON Web Tokens (JWTs) as a witness for ZKP, utilizing established identity verification infrastructure.

TODO ZKP recovery demo 跑一下。

Here's an example of a ZK recovery based on EIP-7702 provided by zklogin. To change the wallet's signing key, the user must use a new WebAuthn key to go through the login flow of an OpenID provider (in the example, Google) to obtain a JWT. This JWT is then used to generate the proof. Finally, the proof along with additional metadata from the JWT and the new WebAuthn public key are submitted to the contract to update the signing key.

### Session Keys and Subscriptions

in traditional systems like credit cards, "trusted" subscription-based payment services can reduce the cognitive cost of authorization.

可以创建独立的 key 用来签署一些低风险的签名，创建一个 signing key，然后保存在 local，之后可以被读取，由前端自动签署操作，不需要 wallet prompts。


# 2025.04.09

## EIP-7702 in Roadmaps of Ethereum and L2s

EIP-7702 can help EOA users experience the functionalities of SC wallets, such as social recovery, subscriptions, etc. However, because it retains the existence of private keys, it cannot completely replace ERC-4337. For users who have applied EIP-7702, how can they fully obtain the security guarantees of SC wallets?

With slight modifications, it could support the migration of EIP-7702 accounts to SC accounts. e.g. adding a deactivated field in the account state for deactivated EOA private keys.

通过一些修改和特殊的字段，直接让 EOA 只能按照 SC 的方式被调用，实现 EOA 升级到 AA 钱包。

a user can upgrade its delegated code to fully migrate to an EVM Object Format (EOF) contract, taking advantage of the new upgrade schemes supported by EOF and removing the need for a legacy proxy contract.

TODO EOF 是什么意思？主要做什么用？

On L2, RIP-7560 has been designed to pioneer the experimentation and implementation of native AA on L2, along with related proposals: RIP-7711, RIP-7712, and RIP-7696. RIP-7560 integrates the tooling provided by ERC-4337 with the design of native AA in EIP-2938, introducing a new AA_TX_TYPE transaction.

实际上，整个以太坊的运行，就是 tx + sign，签名证明这是你发出的 tx，然后被验证之后，保存到 state 里面。Native AA？也需要有相关 EOA 来触发和调用吧？TODO 这里需要验证一下。

using the contract address as tx.origin 是 Native AA 的一些优势。

On Ethereum, EIP-7701 builds upon the design of RIP-7560 to propose a native AA mechanism.

By leveraging EOF, users can upgrade their smart EOAs using EIP-7702 to ensure compatibility with EIP-7701, either by migrating to new EOF-based contracts or by updating their proxy contract's implementation address.


<!-- Content_END -->
