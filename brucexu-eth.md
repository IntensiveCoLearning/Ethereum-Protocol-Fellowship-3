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

TODO 看看 EVM 的代码实现 https://docs.google.com/presentation/d/1_6tKfzWexxCe9og0c-_Qv0BfgSfqdlusXAgkStq3oY8/edit#slide=id.g33f2d85414d_4_0

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

# 2025.3.21

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

<!-- Content_END -->
