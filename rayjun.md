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


### 2025.03.29
在传统的 p2p 网络中，会通过硬编码启动节点的方式来帮助节点冷启动，devp2p 则通过 DNS 来实现动态、可验证的节点列表分发（EIP-1459）。这样节点只需要提前知道 DNS 域名和公钥，就可以安全地获得最新节点信息，而不需要更新软件。

NodeList 是一个组件，主要有三部分内容组成：

- ENRs：包含多个 ENR 记录或者指向其他 NodeList 的链接
- 签名：包含内容的签名，客户端可以使用公钥验证完整性
- 序列号：通过序列号更新版本，防止回滚攻击

代码在 `p2p/dnsdisc/sync.go` 和 `p2p/dnsdisc/tree.go` 中实现：

![image](https://github.com/user-attachments/assets/b292b1a8-68f6-45bd-b536-d595890f7ab2)

![image](https://github.com/user-attachments/assets/defbbb23-896f-4a50-b08f-ac258b5a306a)



NodeList 内容由 Merkle 树组成，并且会对这些内容签名，无论是记录内容被篡改还是签名被篡改，都会导致验证无法通过

- url格式：enrtree://<base32(公钥)>@<DNS域名>
- DNS 记录结构：
    - 根记录：enrtree-root:v1 e=<enr-root> l=<link-root> seq=<序列号> sig=<签名>
        - enr-root：enr 子树的根 hash
        - link-root：其他 NodeList 子树的根 hash
        - 签名：对根记录内容的签名
    - 子树记录
        - 分支（Branch）：enrtree-branch:<哈希1>,<哈希2>,…
        - ENR 叶子（Leaf）：enr:<Base64URL(ENR)>
        - 其他 NodeList 链接（Link）：enrtree://<公钥>@<目标域名>

客户端协议的处理流程：

- 获取根记录
    - 首先获取 DNS 域名的根记录，解析 enrtree-root
    - 使用公钥验证签名，检查序列号是否递增
- 遍历 Merkle 树
    - 解析 enr-root 和 link-root
    - 处理记录
        - 对于分支，递归查询所有的子树
        - 对于 ENR 叶子，解码 ENR 并验证签名，存入本地
        - 对于 link，将新域名加入待解析队列
- 优化：
    - 对于已经访问的域名和 hash 进行记录，避免重复查询
    - 按需加载，避免一次加载所有的内容

### 2025.03.30
RLPX 是以太坊 P2P 网络的数据传输协议，RLPx 以 RLP序列化格式命名，该名称不是首字母缩略词，也没有特殊含义。

在 p2p Server 启动的时候，这里需要初始化 RLPx 协议。
![image](https://github.com/user-attachments/assets/c8eb3356-345f-4a01-a647-bc4ed10407e2)

RLPx 协议有四个核心流程：
- 握手：通过非对称加密交换临时公钥，生成共享密钥
    - 发起方（Initiator）生成临时密钥对
    - 发送 Auth 消息
    - 接收方（Responder）处理 Auth 消息
    - 接收方发送 Ack 消息
    - 双方生成共享密钥
- 认证：验证握手完整性，建立加密通道
    - 交换签名信息
    - 初始化加密引擎：双方使用 AES-256-CTR 和 HMAC-SHA256 初始化加密/解密器
- 会话初始化：协商协议版本和支持的子协议
    - 交换 Hello 消息，使用 RLP 编码 `Hello` 消息，通过已建立的加密通道发送
    - 协议协商：双方比较 Hello 消息中 `capabilities`，选择共同支持的最高版本子协议，例如，若双方均支持 `eth/64` 和 `snap/1`，则启用这些协议
- 数据传输：通过分帧机制传输加密数据
    - 分帧：数据被拆分为多个帧（Frame），帧结构如下：
        - Frame Type: 帧类型（如 0x01 表示常规数据，0x02 表示协议控制）
        - Size: 负载长度（3 字节大端序）
        - Payload: 实际数据（RLP 编码）
        - MAC: 消息认证码，防止篡改
    - 多路复用（Multiplexing）
        - 不同子协议（如 `eth`、`snap`）通过帧类型（Frame Type）区分，比如 `eth` 协议的区块数据使用帧类型 0x01，`les` 的请求使用 0x02
    - 数据加密与发送
        - 发送方对每帧的 Payload 进行 AES-CTR 加密
        - 计算 HMAC 摘要，附加到帧尾，然后发送
    - 数据接收与处理
        - 接收方验证 MAC，解密 Payload
        - 根据帧类型分发到对应的子协议处理器，例如，`eth` 协议处理器解析区块数据并更新本地链状态
- 连接终止（Termination）
    - 正常关闭，发送 `Disconnect` 消息，包含断开原因（如 `REQUESTED` 或 `NETWORK_ERROR`），双方释放连接资源
    - 异常处理，检测到无效 MAC 或超时，立即终止连接

### 2025.03.31
在 p2p/server.go 的 setupConn 中处理 RLPx 连接：
![image](https://github.com/user-attachments/assets/65134d26-2033-4640-99c2-a10882bcf684)

在 server 启动的时候，会设置节点进行连接的方法：
![image](https://github.com/user-attachments/assets/beae1b6b-b19e-4e16-84e8-c4cddaba86f3)

最终在 p2p/dial.go 中的 dial 方法中调用 SetupConn 来为节点创建 RLPx 连接：
![image](https://github.com/user-attachments/assets/0d27a2bb-ff22-4922-a80a-e6c4f1304207)

在 RLPx 通信的过程中，使用的加密方式为 ECIES (Elliptic Curve Integrated Encryption Scheme) 。

- ECIES 加密流程
    - 参数选择：
        - 椭圆曲线：secp256k1。
        - 密钥派生函数（KDF）：NIST SP 800-56 连接模式。
        - MAC：HMAC-SHA256。
        - 对称加密：AES-128-CTR。
    - 加密步骤（Alice → Bob）：
        - 生成随机数 `r`，计算临时公钥 `R = r*G`。
        - 共享密钥：`S = ecdh(r, Bob的公钥Kb)`。
        - 派生密钥：`kE || kM = KDF(S, 32)`（加密密钥和 MAC 密钥）。
        - 加密消息：`c = AES(kE, iv, 明文)`。
        - 生成认证码：`d = HMAC-SHA256(kM, iv || c)`。
        - 发送数据：`R || iv || c || d`。
    - 解密步骤（Bob）：
        - 使用私钥计算共享密钥 `S = ecdh(Bob的私钥kb, R)`。
        - 派生相同 `kE` 和 `kM`。
        - 验证 `d` 的 MAC，解密 `c` 获得明文。
- **节点身份**
    - 每个节点维护静态 secp256k1 私钥，建议通过删除文件手动重置。
    - 公钥通过私钥生成，作为节点唯一标识。

其中 RLPx（基于 TCP） 和 Discv5（基于 UDP）是协同工作的，总体来说 Discv5 负责发现节点，并将节点的 ENR 数据保存在本地，而 RLPx 协议中使用 ENR 中的信息来建立连接，高效传输数据。

- 分工流程
    - 节点发现（discv5）：
        - 通过 UDP 发现邻近节点，获取其 ENR（含 IP、端口、公钥）。
    - 建立连接（RLPx）：
        - 使用 ENR 中的 IP 和端口，通过 TCP 发起 RLPx 握手。
        - RLPx 完成加密通信，传输区块、交易等数据。

### 2025.04.01
Les（**Light Ethereum Subprotocol**）是专为资源受限设备设计的协议，允许轻客户端仅同步区块头并按需获取其他区块链数据（如交易、状态证明等），同时不参与共识过程。全节点和归档节点都可以支持 Les 协议，为轻节点提供支持。

核心设计目标：
- 低资源消耗
- 安全性：通过 Merkle 机制（如果以太坊的底层存储结构换成了 Verkle，那么这块就需要重新设计）来验证数据的完整性，而不需要信任全节点
- 支持交易转发、状态查询、智能合约调用等全节点功能（需要依赖全节点来完成）

核心数据结构：
- Canonical Hash Trie, CHT**：**快速同步和安全检索区块哈希与总难度（TD）值（在 PoS 下，TD 已经没有意义了，这块又需要重新设计）
    - Merkle Patricia Trie：存储`blockNumber -> [blockHash, TD]`映射，键为64位大端编码整数，值为RLP编码的`[hash, number]`对。
    - 生成规则：每 32768 个区块生成一个 CHT（如`CHT[i]`覆盖区块0至`(i+1)*32768-1`），并在2048 区块确认后固化，避免因链重组失效。
    - 验证流程：
        - 客户端已知CHT根哈希后，通过`GetHelperTrieProofs`请求特定区块的Merkle证明
        - 服务器返回区块头及CHT路径节点，客户端可验证哈希是否匹配根哈希
- BloomBits 结构**：**高效日志搜索（如按地址或主题过滤交易）
    - 位转换：将区块的Bloom过滤器按位索引重组，形成按位索引的位向量（Bit Vector）
    - 分块存储：每32768个区块为一组，`BloomBits[bitIdx][sectionIdx]`存储该组中所有区块Bloom过滤器的第`bitIdx`位
    - 压缩算法：
        - 稀疏数据优化：仅存储非零字节，通过`nonZeroBitset`和`nonZeroBytes`压缩数据，减少传输量
        - 检索流程：客户端请求三个 BloomBits 位向量进行位与操作，快速筛选匹配区块
- 客户端流量控制模型
    - 成本模型：服务器为每个请求分配成本（基于消息类型和参数），客户端需消耗“缓冲区值”（Buffer Value, BV）发送请求
    - 参数定义：
        - BL（Buffer Limit）：客户端最大缓冲区容量
        - MRR（Minimum Recharge Rate）：缓冲区最低恢复速率（单位：成本/秒）
        - MRC（Max Request Cost）：各消息类型的最大成本表
    - 服务端：
        - 客户端连接时，初始 BV 设为 BL
        - 每请求扣除对应成本，若 BV<0 则断开连接
        - 响应时返回更新后的 BV（含处理期间的恢复值）
    - 客户端：
        - 维护最低BV估计值（BLE），仅在 BLE ≥ 请求最大成本时发送请求
        - 动态调整请求速率，优先选择负载较低的服务器

### 2025.04.02
Tire 是以太坊的核心数据结构，用于存储以太坊的状态数据，使用的数据结构是：Merkle Patricia Trie（MPT），它结合了 Merkle Tree 和 Patricia Trie 的特点。

- Merkle Tree
    - 每个节点包含其子节点的哈希值，不是严格的二叉树，但是在以太坊中使用的是二叉树结构
    - 根节点的哈希值可以验证整个数据集的完整性
    - 支持轻客户端验证
- Patricia Trie
    - 前缀树（Trie）的一种变体
    - 可以实现路径压缩
    - 可以提供高效的查找和更新

主要用途：

- 存储账户状态（状态 trie）
- 存储交易收据（收据 trie）
- 存储区块头信息（状态根、收据根等）

```Go

type node interface {
    cache() (hashNode, bool)
    encode(w rlp.EncoderBuffer)
    fstring(string) string
}

type (
    fullNode struct {
        Children [17]node
        flags    nodeFlag
    }
    shortNode struct {
        Key   []byte
        Val   node
        flags nodeFlag
    }
    hashNode  []byte
    valueNode []byte
)
```

节点类型说明：
- valueNode：存储实际数据的叶节点
- shortNode：包含键和值的节点，用于路径压缩
- fullNode：包含16个分支的完整节点
- hashNode：存储节点哈希的引用节点


### 2025.04.03
在理解 MPT 之前，需要理解一些基础概念：

- tire：搜索树数据结构
    - 根节点为空
    - 每个节点代表一个字符
    - 前缀共享：不同字符串共享相同前缀路径（如"app"和"apple"共享前三个字符），减少冗余存储
    - 时间复杂度，与 key 的长度 m 有关，与数据量无关：
        - 插入：O(m)
        - 查找:  O(m)
        - 前缀匹配：O(m)
        - 删除: O(m)
    - 优点：
        - 高效前缀操作：天然支持前缀搜索（如自动补全）
        - 无 hash 冲突：查找时间与数据量无关
        - 有序性：可对键进行字典序遍历
    - 缺点：
        - 空间开销大，每条边仅表示单个字符，导致边数量多，存储开销大
        - 不通用，对非字符串或动态编码数据（如浮点数）支持较差
     
![image](https://github.com/user-attachments/assets/352d5c23-20ef-48ca-afb0-9bab3deb80c2)

- redix tree：对 trie 在空间利用率上做了很多优化
    - 特点：可以通过基数 r 的选择来决定每个节点可以拥有的最大子节点数，直接影响节点的深度和空间效率
        - 基数 r 通常取 2 的幂（如 r=2, 4, 8, 16, 256）
        - 键处理：
            - 将键按基数 r 对应的 **比特位数** 分割为多个块，每块作为一层节点的索引：
                - **r=2**：每次处理 1 位（bit），如 `0` 或 `1` → 节点分左右子树（**深度最深**）
                - **r=4**：每次处理 2 位（bit），索引值 0~3 → 每个节点最多 4 个子节点
                - **r=16**：每次处理 4 位（bit），索引值 0~15 → 每个节点最多 16 个子节点（后续节点逐渐稀疏）
            - 当 r 为 4 时，键 `0xAB`（二进制 `10101011`）的分块为：
                - 第1层：`10`（对应十进制 2）
                - 第2层：`10`（对应十进制 2）
                - 第3层：`10`（对应十进制 2）
                - 第4层：`11`（对应十进制 3）
            - 树的构建和查询，插入和查询键 0xAB：
                - 按 2 位分块，从根节点开始，依次检查第1块 `10` → 第2块 `10` → 第3块 `10` → 第4块 `11` ，若路径不存在，则逐层创建节点，直到键完全插入
                - 沿分块路径逐层跳转，若所有块均匹配，则命中
    - 路径压缩（Path Compression）**：**减少节点数量
        - 在 trie 中，apple 和 app 两个字符串存储需要 5 个节点
        - 在 radix tree 中，app 前缀会被压缩为一个节点，le 单独一个节点，再加上一个叶子节点，只需要 3 个节点就能完成存储
    - 边标签的优化（Edge Labeling）：减少边的数量
        - 在 trie 中，每条边代表单个字符
        - 在 radix tree 中，边的标签可以由多个字符组成
    - 时间复杂度：与 key 的长度 m 有关，与数据量无关
        - 插入：O(m)
        - 查找：O(m)
        - 删除：O(m)
    - 优点：
        - 空间高效：通过路径压缩减少冗余节点，尤其适合长字符串键或共享长前缀的键集合
        - 无 hash 冲突：查找时间与数据量无关
        - 有序性：可对键进行字典序遍历
    - 缺点：
        - 不通用，对非字符串或动态编码数据（如浮点数）支持较差
        - 基数 r 的选择需权衡深度与空间开销，可能影响性能
     
![image](https://github.com/user-attachments/assets/ee37b6fc-023f-462c-a406-05fd47125c99)

### 2025.04.04

- Patricia Trie： **在** radix tree 的基础上进一步优化
    - 消除单子节点，所有节点必须分叉或者是叶子节点
    - 引入扩展节点的概念：
        - 分支节点（Branch Node）：包含 16 个子节点指针和 1 个终值，支持多路径分叉
            - 最多 16 个子节点，通过索引直接访问，没有路径压缩能力，支持终值存储
        - 扩展节点（Extension Node）：存储共享前缀和指向下一节点的指针，压缩长路径
            - 只有一个子节点，支持路径压缩，不能存储终值
        - 叶子节点（Leaf Node）：存储路径末尾的键值对，表示路径终止
    - 时间复杂度：
        - 插入、查找、删除操作因路径压缩接近 O(log n)
    - 路径奇偶性和紧凑编码：
        - 计算机存储以字节（8 bits）为单位，而 Patricia Trie 的路径按半字节（4 bits，即十六进制字符 `0-f`）拆分，也就是一个半字节表示一个 16 进制的数字
        - 路径长度可能为奇数或偶数个半字节（例如，`0x123` 是 3 个半字节，`0x1234` 是 4 个半字节）
        - 问题：奇数长度路径无法直接转换为整数个字节，需填充对齐，例如路径 `0x123`（3 个半字节）需填充为 `0x01 0x23`（2 字节），否则无法存储为字节序列
        - 通过**紧凑编码**解决这个问题，紧凑编码的首个半字节包含标志位，标明**路径长度奇偶性**和**节点类型，**标志位告知解码器是否需要填充，确保路径能被正确还原
            - 0x0/0x1：如果是扩展节点，那么标志位使用 0x0 表示路径长度为偶， 0x1 表示路径长度为奇
            - 0x2/0x3：如果是叶子节点，那么标志位使用 0x2 表示路径长度为偶，0x3表示路径长度为奇
        - 编码示例（**这里需要注意，虽然标志位也是半字节，但是为了对齐，都会在后面补0**）：
            - 路径为 `0x1234`（偶数长度，扩展节点），偶数长度路径不用补齐
                - 标志位：`0x0`（扩展节点 + 偶数长度）
                - 紧凑编码：`0x0`（标志位） + `0x12 0x34` → 字节序列 `0x00 0x12 0x34`
            - 路径为 `0xF`（奇数长度，叶子节点）
                - 标志位：`0x3`（叶子节点 + 奇数长度）
                - 填充：路径变为 `0x0 0xF`
                - 紧凑编码：`0x3`（标志位） + `0x0F` → 字节序列 `0x30 0x0F`


### 2025.04.05
- Merkle Tree
    - 每个叶子节点存储数据块（如文件片段、交易记录）的哈希值；每个非叶子节点存储其子节点哈希值的组合哈希（如 `hash(child1 + child2)`）
    - 根哈希（Root Hash）：树的顶部哈希，代表整个数据集的唯一指纹，用于全局完整性验证
    - 验证单个数据块仅需计算从叶子到根的路径哈希，复杂度为 O(log n)（n 为数据量）
    - 修改单个数据块只需更新对应路径的哈希，无需重构整棵树
    - 依赖哈希函数（如SHA-256）的抗碰撞性，任何数据变动都会导致根哈希变化
 
![image](https://github.com/user-attachments/assets/ef299a27-3455-47ef-8ae0-c5513805c783)

- Merkle Patricia Trie（MPT）**：**结合 Merkle Tree 的加密验证特性和 Patricia Trie 的高效检索能力
    - 树的特性与 Patricia Trie 一致
    - 所有的节点数据采用 RLP 编码
    - 应用场景：
        - 全局状态树（State Trie）
        - 交易树（Transactions Trie）
        - 收据树（Receipts Trie）
    - Merkle 树特性
        - 根哈希由所有底层数据生成，任意数据的修改会导致根哈希变化，确保状态唯一性
        - 通过节点哈希链可生成 Merkle Proof，验证特定路径值的存在性和正确性
        - 每个节点的哈希由其 **内容** 决定，遵循以下规则：
            - 叶子节点（Leaf Node） 和 扩展节点（Extension Node）：
                - 哈希值 = `keccak256(RLP编码([encodedPath, value/nextNode]))`
                - 若序列化后数据长度 ≥32字节，存储哈希值；否则直接存储节点数据
            - 分支节点（Branch Node）：
                - 哈希值 = `keccak256(RLP编码([child_0, child_1, ..., child_15, value]))`
                - 每个子节点引用其哈希值（若子节点数据长度 ≥32字节）
            - 空节点：哈希值为空字符串。
        - 当插入、修改或删除数据时，MPT 从叶子节点向上逐层更新哈希，直至根节点：
            - 叶子节点变更（例如更新账户余额）：
                - 修改叶子节点的 `value` → 重新计算其哈希
                - 父节点（可能是扩展节点或分支节点）的子指针更新为新哈希
            - 扩展节点变更（路径分叉）：
                - 若共享前缀路径 `encodedPath` 被拆分，生成新的扩展节点和分支节点 → 更新相关节点的哈希。
            - 分支节点变更（子节点或终值变化）：
                - 修改子节点指针或终值 → 重新计算分支节点的哈希
            - 根哈希更新：
                - 所有底层变更最终导致根节点的哈希更新 → 新根哈希写入区块头（`stateRoot`）


### 2025.04.06
hashNode 表示一个节点的哈希值
![image](https://github.com/user-attachments/assets/194695d6-7428-40cb-8c02-9c9a573208fa)

cache 方法用来判断一个节点及其子节点是否被修改，如果没有被修改，那么直接返回节点的 hash 值就可以，如果修改了，就重新计算 hash 值：
![image](https://github.com/user-attachments/assets/5011d358-6160-43a5-9f78-b4486c71265b)

先判断是否真的需要重新计算，有些只读的情况下，数据没有改变，不需要重新提交：
![image](https://github.com/user-attachments/assets/f359cde7-d277-430b-8475-3264fb4914a2)

如果真的需要重新提交，那么就重新计算 hash 值：
![image](https://github.com/user-attachments/assets/38ad6844-af88-4404-a799-927846ba0745)

commit 是一个递归的过程，从 root 到子节点的计算过程，如果子节点没有被修改，直接返回 hash 值，不需要重新计算：
![image](https://github.com/user-attachments/assets/2def9373-2d64-4580-9e4f-c9a2dca6a1a1)


### 2025.04.07
trie.go 文件实现了以太坊的 Merkle Patricia Trie (MPT) 数据结构的核心功能。主要包括以下的功能：
```Go
type Trie struct {
    root  node           // Trie 的根节点
    owner common.Hash    // Trie 的所有者标识（通常是账户地址）
    
    committed bool       // 标记 Trie 是否已提交
    unhashed int         // 自上次哈希操作以来插入的叶子节点数量
    uncommitted int      // 自上次提交以来的更新次数
    
    reader *trieReader   // 用于从底层存储检索节点的处理器
    tracer *tracer       // 用于跟踪 Trie 变更的工具
}
```

trie 的创建，者个函数创建一个新的 Trie 实例，如果提供了非空的根哈希，则从数据库加载根节点：
```Go
func New(id *ID, db database.NodeDatabase) (*Trie, error) {
    // 创建 trieReader 用于从数据库读取节点
    reader, err := newTrieReader(id.StateRoot, id.Owner, db)
    if err != nil {
        return nil, err
    }
    
    // 初始化 Trie 结构
    trie := &Trie{
        owner:  id.Owner,
        reader: reader,
        tracer: newTracer(),
    }
    
    // 如果根哈希不为空，则从数据库加载根节点
    if id.Root != (common.Hash{}) && id.Root != types.EmptyRootHash {
        rootnode, err := trie.resolveAndTrack(id.Root[:], nil)
        if err != nil {
            return nil, err
        }
        trie.root = rootnode
    }
    return trie, nil
}
```

get 方法是一个递归函数，根据节点类型进行不同的处理：

```Go
func (t *Trie) Get(key []byte) ([]byte, error) {
    // 检查 Trie 是否已提交（不可用）
    if t.committed {
        return nil, ErrCommitted
    }
    
    // 递归查询节点信息
    value, newroot, didResolve, err := t.get(t.root, keybytesToHex(key), 0)
    // .....
}

```

更新操作实际是做删除或者插入操作：
```Go
func (t *Trie) Update(key, value []byte) error {
    // 检查 Trie 是否已提交（不可用）
    if t.committed {
        return ErrCommitted
    }
    return t.update(key, value)
}

func (t *Trie) update(key, value []byte) error {
    // 增加未哈希和未提交计数器
    t.unhashed++
    t.uncommitted++
    
    // 将原始键转换为十六进制格式
    k := keybytesToHex(key)
    
    if len(value) != 0 {
        // 插入或更新键值对
        _, n, err := t.insert(t.root, nil, k, valueNode(value))
        if err != nil {
            return err
        }
        t.root = n
    } else {
        // 删除键值对
        _, n, err := t.delete(t.root, nil, k)
        if err != nil {
            return err
        }
        t.root = n
    }
    return nil
}

```

插入操作也是一个递归的过程，同样是根据不同的节点类型来做出相应的操作。

删除也是一个复杂的递归操作，根据节点类型进行不同的处理，并在删除后尝试简化 Trie 结构

在做完更新之后，需要将 trie 重新 commit。


### 2025.04.08
Trie 的 Commit 操作是以太坊状态管理中的关键环节，在需要将内存中的数据持久化的时候，就会实例化一个 Committer，它主要完成以下几个重要功能：

1. 持久化修改：Commit 操作将内存中的 Trie 修改持久化到底层数据库中。在 Commit 之前，所有对 Trie 的修改（插入、更新、删除）都只存在于内存中，通过 Commit 操作，这些修改会被永久保存
2. 计算和收集脏节点：Commit 会收集 Trie 中所有"脏"节点（即被修改过的节点），并将它们替换为对应的哈希引用。这些收集到的节点会被封装到一个 NodeSet 中返回，以便调用者可以将它们写入数据库
3. 生成状态根：Commit 过程中会计算 Trie 的根哈希，这个哈希是整个 Trie 状态的唯一标识符。在以太坊中，这个哈希会被用作区块头中的状态根，用于验证状态的完整性
4. 将 Tire 标记为提交状态：Commit 操作完成后，会将 Trie 标记为"已提交"状态，此时 Trie 不再可用于进一步的操作。这是因为 Commit 后的 Trie 中的节点已经被替换为哈希引用，需要重新创建 Trie 才能继续使用
5. 处理删除节点：Commit 会收集所有被删除的节点路径，并将它们添加到返回的 NodeSet 中，标记为已删除。这样数据库可以知道哪些节点需要被移除
6. 重置未提交计数器：Commit 完成后，会重置未提交修改的计数器，为下一轮操作做准备


### 2025.04.09
triedb 模块是trie 的持久化层，负责将 trie 的节点数据持久化到数据库中。简单来说，trie 和 triedb 的关系就像内存中的数据结构和磁盘上的存储系统：

- trie 是内存中的数据结构，负责高效的数据操作
- triedb 是持久化层，负责将 trie 的数据持久化到数据库
- 两者配合工作，trie 负责计算和操作，triedb 负责存储和检索

在需要处理状态变化的场景，triedb 使用 NodeDatabase 来创建 trie：
![image](https://github.com/user-attachments/assets/8ac80fe9-1023-4a9f-a42b-dacb4a8b97fb)

trie 在创建时，也需要接收一个 NodeDatabase 参数，使用 triedb 提供的 NodeDatabase 来创建 trie， trie 的所有读写操作都会通过 triedb 来访问实际的存储，当需要提交更改时，通过 triedb 来持久化这些更改：
![image](https://github.com/user-attachments/assets/d752d66e-26b4-4fd4-bb2c-cd58eac34307)

新建一个 DiffLayer，会在一个已经存在的 layer 上创建一个新的 Layer：
![image](https://github.com/user-attachments/assets/84f368d6-60e8-4a00-97c9-135a4d0d79e2)

在 triedb 中，使用 layer 的设计来实现不同层级的逻辑，让每层只专注处理当前的层的逻辑，而不需要关心其他层的设计：
```Go
type layer interface {
	
	node(owner common.Hash, path []byte, depth int) ([]byte, common.Hash, *nodeLoc, error)
	account(hash common.Hash, depth int) ([]byte, error)
	storage(accountHash, storageHash common.Hash, depth int) ([]byte, error)
	rootHash() common.Hash
	stateID() uint64
	parentLayer() layer
	update(root common.Hash, id uint64, block uint64, nodes *nodeSet, states *StateSetWithOrigin) *diffLayer
	journal(w io.Writer) error
} 
```

在 triedb 中有 diffLayer 和 diskLayer 实现了 layer 接口。layer 的设计很有意思，通过层级来表示存储版本，difflayer 用来临时存储状态变更，可以同时存在多个 difflayer，disklayer 作为持久化层，负责数据的最终持久化，在一个 triedb 的实例中，只能有一个 disklayer 存在：

- 从上到下：diffLayer -> diffLayer -> ... -> diskLayer
- diffLayer 只存储与父层的差异
- 每个层都有唯一的 root 和 id
- 层间通过 parent 指针连接
- diffLayer -> diskLayer: 通过 persist() 递归合并


### 2025.04.10
ethdb 是对数据库层的抽象，比如当前主流的数据库使 leveldb，ethdb 对向 triedb 提供向数据库进行 kv 读写的接口。 

1. ethdb 是底层的通用键值对存储，提供基础的存储能力
2. triedb 是基于 ethdb 的高级状态存储系统，专门用于管理 Ethereum 的状态树
3. triedb 通过 disklayer 与 ethdb 交互，将状态数据持久化到 ethdb
4. ethdb 为 triedb 提供了性能优化的基础，而 triedb 在此基础上实现了更高级的状态管理功能

ethdb 提供了四个主要的接口：
```Go
type KeyValueReader interface {
	Has(key []byte) (bool, error)
	Get(key []byte) ([]byte, error)
}

type KeyValueWriter interface {
	Put(key []byte, value []byte) error
	Delete(key []byte) error
}

type KeyValueRangeDeleter interface {
	DeleteRange(start, end []byte) error
}

type KeyValueStater interface {
	Stat() (string, error)
}
```
其他更复杂的功能也是由这几个接口来组合完成，这也是 Go 语言中常用的方式。此外，在 core/rawdb 中的实现在以太坊的协议中实现一些数据的操作，这些是逻辑层的业务，也是通过调用 ethdb 的能力来完成。


### 2025.04.11
EVM 是以太坊的核心组件，在 `core/vm` 中实现，EVM 是一个基于栈的虚拟机，负责执行智能合约代码，提供的核心功能包括：

- 提供EVM执行环境
- 实现所有EVM操作码(opcodes)
- 处理合约调用和创建
- 管理合约执行的状态和上下文
- 实现gas计算和消耗机制

EVM 的实现有三个主要的组件， EVM 结构体定义了 EVM 的总体结构及依赖，包括执行上下文，状态数据库依赖等等，EVMInterpreter 结构体定义了解释器的实现，负责执行 EVM 字节码，Contract 结构体封装合约调用的具体参数，包括调用者、合约代码、输入等等，并且在 opcodes.go 中定义了当前所有的操作码：
```Go
// EVM
type EVM struct {
	Context BlockContext
	TxContext
	StateDB StateDB
	depth int
	chainConfig *params.ChainConfig
	chainRules params.Rules
	Config Config
	interpreter *EVMInterpreter
	abort atomic.Bool
	callGasTemp uint64
	precompiles map[common.Address]PrecompiledContract
	jumpDests map[common.Hash]bitvec
}

type EVMInterpreter struct {
	evm   *EVM
	table *JumpTable

	hasher    crypto.KeccakState // Keccak256 hasher instance shared across opcodes
	hasherBuf common.Hash        // Keccak256 hasher result array shared across opcodes

	readOnly   bool   // Whether to throw on stateful modifications
	returnData []byte // Last CALL's return data for subsequent reuse
}

type Contract struct {
	caller  common.Address
	address common.Address

	jumpdests map[common.Hash]bitvec // Aggregated result of JUMPDEST analysis.
	analysis  bitvec                 // Locally cached result of JUMPDEST analysis

	Code     []byte
	CodeHash common.Hash
	Input    []byte

	IsDeployment bool
	IsSystemCall bool

	Gas   uint64
	value *uint256.Int
}
```

### 2025.04.12
在之前有说到其实以太坊的执行层可以看作是一个交易区块的状态机，那么 EVM 就可以看作是其中的状态转换函数，所以只能由交易来驱动状态的转变，在 core/state_processor.go 中，通过 ApplyTransaction 方法或者 Process 作为入口，让 EVM 开始处理交易，这两个方法是  EVM 的入口：
![image](https://github.com/user-attachments/assets/5427831b-ed01-4845-a73b-26c4238005ea)

![image](https://github.com/user-attachments/assets/b8e27ed5-b206-41eb-8878-813df5c7b1e5)

ApplyTransaction 和 Process 都可以作为交易的入口，但是适用的场景不一样：

- Process：Process 用于区块导入和验证过程，而且需要准备整个区块的执行的环境，包括 EVM 实例、区块上下文等等，处理执行交易外，还需要处理区块奖励、系统调用等等内容
- ApplyTransaction：可用于模拟交易执行、测试或交易池验证，在调用  ApplyTransaction 时，需要实例化 EVM，并且准备好交易的上下文，只会执行单个交易，不涉及其他区块级别的操作
- Process 和 ApplyTransaction 内部都会调用 ApplyTransactionWithEVM 来执行交易

在执行交易之前，需要先初始化执行交易所需要的上下文：

- 区块相关信息：
    - 区块头信息
    - 区块 hash
    - 区块号
    - gas 池，初始容量为区块的 Gas 上限
 
![image](https://github.com/user-attachments/assets/190a3c29-7e5e-49f0-8c11-fcde51bef8af)

- 验签者：用于验证签名和恢复交易发送者的地址
![image](https://github.com/user-attachments/assets/da567e1b-7778-4c8f-a2f5-231a332b8b18)


- EVM 执行环境
    - 状态数据库
    - EVM 实例

![image](https://github.com/user-attachments/assets/6bd92059-d51f-4838-9268-6c71ca09c0f7)


- 系统调用
    - 比如处理信标链根（EIP-4788）
    - 比如处理父区块哈希（EIP-2935/EIP-7709）

![image](https://github.com/user-attachments/assets/868e6522-0c4d-4c75-affb-610d5a7ab481)

- 交易上下文
    - 将交易转换为 Msg
    - 交易上下文

![image](https://github.com/user-attachments/assets/70b90ca9-32e6-47b3-a1d8-e7436c0713d4)

### 2025.04.13
在交易执行完成之后，还需要做一些后续的操作，包括，这些需要在Petrca 升级之后才会启用：
- 处理提款请求 ( requests )

![image](https://github.com/user-attachments/assets/30cf8314-0a7e-4f75-a39c-789c0ab57911)

在以太坊协议中，"系统调用"指的是以太坊协议中特定的预编译合约调用或特殊操作，这些操作由系统账户（而非普通用户）发起，用于实现特定的协议功能。

常见的几个系统调用：

- 信标链根处理 (EIP-4788)：ProcessBeaconBlockRoot 函数实现了EIP-4788，它将信标链区块根写入到一个特殊的预编译合约中。这使得以太坊执行层可以访问共识层（信标链）的信息
- 父区块哈希处理 (EIP-2935/7709)：ProcessParentBlockHash 函数实现了EIP-2935/7709，它将父区块哈希存储在历史存储合约中，使智能合约能够访问历史区块哈希
- 提款队列处理 (EIP-7002)：ProcessWithdrawalQueue 函数处理EIP-7002中定义的提款队列，用于处理从共识层到执行层的提款请求
- 合并队列处理 (EIP-7251)：ProcessConsolidationQueue 函数处理EIP-7251中定义的合并队列

系统调用和普通交易不一样：

- 它们由系统账户（ params.SystemAddress ）发起，而非普通用户
- 它们不消耗实际的gas（使用固定的大量gas限制，如30,000,000）
- 它们调用特定的预编译合约地址
- 它们在区块处理的特定阶段执行，而不是作为普通交易

退款完整流程：

- 共识层（信标链）决定哪些验证者可以提款，并生成提款列表
- 这些提款被包含在区块的提款字段中
- 执行层在处理区块时，首先应用这些提款（增加接收者账户余额）
- 然后执行区块中的交易
- 最后处理提款队列，为下一个区块准备新的提款请求

在 process 一个区块的时候，会处理这个区块的中的退款，直接在用户的余额中增加：
![image](https://github.com/user-attachments/assets/c2dee03e-de3f-417d-8337-1fd0ab4ff9ee)

![image](https://github.com/user-attachments/assets/1cc6cd45-43f6-44b7-bad5-bf882adef915)


### 2025.04.14
在准备好上下文之后，EVM 就需要开始准备执行合约的字节码了，在实例化 EVM 时，需要根据当前的以太坊协议版本来初始化字节码解释器的版本：
![image](https://github.com/user-attachments/assets/8ca969f5-794b-483b-a628-51e2428d5a27)

比如在最新的 Pectra 升级中，就新增加了 EIP7702 的字节码：
![image](https://github.com/user-attachments/assets/d62b2242-9764-4db1-8abe-a7e379a43fc7)

在调用合约时需要先加载合约：
![image](https://github.com/user-attachments/assets/bbe79e10-d90b-4d49-93c8-227c07bf4bc3)

合约加载完成之后，就会开始在解释器中执行，执行的过程在一个 for 循环中展开，合约字节码执行的流程如下：

- 获取指令
- 检查指令操作是否有效
- 检查是否超过了栈的最大深度
- 计算指令需要消耗的 gas，并计入当前的 gas 消耗量
- 执行指令

![image](https://github.com/user-attachments/assets/4651b432-e561-4671-bc6b-87655862d368)

execute 就是执行具体指令的逻辑，具体的指令都在 core/vm/instructions.go 中实现：
![image](https://github.com/user-attachments/assets/9599187c-a8d3-4410-bbb5-eb66c812c180)

![image](https://github.com/user-attachments/assets/b2cfe762-6d89-4c3d-88ed-3e01e27b7b67)


### 2025.04.15
可以将以太坊看作是一个由交易驱动的状态机，执行层负责执行交易以及维护交易执行之后的状态，共识层负责选择哪些交易来执行。EVM 则是这个状态机中的状态转换函数，函数的输入会来源于多个地方，有可能来源于共识层提供的最新区块信息，也有可能来源于 p2p 网络下载的区块。

共识层和执行层通过 Engine API 来进行通信，如果 Validator 拿到了出块权，就会通过 Engine API 让执行层产出新的区块，如果没有拿到出块权，就将同步到的最新区块让执行层验证和执行，从而维护最新的区块。

执行层本身可以分为 5 个部分：

- EVM：负责执行交易，交易执行也是修改状态数的唯一方式
- State 管理：负责状态树的维护以及对实际存储的抽象
- 交易池（Mempool）：用于用户提交的交易，暂时存储，并且会通过 P2P 网络在不同节点之间传播
- devp2p：用于发现节点、同步交易、下载区块等等功能
- RPC 服务：为用户提供访问节点的能力，比如用户向节点发送交易



<!-- Content_END -->
