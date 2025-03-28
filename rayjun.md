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


### 2025.03.18
以太坊的 p2p 协议实现是 devp2p，包含了以太坊中用到的所有 p2p 协议，libp2p 经常会和 devp2p 一起比较，总的来说，devp2p 是为以太坊协议开发的，而且 libp2p 是一个独立的 p2p 协议库，不和具体的应用绑定。devp2p 也在逐步采用 libp2p 的实现。

devp2p 有三个核心组件：

- 节点发现服务（Discv4/Discv5）：用于和其他节点进行交互，并将发现的节点信息保存在本地，代码实现在 `p2p/discover` ，其中使用 Kademlia DHT 算法来维护 p2p 网络的拓扑。
- ENR（Ethereum Node Record）：用于存储发现的节点信息，发现服务中发现的节点信息会被保存为 ENR 格式，代码实现在 `p2p/enr`。
- RLPx 传输协议：当前执行层的数据都是编码成 RLP 格式，RLPx 就是节点之间传输数据所采用的协议的，代码实现在 `p2p/rlpx` 。

基于 devp2p 实现的协议的都在 `eth/protocols` 中，其中 `eth/protocols/eth` 协议是其中主要的一个实现，主要用于区块和交易的传播，在之前分析交易源码的流程中已经说到这个协议的作用。还有 snap 协议用于具体节点的快速同步，代码实现在 `eth/protocols/snap`。

### 2025.03.19
eth 是 devp2p 的一个子协议，用于管理节点的区块、交易和状态数据的传播，当前协议的版本号是 eth68。

通过上面的代码可以知道以太坊节点在启动的时候会通过 `eth/backend.go` 中的 `Ptotocols` 方法来获取当前需要启动的协议。

eth 中节点和对等发现功能的实现其实就是封装了 devp2p 中的实现，并在这个基础上增加了 eth 协议的功能：
![Clipboard_Screenshot_1742350589](https://github.com/user-attachments/assets/73405660-7318-4c3e-86cd-42d4f62f5894)

节点的信息被组装成 enr，发现新的节点之后，将会将 enr 添加到路由表中，然后由 discv5 来实现路由表的更新：
![Clipboard_Screenshot_1742350599](https://github.com/user-attachments/assets/14077133-a5c5-4e80-ab86-08e126ea8555)

通过 MakeProtocols 方法来组装具体的协议：
![Clipboard_Screenshot_1742350613](https://github.com/user-attachments/assets/65df1211-1ff7-49cf-94a1-97f2b7b68199)

为每一个协议启动一个 peer 节点，并且为每个 peer 配置一个处理消息的 Handler：
![Clipboard_Screenshot_1742350624](https://github.com/user-attachments/assets/4531e726-b138-4dfc-aaff-0b3531626d7b)

在 RunPeer 方法中会启动 peer 节点的握手流程，p2p 的握手流程其实就是把节点自身的信息发布出去：
![Clipboard_Screenshot_1742350635](https://github.com/user-attachments/assets/4d1781cd-5404-49d4-b6e9-21efadb20823)
![Clipboard_Screenshot_1742350648](https://github.com/user-attachments/assets/9a16eb10-2e01-42d0-8a72-4f2634c126ee)

Peer 节点启动之后，会持续处理消息：
![Clipboard_Screenshot_1742350664](https://github.com/user-attachments/assets/d27b9694-f8da-45a4-8bda-cedd97c3a10d)

当前用来处理消息的协议是 eth68:
![Clipboard_Screenshot_1742350675](https://github.com/user-attachments/assets/73f399bc-8503-44c9-bef1-a64e88803669)

下面是当前 eth68 协议支持的消息类型，实际上 NewBlockMsg 和 NewBlockHashesMsg 已经不在执行层处理了，而是在共识层处理，所以这里的两个 Handler 为空：
![Clipboard_Screenshot_1742350690](https://github.com/user-attachments/assets/eff712ca-c903-403b-b96d-ff1aca7e5dba)


### 2025.03.20
对于交易的一些补充：

对于 blob 交易或者一些大的交易：
![image](https://github.com/user-attachments/assets/9a36f93b-58bb-4a8e-8c8b-179c460ef0cb)

在实例化 handler 的时候，会指定好从其他节点获取交易的方式，会通过 `eth/fetcher 中的 TxFetcher 去获取远端的交易，TxFetcher 会通过这里的 fetchTx 去获取远端的交易，实际是调用了 `eth/protocols/eth` 协议中实现的方法去获取交易：

![image](https://github.com/user-attachments/assets/d05104c6-5e72-44dd-93a3-3b0551aa8743)

通过发送 GetPooledTransactionsMsg 消息，然后收到 PooledTransactionsMsg 的响应：
![image](https://github.com/user-attachments/assets/39290c4c-1c66-4c90-a879-2f61dc7a77cc)

最终还是由 txFetcher 的  Enqueue 来处理：
![image](https://github.com/user-attachments/assets/a7ff66a6-2bb1-4c88-a024-ad4db6f91093)

然后调用 addTxs 来将交易添加到交易池，这个方法在 newHandler 中已经声明，就是调用交易池的 Add 方法来将交易打包进交易池：
![image](https://github.com/user-attachments/assets/e073bbe2-d070-4a95-8ab3-b8118f3c746b)

### 2025.03.21
snap 协议用来完成状态的快速同步，在 full 的同步模式一下，节点同步到区块之后，会通过执行区块的中的交易来恢复状态数据库，但是这样会导致非常耗时，而 snap 协议在 snap 的同步模式下会启用，允许节点快速恢复状态数据库，而不需要从创世区块开始验证每个区块。

在节点启动时，在 `eth/backend.go` 的 newHandler 中，会初始化 snap 协议：
![image](https://github.com/user-attachments/assets/cf5741f1-3191-4f35-9b73-5d554dbe64da)

![image](https://github.com/user-attachments/assets/2393bc9c-b924-4960-a6f7-f3edf5e816ce)

在前面讨论 eth 协议时由说道每启动一个协议节点，就会创建 peer 节点，并且会与其他的节点进行握手通信，在这个过程中，也会在 snap 协议上握手，具体实现在 eth/handler.go 的 runEthPeer 方法中：
![image](https://github.com/user-attachments/assets/b7cbb1c0-5bb9-4cca-8d1a-e89815b8014b)

在 snap 协议中，Task 是一个很重要的概念，Task 有三类，**accountTask、storageTask、healTask。**

- accountTask：代表账户快照的同步任务。每个 **`accountTask`** 负责处理一段账户范围的同步，包括从远程对等节点获取账户数据。
- storageTask：代表存储快照的同步任务。它负责处理特定账户的存储数据的同步，storageTask 会作为 accountTask 的子任务存在。
- healTask：代表修复任务，用于处理同步过程中状态不一致或者错误的修复。

其中同步过程以 accountTask 为维度拆分。
![image](https://github.com/user-attachments/assets/ff3b0a83-9254-4c71-8f8e-d85d2ee711f9)
![image](https://github.com/user-attachments/assets/86aa6219-ac8a-4cdc-a3f1-b8668ab06b54)
![image](https://github.com/user-attachments/assets/c9a62b7d-9ef1-4a86-8746-1770b62e0da8)

状态同步的逻辑会通过 Sync 方法来完成，root 参数是需要同步的状态树的根哈希值：
![image](https://github.com/user-attachments/assets/4f6283df-f53f-4e35-a5f9-84e309908aed)
在开始同步的时候，会创建一个 healTask：
![image](https://github.com/user-attachments/assets/a5ec86f1-d114-47fd-8dc2-296432c53237)

调用 loadSyncStatus() 方法从数据库中检索之前的同步状态，如果状态不存在，那么就重新启动一个的状态同步过程：
![image](https://github.com/user-attachments/assets/ecca09c1-9922-47bb-a140-56d4c45566b4)

如果所有的同步任务和修复任务都完成了，则结束同步流程：
![image](https://github.com/user-attachments/assets/f58bcf9b-d1b5-41a8-917b-c5ff30cc5f53)

这里开始进入通过的核心逻辑，通过 for 循环保证同步的持续进行，在同步的过程中也会判断任务和修复过程是否完成，如果完成了，就会退出同步过程。通过会将任务分发给已经组成的 peer 节点去完成：
![image](https://github.com/user-attachments/assets/0a3fcd21-ee2c-41d6-a0d4-a48188e2c37c)

通过 select 机制来处理各个任务的结果：
![image](https://github.com/user-attachments/assets/1a7d9a3f-2a1d-4431-93d8-22e4afaac3a1)


### 2025.03.22
**`downloader`** 模块的主要目标是从远程对等节点下载区块和相关数据，以确保本地节点的区块链状态与网络中的最新状态保持一致。该模块支持不同的同步模式，包括完整同步和快照同步。

当前 go-ethereum 只支持了两种同步模式：

- FullSync
- SnapSync

downloader 同样在 `eth/handler.go` 的 newHandler 中初始化：
![image](https://github.com/user-attachments/assets/ca632341-abb7-487e-8671-694e43849b7d)

初始化状态同步的框架：
![image](https://github.com/user-attachments/assets/86230535-b0d0-494e-b3e6-4764eac4bb6e)

然后启动同步：
![image](https://github.com/user-attachments/assets/4ed14f53-1a97-40cf-95ef-aad0028e0577)

在接收到新区块头事件之后，就会开始同步：
![image](https://github.com/user-attachments/assets/3176b8cb-9593-48ff-8bd2-472226cc23ff)

最终会调用 resume 方法来拉取区块和状态：
![image](https://github.com/user-attachments/assets/5bf63dd0-6370-4c26-8eed-ea04c59a2402)

然后调用 downloader 的 synchronise 方法来真正同步数据：
![image](https://github.com/user-attachments/assets/cbd3883c-d336-4393-9e27-de83a33787e6)

然后调用了 syncToHead 方法：
![image](https://github.com/user-attachments/assets/101dd22b-ebdb-4e22-8c01-d226a2ea5823)

在这个方法中定义了各类 fetchers 去拉取不同的数据：
![image](https://github.com/user-attachments/assets/f0be8fe5-e85d-466a-9a63-26b61c88c1e9)


### 2025.03.23

在以太坊中每一个节点都使用 ENR 的数据结构来表示，ENR 在 EIP-778 中定义，每个节点必须提供 ip 和 UDP 的端口（discv5 是构建在 UDP 协议之上），这些都会被节点记录，如果节点所在的网络是 NAT 网络，这需要通过 UPnP/NAT-PMP 协议来做转换。

每个节点都有一个 32 字节的 ID，这里有个很关键的概念：节点之间的距离，这个距离不是真实的距离，而是逻辑上的距离，是两个节点 ID 通过异或操作得到的：

```go
distance(n₁, n₂) = n₁ XOR n₂

// 但是在实际的使用中，会使用对数距离
logdistance(n₁, n₂) = log2(distance(n₁, n₂))
```

两个节点在开始通信之前，需要进行握手获取会话密钥的流程，握手是为了防止有人伪造节点进行女巫攻击，而且会通过 Nonce 和签名机制来防止重放攻击。对于区块链节点来说，会同时承担客户端和服务端的职责。

假设现在有节点 A 想要（A 此时有 B 的 ENR 信息）和节点 B 进行通信，分为两种情况：

- 如果 A 和 B 之前已经交换过会话密钥，直接开始通信
- 如果没有会话密钥，则需要进入握手流程
    - A 首先发送随机消息
    - B 响应 WHOAREYOU 数据包（discv5 中定义的数据包类型），并在请求中加上随机生成的 nonce 信息，此时 B 还会保留 A 记录一段时间
    - A 接收到信息之后，会对 WHOAREYOU 和附加的一些信息进行签名，然后发送 FINDNODE 请求，并且请求内容使用新生成的会话密钥加密
    - B 接收到请求之后，会用第二步中存储的 A 节点信息中的公钥来验证签名，如果本地没有，那就从 A 发送的请求中获取，然后使用同样的方式和参数来生成会话密钥，然后解密 A 发送的消息，如果解密成功，那么就会发送一条消息的影响消息，比如 NODES，然后使用会话密钥加密
    - A 接收到响应之后，就会使用会话密钥对消息进行解密，如果解密成功，那么握手成功，两个节点就都保留了对方的 ENR 信息

女巫攻击：女巫攻击是一种常见的网络攻击方式，攻击者通过创建多个虚假身份（或节点）来控制网络中的大部分节点。
日蚀攻击：日蚀攻击是一种针对 P2P 网络的攻击，攻击者通过控制一个节点的所有入站和出站连接来“遮蔽”该节点。这使得被攻击的节点无法与网络中的其他节点进行有效通信。

### 2025.03.24
Kademlia 是一种高效的分布式哈希表（DHT）算法，广泛用于点对点（P2P）网络中，特别是在节点发现和数据存储方面。通过节点 ID 来判断两个节点之间的距离，然后会递归地查找这个节点，界限信息使用 K-buckets 数据结构存储

discv5 使用了一种类似 Kademlia 算法的方式来实现 p2p 模块，但在核心结构的设计上也所差别，没有直接使用 K 桶，而是使用了 Table 作为数据结构，实现更加复杂，实现了节点的初始启动还有异步请求处理等等问题。

discv5 被用来维护节点信息结构，在 `p2p/server.go` 中，会调用 setupDiscovery 方法来开始维护底层节点：
![image](https://github.com/user-attachments/assets/c1b214dc-5a5a-4dd8-bea0-806380b74474)

discv5的底层使用是一个 p2p/discover/table.go 的数据结构用来维护 p2p 网络的结构，节点发现和管理使用的是一种类似 Kademlia 算法的分布式哈希表算法：
![image](https://github.com/user-attachments/assets/5ce3d420-0a0e-48fb-a6c6-def12811c610)

在这个方法里会实现底层路由的表的维护和更新：
![image](https://github.com/user-attachments/assets/f5aa27a6-0954-4aac-8bb3-660c34ffe46a)

discv5 是通过 UDP 来实现的，在 ListenV5 中会初始化 discv5 协议：
![image](https://github.com/user-attachments/assets/76c901aa-79f0-4cc4-a658-88440b74c1da)

newTalkSystem 初始化 talkSystem 实例，负责管理与其他节点的通信，table 实例负责管理节点，用于存储和维护网络中的节点信息：
![image](https://github.com/user-attachments/assets/f2c67264-6beb-4b7b-b3c3-58e873ea9067)

table 在初始化时，会加载种子节点（seed nodes）和备用节点（fallback nodes），以确保节点表在启动时具有有效的节点信息。
![image](https://github.com/user-attachments/assets/0c1465be-e81b-469a-97b7-49142f44e1fc)

discv5 模块的节点发现依赖 lookup 模块来完成，当前需要查找特定节点时，会调用 Lookup 方法来完成查找，实例化一个 lookup，并运行：
![image](https://github.com/user-attachments/assets/82613fa2-17d3-4cbc-a9e5-90942de6d08e)

发送查询的请求，并对响应的结果进行处理：
![image](https://github.com/user-attachments/assets/4f149561-9a29-4079-977d-58dd607e147a)


### 2025.03.25
节点之间建立连接之后，节点会在一段时间内缓存会话密钥，以 IP 和端口为 key，会话密钥为 value，通过 LRU 来进行缓存。

节点需要保存他们的邻居节点，这个节点表由 k-buckets 组成，按照节点之间的距离来划分为 256 个桶，每个桶最多存储 16 个节点信息，节点按 **最后通信时间** 排序，**最久未活跃的节点在头部**，最新活跃的节点在尾部：

```go
logdistance(self, n) == i, 0 ≤ i < 256, k = 16
```

新节点插入逻辑：

- 桶没满：
    - 将新节点插入到桶的头部（最久未活跃的位置）， 比如桶中有 10 个节点，新节点 `N₁` 插入后成为第 11 个节点，位于头部
- 桶已满：
    - 需验证桶中 **最久未活跃的节点 `N₂` 是否存活**，
        - 向 `N₂` 发送 Ping 消息
        - 若 `N₂` 回复 Pong，则保持桶不变
        - 若 `N₂` 无响应，将其移出桶，并插入新节点 `N₁` 到头部

每个节点的 ID 都是使用公钥作为哈希算法（当前使用的是 **Keccak-256 哈希算法**）的参数计算得出，所以这里的距离不是真正意义上的距离，而只是逻辑距离。实际网络中几乎不存在逻辑距离为 0 的点，所以在实际的存储中，可以忽略距离很小的节点，减少内存占用和节点开销。

节点需要定期和节点表中的相邻节点通信，会通过以下策略来验证节点的活跃度（每个节点有一个活跃度标识），因为都全量验证不太现实，容易造成 Dos 攻击：

- 会定期查找**最近刷新次数最少**的存储桶，避免全量检查
- 偶尔随机抽查节点是否活跃，并且对多次通过活跃性检查的节点，允许偶发无响应
- 在响应 FINDNODE 请求时，只返回活跃度标记位是 True 的节点
- 在桶已满时，缓存已经经过验证的新节点，当桶内某节点被标记为失效时，从替换缓存中选择**最近活跃的节点**替换

是否活跃标记位：
![image](https://github.com/user-attachments/assets/d0ff17c3-3480-4f09-a566-978146766862)

缓存已经验证的新节点：
![image](https://github.com/user-attachments/assets/e3b6020b-0f3c-44c0-85b7-8dc411925434)


### 2025.03.26
lookup 是一个核心功能，负责查找对应距离目标节点最近的 k 个节点。具体流程如下：

- **选择初始节点**：从本地路由表中选取 **α 个最接近目标** 的节点（通常 α=3）
- 向选择的三个节点发送 `FINDNODE` 请求
- 每个被查询节点返回其路由表中满足要求的节点
- 合并所有响应节点，按距离排序，从列表中选取 **α 个未查询的最近节点**，重复步骤 2-3
- 当已查询并收到来自当前 **k 个最近节点** 的响应，请求结束

优化方式：

- 多路查询：通过独立路径降低恶意节点影响，提高查找成功率
    - **分割初始节点**：将初始的 k 个最近节点分为 **m 个路径**（如 m=3）
    - **独立执行基本流程**：每个路径维护自己的候选列表和查询历史
    - **禁止跨路径复用**：路径 1 发现的节点不用于路径 2 的后续查询
    - **记录已查询节点**：所有路径共享已查询节点列表，避免重复请求
    - **综合各路径结果**：取所有路径中找到的最近 k 个节点
- 动态调整查询距离
    - 当初始查询距离 d 返回节点不足时，增加距离继续查询

在 `p2p/discover/lookup.go` 中，会初始化 lookup 结构：
![image](https://github.com/user-attachments/assets/20cf5b0f-07a1-499f-ac96-5b4bba9d94fe)

启动 lookup 流程：
![image](https://github.com/user-attachments/assets/94211d02-d67d-4f44-96b5-231387f0c6af)

一路跟进代码发现最终执行 lookup 的方式在这里实现：
![image](https://github.com/user-attachments/assets/b9181a61-0ed0-4745-8e22-c518e3ff1d21)

实际调用的方法是在这里配置的：
![image](https://github.com/user-attachments/assets/72f5a0d0-d953-41d8-b312-132f7c0ffdf1)

执行 Findnode 操作：
![image](https://github.com/user-attachments/assets/45271590-46a8-4326-95f1-6388e87b21d1)


### 2025.03.27
在构建分布式应用时，通常希望将搜索限制在提供特定服务的参与者，节点通过发布"主题"(Topic)声明自身服务能力（如文件共享、轻节点服务等），其他节点可根据 Topic 搜索匹配的服务提供者。到目前为止，**Topic Advertisement** 还没有在 go-ethereum 中实现。

- 协议的核心作用：服务发现，允许节点通过主题快速找到服务提供者（如轻节点客户端寻找中继服务器）
- 核心功能：
    - 服务注册和声明：节点通过发布「主题」（如 `eth-light-client`）声明自身服务能力
    - 所有参与协议的节点均承担 Topic 存储功能：
        - **主题表(Topic Table)**：采用三级存储结构（全局表→主题队列→广告条目），单个主题队列容量上限100条，全局上限5万条
        - **防滥用机制**：限制同一节点在单个队列的重复注册
    - 流量控制和公平调度
        - 通过 Ticket 机制确保 Topic 注册的公平性和网络稳定性，Ticket 包含节点身份、IP、时间戳等信息，防止伪造
    - 动态广告分布
        - 定义节点 ID 与 Topic 的逻辑距离，强制要求最小半径，防止 Topic 过度集中导致单点故障
    - 防止 Topic 滥用
        - Ticket 加密，确保不可伪造
        - 限制同一 节点/IP 的注册频率
        - 全局 Topic 数量不能超过 5 万


### 2025.03.28
ENR（EIP-778）是用于存储和传播节点信息的格式，标准化了 p2p 网络中节点的身份验证和连接信息，ENR 由三个部分组成：

- 签名（s**ignature**）：对 ENR 内容的签名，确保数据完整性和来源可信
- 序列号（**seq**）：记录更新的计数器，递增更新
- 键值对（pairs）：节点信息的键值对，enr 有一组默认的键值对
    - id：身份方案名称，当前为 v4
    - secp256k1： 压缩公钥
    - ip：IPv4 地址
    - tpc/udp：端口号
    - ip6: IPv6 地址
    - tcp6/udp6：端口号
- 签名生成：
    - content = RLP([seq, k1, v1, k2, v2, ...])
    - 对 `content` 计算 Keccak256 哈希
    - 使用私钥对哈希值生成 ECDSA 签名

ENR 内容使用 RLP 方式编码，按照 `[signature, seq, k1, v1, k2, v2, ...]` 的方式（递归长度前缀）序列化。

在 go-ethereum 中对应的实现：
![image](https://github.com/user-attachments/assets/20cb2871-b763-4103-b586-a173f7fd54c9)



<!-- Content_END -->
