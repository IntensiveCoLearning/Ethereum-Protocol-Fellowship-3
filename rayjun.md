---
timezone: UTC+8
---

> 请在上边的 timezone 添加你的当地时区(UTC)，这会有助于你的打卡状态的自动化更新，如果没有添加，默认为北京时间 UTC+8 时区


# 你的名字

1. 自我介绍
   Ray，一个准备深入学习以太坊的人
3. 你认为你会完成本次残酷学习吗？
   会的
5. 你的联系方式（推荐 Telegram）
   TG：@rayoo_eth

## Notes

<!-- Content_START -->

### 2025.03.10
这次共学用来研究 go-ethereum 代码，代码版本：https://github.com/ethereum/go-ethereum/commit/4cdd7c86314e5f5a98fa68e928deaa99c026f40d。

go-ethereum 代码结构及各个模块功能如下：
- accounts：管理以太坊账户，包括公私钥对的生成、签名验证、地址派生等
- beacon：处理与以太坊信标链（Beacon Chain）的交互逻辑，支持权益证明（PoS）共识的合并（The Merge）后功能
- build：构建脚本和编译配置（如 Dockerfile、跨平台编译支持）
- cmd：命令行工具入口，包含多个子命令
- common：通用工具类，如字节处理、地址格式转换、数学函数
- consensus：定义共识 API，包括之前的工作量证明（Ethash）和单机权益证明（Clique）以及 Beacon engine 等
- console：提供交互式 JavaScript 控制台，允许用户通过命令行直接与以太坊节点交互（如调用 Web3 API、管理账户、查询区块链数据）
- core：区块链核心逻辑，处理区块/交易的生命周期管理、状态机、Gas计算等
- crypto：加密算法实现，包括椭圆曲线（secp256k1）、哈希（Keccak-256）、签名验证
- docs：文档（如设计规范、API 说明）
- eth：以太坊协议的完整实现，包括节点服务、区块同步（如快速同步、归档模式）、交易广播等
- ethclient：实现以太坊客户端库，封装 JSON-RPC 接口，供 Go 开发者与以太坊节点交互（如查询区块、发送交易、部署合约）
- ethdb：数据库抽象层，支持 LevelDB、内存数据库等，存储区块链数据（区块、状态、交易）
- ethstats：收集并上报节点运行状态到统计服务，用于监控网络健康状态
- event：实现事件订阅与发布机制，支持节点内部模块间的异步通信（如新区块到达、交易池更新）
- graphql：提供 GraphQL 接口，支持复杂查询（替代部分 JSON-RPC 功能）
- internal：内部工具或限制外部访问的代码
- log：日志系统，支持分级日志输出、上下文日志记录
- mertrics：性能指标收集（Prometheus 支持）
- miner：挖矿相关逻辑，生成新区块并打包交易（PoW 场景下）
- node：节点服务管理，整合 P2P、RPC、数据库等模块的启动与配置
- p2p：点对点网络协议实现，支持节点发现（如Kademlia）、数据传输、加密通信
- params：定义以太坊网络参数（主网、测试网、创世区块配置）
- rlp：实现以太坊专用的数据序列化协议 RLP（Recursive Length Prefix），用于编码/解码区块、交易等数据结构
- rpc：实现 JSON-RPC 和 IPC 接口，供外部程序与节点交互
- signer：交易签名管理（硬件钱包集成）
- tests：集成测试和状态测试，验证协议兼容性
- trie & triedb：默克尔帕特里夏树（Merkle Patricia Trie）的实现，用于高效存储和验证账户状态、合约存储


### 2025.03.11
geth 节点是如何启动的？

TL;DR：geth 的启动的入口是在 cmd/geth 下，在启动时，会初始化交易池、后端 RPC 服务、p2p 模块、状态数据库、链区块数据库等模块，等待这些模块都初始化之后，节点启动完成。 


geth 程序入口在 cmd/geth/main.go 中实现， 使用了 https://github.com/urfave/cli 库来定义程序的入口，这个定义在 main.go 中的 init 方法中定义了 Action 是 geth 函数：
![image](https://github.com/user-attachments/assets/965b788b-289c-4bb1-8be0-5d3d2eba25e1)

在 geth 函数中会做一系列的准备，并启动 node，其中准备工作是在 makeFullNode 函数中来实现的：
![image](https://github.com/user-attachments/assets/11e304cb-0287-4d6a-acd6-1f8e5bcce845)

在这个函数中去注册和实现以太坊真正的需要实现的一些服务，比如下面的 utils.RegisterEthService 就是启动以太坊节点抽象的实现：
![image](https://github.com/user-attachments/assets/c602d45a-d945-4e5d-807e-a2b120025cd9)

通过 New 函数来实现以太坊节点：
![image](https://github.com/user-attachments/assets/b44ac0ab-8be4-40d5-9c4c-7a35072bb088)

geth 整个节点被抽象成一个 Ethereum 的结构体，在 eth/backend.go 中，eth.New() 就是实例化这个 Ethereum 节点的函数：
```Go
type Ethereum struct {
	// core protocol objects
	config         *ethconfig.Config
	txPool         *txpool.TxPool
	localTxTracker *locals.TxTracker
	blockchain     *core.BlockChain

	handler *handler
	discmix *enode.FairMix

	// DB interfaces
	chainDb ethdb.Database // Block chain database
   //....
}
```

`eth.New` 主要用来实现 Ethereum 的初始化，其中会有一些参数和当前节点状态的检查，也会执行一些初始化的操作.

初始化链数据库：
![image](https://github.com/user-attachments/assets/0d66a953-a2c1-4c95-822b-af9012228183)

初始化 BlockChain：
![image](https://github.com/user-attachments/assets/357e6e53-ec24-4e05-9d81-c3d399f9c360)

初始化交易池：分为 blob 交易的交易池和其他交易的交易池：
![image](https://github.com/user-attachments/assets/0fc0dbb6-944d-4d06-9675-2c12152eed1d)

初始化 handler，handler 代表 Ethereum 协议的入口，用来同步节点数据、接受 p2p 交易消息广播、并将新的交易添加到交易池中：
![image](https://github.com/user-attachments/assets/6af7bc78-bf24-42db-bae8-6a8d72782613)

实例化 RPC 的服务端，并注册 gasPrice 的预言机：
![image](https://github.com/user-attachments/assets/493b1bd3-5af0-4283-aa3a-35a7eba42d18)

RPC 会在 node 中启动，RPC 接口通过注册的方式注册到 node.go 中：

初始化配置
```Go
func New(conf *Config) (*Node, error) {
    //.....
    // Register built-in APIs.
    node.rpcAPIs = append(node.rpcAPIs, node.apis()...)
    // Configure RPC servers.
	node.http = newHTTPServer(node.log, conf.HTTPTimeouts)
	node.httpAuth = newHTTPServer(node.log, conf.HTTPTimeouts)
	node.ws = newHTTPServer(node.log, rpc.DefaultHTTPTimeouts)
	node.wsAuth = newHTTPServer(node.log, rpc.DefaultHTTPTimeouts)
	node.ipc = newIPCServer(node.log, conf.IPCEndpoint())
    //.....
}
```
启动节点
```Go
func (n *Node) Start() error {
    // .....
	// open networking and RPC endpoints
	// 这里会去启动 rpc 服务
	err := n.openEndpoints()
    //.....
}
```

然后 client 就可以连接节点，开始访问 RPC，RPC 的实现细节在 `internal` 包中实现，比如 eth namespace 在 inernal/ethapi 下实现，所有的接口定义在 backend.go 中。

注册 api:

```Go
func GetAPIs(apiBackend Backend) []rpc.API {
	nonceLock := new(AddrLocker)
	return []rpc.API{
		{
			Namespace: "eth",
			Service:   NewEthereumAPI(apiBackend),
		}, {
			Namespace: "eth",
			Service:   NewBlockChainAPI(apiBackend),
		}, {
			Namespace: "eth",
			Service:   NewTransactionAPI(apiBackend, nonceLock),
		}, {
			Namespace: "txpool",
			Service:   NewTxPoolAPI(apiBackend),
		}, {
			Namespace: "debug",
			Service:   NewDebugAPI(apiBackend),
		}, {
			Namespace: "eth",
			Service:   NewEthereumAccountAPI(apiBackend.AccountManager()),
		}, {
			Namespace: "personal",
			Service:   NewPersonalAccountAPI(apiBackend, nonceLock),
		},
	}
}
```

### 2025.03.12
交易是如何被节点处理的（1）

向以太坊节点发送交易的方式由两种：

- `SendTransaction`
- `SendRawTransaction`

SendTransaction 需要节点本身有对应的私钥来对签名交易，SendRawTransaction 则是需要提前对交易签名，然后将签好名的交易提交到节点，这种方式更常用，使用 Metamask 等钱包使用的就是这种方式。

在这里以 SendRawTransaction 为例，在节点启动之后，节点会启动以一个 API 模块，用来处理外部的各类 API 请求，SendRawTransaction 就是其中的一个 API，源代码在 `internal/ethapi/api.go` :
```Go
func (api *TransactionAPI) SendRawTransaction(ctx context.Context, input hexutil.Bytes) (common.Hash, error) {
	tx := new(types.Transaction)
	if err := tx.UnmarshalBinary(input); err != nil {
		return common.Hash{}, err
	}
	return SubmitTransaction(ctx, api.b, tx)
}
```

当前以太坊总共支持五类交易，通过 `tx.UnmarshalBinary` 方法，将对应的交易类型解析出来，并且将交易数据反序列化。

五类交易：

- LegacyTxType
- AccessListTxType
- DynamicFeeTxType
- BlobTxType
- SetCodeTxType

在完成交易的反序列化之后，就会调用 `SubmitTransaction` 函数向交易池提交交易，SendTransaction 和 SendRawTransaction 都会使用这个函数来向交易池提交交易。

检查 gas fee 是否合理：
![image](https://github.com/user-attachments/assets/c31c4b78-18ec-4acf-8e2d-e8ece01ed6cb)

EIP-155 通过在交易签名中引入 chainID 参数，解决了跨链交易重放问题。该检查确保当节点配置开启 EIP155Required 时，所有通过 RPC 提交的交易都必须符合这个标准。
![image](https://github.com/user-attachments/assets/8293518e-24ce-4d35-9341-3c3c30d339dc)

提交交易到交易池：
![image](https://github.com/user-attachments/assets/14e12d3a-7dc5-4c2c-8617-5eaa7aca6b12)

![image](https://github.com/user-attachments/assets/80b12d41-34df-4263-bb50-519c2c638b65)

在交易池中，会根据对应的交易类型来匹配对应的交易池，如果是 blob 交易，那么就会放入 BlobPool ，否则放入LegacyPool 。
![image](https://github.com/user-attachments/assets/fe592681-3ccc-4500-a219-db0e569534dc)

到这里，用户提交的交易就处理完成了，后续的步骤会由节点来处理。

如果在交易打包之前，重新发送了一笔交易，新的交易设置了新的 gasPrice 和 gasLimit，就会把原来交易池中的交易删除，替换成了新的 gasPrice 和 gasLimit 之后重新返回到交易池中。
![image](https://github.com/user-attachments/assets/fd852ae3-ed64-4814-8d99-d8ebcd7aaebc)

### 2025.03.13
交易是如何被节点处理的（2）

交易在被发送到节点之后，需要在节点之间传播。

节点接收到交易之后，需要在网络中进行传播，txpool（core/txpool/txpool.go ）提供了 SubscribeTransactions 方法，可以订阅交易池中的新事件：

```Go
func (p *TxPool) SubscribeTransactions(ch chan<- core.NewTxsEvent, reorgs bool) event.Subscription {
	subs := make([]event.Subscription, len(p.subpools))
	for i, subpool := range p.subpools {
		subs[i] = subpool.SubscribeTransactions(ch, reorgs)
	}
	return p.subs.Track(event.JoinSubscriptions(subs...))
}
```

新交易会通过 p2p 模块广播出去，同时也会从 p2p 网络中接收新的交易。在 eth/backend.go 中初始化 Ethereum 实例时，会初始化 p2p 模块，添加交易池的接口。p2p 模块运行起来之后，会从 p2p 消息中解析出交易请求添加到交易池中：
```Go
// eth/backend.go New 函数
if eth.handler, err = newHandler(&handlerConfig{
		NodeID:         eth.p2pServer.Self().ID(),
		Database:       chainDb,
		Chain:          eth.blockchain,
		TxPool:         eth.txPool,
		Network:        networkID,
		Sync:           config.SyncMode,
		BloomCache:     uint64(cacheLimit),
		EventMux:       eth.eventMux,
		RequiredBlocks: config.RequiredBlocks,
	}); err != nil {
		return nil, err
	}
	
	// eth/handler.go newHandler 函数，注册获取信息新交易的过程
	fetchTx := func(peer string, hashes []common.Hash) error {
		p := h.peers.peer(peer)
		if p == nil {
			return errors.New("unknown peer")
		}
		return p.RequestTxs(hashes)
	}
	addTxs := func(txs []*types.Transaction) []error {
		return h.txpool.Add(txs, false)
	}
	h.txFetcher = fetcher.NewTxFetcher(h.txpool.Has, addTxs, fetchTx, h.removePeer)
	
	
	// eth/handler_eth.go Handle 方法，在接收到新的交易之后，会添加到交易池中
		for _, tx := range *packet {
			if tx.Type() == types.BlobTxType {
				return errors.New("disallowed broadcast blob transaction")
			}
		}
		return h.txFetcher.Enqueue(peer.ID(), *packet, false)
	
	// eth/fetcher/tx_fetcher.go 的 Handle 方法会调用上面注册的 addTxs 来将讲义添加到交易池
	for j, err := range f.addTxs(batch) {
		//....
	}
```

同时 p2p 模块 (eth/handler.go)  会持续监听新交易事件，如果接收到了新交易，那么就会发送广播，将交易广播出去：
```Go
// eth/handler.go 在产生新交易之后，会通过 p2p 网络广播出去
func (h *handler) txBroadcastLoop() {
	defer h.wg.Done()
	for {
		select {
		case event := <-h.txsCh:
			h.BroadcastTransactions(event.Txs)
		case <-h.txsSub.Err():
			return
		}
	}
}
```

在广播交易时，需要对交易进行分类，如果是 blob 交易或者是超过了一定大小的交易，无法直接传播，对于普通的交易，则标记为可以直接传播。然后从当前节点的对等节点中去找那些没有这笔交易的节点。如果节点可以直接广播，则标记为 true：

![image](https://github.com/user-attachments/assets/8274a365-4702-4c7b-94ab-060d6682d400)

依照上面的原则对交易分类完成，可以直接传播的交易就直接发送，blob 交易或者大交易则只传播 hash，等到需要用到这笔交易的时候再来获取。
![image](https://github.com/user-attachments/assets/48094943-793a-4592-a4e8-f7cec28b1987)

这些延迟加载的逻辑在 core/txpool/subpool.go 中实现，实际需要获取交易时，再从具体的交易池获取：

```Go
type LazyTransaction struct {
	Pool LazyResolver       // Transaction resolver to pull the real transaction up
	Hash common.Hash        // Transaction hash to pull up if needed
	Tx   *types.Transaction // Transaction if already resolved

	Time      time.Time    // Time when the transaction was first seen
	GasFeeCap *uint256.Int // Maximum fee per gas the transaction may consume
	GasTipCap *uint256.Int // Maximum miner tip per gas the transaction can pay

	Gas     uint64 // Amount of gas required by the transaction
	BlobGas uint64 // Amount of blob gas required by the transaction
}

func (ltx *LazyTransaction) Resolve() *types.Transaction {
	if ltx.Tx != nil {
		return ltx.Tx
	}
	return ltx.Pool.Get(ltx.Hash)
}
```

### 2025.03.14
交易是如何被节点处理的（3）

在交易提交到交易池之后，会在节点之间传播，但某个 Validator 被选中出块之后，就会委托 CL 和 EL 构造区块。

CL 在接收到区块构造请求之后，就会调用 EL 的 engineAPI 来构造区块。会先调用 `ForkchoiceUpdated`  API 来发送构造区块的请求，具体调用哪个版本依据当前网络版本来决定，调用完成之后会返回 PayloadID，然后根据这个参数调用 `GetPayload` 对应版本 API 来获取区块的构造结果。

无论调用的是 ForkchoiceUpdated 的哪个版本，最终都是调用 forkchoiceUpdated 方法来构造区块：
![image](https://github.com/user-attachments/assets/3531596f-3d1a-4201-9343-8e8a62db2dc9)

并且在这个方法中会对 EL 当前的状态做校验，如果当前 EL 正在同步区块、或者最终的区块不对，那么都会直接返回，构造区块失败：
![image](https://github.com/user-attachments/assets/5be19e07-094d-439c-9f9f-5664d624b82a)
![image](https://github.com/user-attachments/assets/4faa18c0-a9fc-4f8d-9c4e-fb65d7c34a15)

在校验完成之后，就会调用 miner 的 BuildPayload 来构造区块：
![image](https://github.com/user-attachments/assets/4a76ab9c-fbfd-465b-8805-c9cdfd05e9b2)

构造区块的具体操作都在 generateWork 中完成，但这里需要主要，调用这个方法之后，就产生了一个 payload，并把这个 payloadID 返回给 EL 层。同时会启动一个 goroutine，持续的去交易池中找价值更高的交易，每次重新打包交易之后，都会更新 payload。
![image](https://github.com/user-attachments/assets/548150e1-a51a-4ed0-8a18-a345242a6050)

打包交易的是通过 fillTransactions 方法来完成：
![image](https://github.com/user-attachments/assets/bfd81d4d-440b-414b-af50-05fda20dc778)

实际上就是调用 txpool 的 Pending 方法来获取待打包的交易：
![image](https://github.com/user-attachments/assets/b5ae4a52-29c5-4529-9ed2-a9ee073deea8)

CL 层在 slot 结束之前会调用 getPayload API 来获取最终打包好的区块。如果提交的交易被打包在这个区块当中，那么交易就会被 EVM 执行，并改变世界状态。如果这次没有被打包，那么就会等待下一次被打包。

### 2025.03.15
交易是如何被节点处理的（4）

在打包交易的过程中，同时也会将交易在 EVM 中执行，得到区块交易完成之后状态的变化，同样还是在 generateWork 函数中：
![image](https://github.com/user-attachments/assets/52e698cd-502f-4559-bc66-a8af220a95e6)

准备好当前区块执行的环境变量，主要是获取最新区块和最新状态数据库：
![image](https://github.com/user-attachments/assets/74a14aee-4d5c-493c-a387-e881b980b2ab)

这里的 state 就是代表状态数据库：
![image](https://github.com/user-attachments/assets/9144eb2d-491c-446e-a7cd-922c529dd1a7)

在这里形成了一个 StateDB → stateObjects→ stateAcount 的结构，分别代表完整的状态数据库、账号对象集合以及单个账户对象。其中 StateObject 中结构中，dirtyStorage 表示当前交易执行后的状态，pendingStorage 表示当前区块执行之后的状态，originStorage 表示原始的状态，所以这三个状态从新到是 dirtyStorage → pendingStorage → originStorage：
![image](https://github.com/user-attachments/assets/7d0df004-e17d-4a7e-996e-9a5b1e649331)
![image](https://github.com/user-attachments/assets/1a07a1f1-33e5-4d79-beda-bf196abe0406)

在获取到当前的环境变量之后，就可以执行交易了，首先回将交易区分成本地交易和正常交易，然后会按照矿工费从高到低打包本地交易，本地交易执行完成之后，再按照矿工费从高到低打包正常交易，交易具体的执行都在 commitTransactions 方法中进行：
![image](https://github.com/user-attachments/assets/c00b1fb2-810b-46a6-95a2-32e9c4fb2db4)

最终都是调用 ApplyTransaction 函数，在这个函数中，会调用 EVM 执行交易，并修改状态数据库：
```Go
func ApplyTransaction(evm *vm.EVM, gp *GasPool, statedb *state.StateDB, header *types.Header, tx *types.Transaction, usedGas *uint64) (*types.Receipt, error) {
	msg, err := TransactionToMessage(tx, types.MakeSigner(evm.ChainConfig(), header.Number, header.Time), header.BaseFee)
	if err != nil {
		return nil, err
	}
	// Create a new context to be used in the EVM environment
	return ApplyTransactionWithEVM(msg, gp, statedb, header.Number, header.Hash(), tx, usedGas, evm)
}
```

### 2025.03.16
交易是如何被节点处理的（5）
上面讨论的情况都是交易被打包进区块的流程，大多数情况下，节点只会验证已经被打包好的区块，而不是自己打包区块。

共识层在同步到区块之后，使用 engine API 将同步到的最新区块传输到执行层。使用的是 engine_NewPayload 系列方法。这系列的方法最后都会调用 `newPayload` 方法：

在这个方法中将共识层的 payload 组装成一个 block：
![image](https://github.com/user-attachments/assets/cd1dd532-f182-4784-a1c6-6ef3268e22df)

然后检查这个区块是否已经存在了，如果存在了，那么就直接返回取消有效的状态：
![image](https://github.com/user-attachments/assets/e4a465cd-8d4b-4b46-8a62-22f9e939a638)

如果当前执行层还在同步状态，那么暂时就无法接收新的区块：
![image](https://github.com/user-attachments/assets/df01208d-d1e7-487f-a885-46df36785c0d)

如果上面的条件都满足，那么就开始将区块插入到区块链中，这里需要注意，在插入区块的时候不会直接指定链头，因为链头的决策会涉及到链分叉的选择，这个需要依靠共识层来决定：
![image](https://github.com/user-attachments/assets/b9175034-6f86-4d13-9b9a-9c895d0d9725)

共识层会调用 forkChoiceupdated API 来调用 core/blockchain.go 中的 SetCanonical  方法来决定区块头：
![image](https://github.com/user-attachments/assets/c504f943-4985-4e59-ae4b-4516e9c89e7c)

还有一种情况会触发区块头的设置，那就时区块发生重组，区块重组会执行 `core/blockchain.go` 的 `reorg` 方法，在这个方法中同样会设置当前最新确定的区块头。 

回到区块的执行过程，内部方法会调用 `insertChain` 方法，在这个方法中，会做一系列条件的检查，检查完成之后，就会开始处理区块：
![image](https://github.com/user-attachments/assets/351c58bf-ccfa-4caa-b739-0ae804b90b75)
![image](https://github.com/user-attachments/assets/8c013c96-a9bd-43f3-9ed3-56a6ea570816)

在具体的 Process 里面，处理逻辑就很清晰了，和之前打包交易的流程类似，不断在 evm 中执行交易，然后修改状态数据库，与打包不同的地方在于，这里只是将新区块中的交易重放一遍，而不需要去交易池中获取。
![image](https://github.com/user-attachments/assets/5d8ac890-f025-48b5-a655-1274ef58a9a0)

### 2025.03.17
在 cmd/geth/main.go 的 startNode 中启动了 p2p 模块，具体在 utils.StartNode 中实现：
![image](https://github.com/user-attachments/assets/5b8f8387-be10-4daf-9bf8-541151ee2ad7)
![image](https://github.com/user-attachments/assets/636a5e85-b755-4b59-9370-b9e46e4b3c63)
![image](https://github.com/user-attachments/assets/7cc041c5-d27e-4845-a1a1-b5cec97078bf)

在 eth/backend.go 中的 New 函数中，注册当前需要启动的 p2p 的协议，具体的协议在 eth/ptotocols 中实现：
![image](https://github.com/user-attachments/assets/20ca22c8-4e89-40db-b3e9-886540362ab1)

在启动的时候，会先启动 p2p 服务，然后会启动节点的 RPC 服务：
![image](https://github.com/user-attachments/assets/7ccca72c-66f8-4899-9d40-8610a3e75f0c)

在 Start 方法中，会启动节点本身以及节点的发现服务：
![image](https://github.com/user-attachments/assets/50888072-00f5-4a49-aed2-6a376ea6f0f6)


<!-- Content_END -->
