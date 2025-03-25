---
timezone: UTC+8
---


# Yihao

1. 自我介绍
  - full stack developer
  - 期待能參與 web3 開發學習和分享
2. 你认为你会完成本次残酷学习吗？
  - 會
  - 
3. 你的联系方式（推荐 Telegram）
  - zero5011@gmail.com

## Notes

<!-- Content_START -->

### 2025.03.10

![image](https://github.com/user-attachments/assets/e6ae4706-c3b2-41f7-9f7a-8300c62ac288)

#### Consensus Layer(共識層)
- Validators(驗證者): 負責處理交易、提議和驗證區塊，確保區塊鏈的安全和完整性
- LMD-GHOST(最後投票決定): ETH2.0 採用的 Fork Choice 演算法
- Fork choice: 用來決定哪個分叉應該成為主鏈的一部分
- RANDAO: 負責隨機選出下一個 Proposer(提議者)
- Blobs: Binary Large Object 的縮寫，指的是一種專門設計的數據結構，用於存儲大量數據。在以太坊的上下文中，Blobs 是一種臨時存儲的數據塊，專門用於 Layer 2 解決方案（如 Optimistic Rollups 和 ZK-Rollups）將交易數據提交到以太坊主網。
  
#### Execution Layer(執行層)
- TXs (mempool): 交易首先進入 mempool（記憶池），等待驗證者打包進區塊
- EVM:
  - 執行 Ethereum 智能合約的核心環境
  - 解析智能合約的 opcode（操作碼） 並運行計算
  - 負責管理 Ethereum 的 Gas 消耗
- State (data)
  - 儲存所有帳戶資訊、智能合約狀態、餘額、Nonce 等
  - EVM 透過 State 來讀取與修改帳戶的狀態
- JSON-RPC API: 讓 使用者 或 Web3 應用 與 Ethereum 節點互動的 API

### 2025.03.12
共識層
- Validators(驗證者): 

#### Consensus跟Execution之間的互動
共識層與執行層透過 Engine API 進行溝通，過程如下：
1. 驗證者 選擇新的區塊，並透過 Engine API 發送給執行層
2. 執行層 解析區塊中的交易，透過 EVM 執行，更新 State 並驗證區塊完整性
3. 若區塊合法，共識層會透過 LMD-GHOST 選擇最佳鏈，並通知其他驗證者

### 2025.03.11
#### Consensus Layer(共識層)
Ethereum 的共識機制歷經了 工作量證明(PoW)到權益證明(PoS)的轉變

#### 拜占庭將軍問題(Byzantine Generals Problem, BGP)
是一個分散式系統中的容錯問題，描述了在不可靠的通訊環境下，如何讓多個節點就某個決策達成共識，即使其中部分參與者可能是惡意的。這個問題的核心在於 如何確保整個系統能夠在惡意行為的干擾下依然保持一致性（Consensus）

#### PoS(Proof of Stake, 權益證明)
以太坊在 2022 年 9 月 15 日 的 The Merge 升級後，正式由 PoW 轉為 PoS，取消了挖礦，改為由「驗證者（validator）」負責共識

PoS 的運作方式
1. 質押（Staking）
   - 想參與 PoS，需要 質押 32 ETH 成為「驗證者」
   - 這些 ETH 會被鎖定，作為參與共識的保證金
2. 隨機選擇出塊者
   - 以太坊 PoS 採用 Casper FFG + LMD GHOST 機制來選擇出塊者
   - Casper FFG（Finality Gadget）：確保交易的「最終性」（Finality）
   - LMD GHOST（Greedy Heaviest Observed Subtree）：決定哪條鏈是最重（最多驗證者支持），用來防止分叉攻擊
   - 在每個「slot」(12 秒)中，系統會隨機選擇1名驗證者 負責提議區塊
   - 其他驗證者會對該區塊進行投票，決定是否接受
3. 區塊驗證與共識
   - 選出的驗證者會打包交易，創建新的區塊，並廣播至網絡
   - 其他驗證者會檢查該區塊是否正確，然後投票
     - 若區塊通過 2/3 的驗證者投票，則區塊被確認，並成為最終區塊（Finalized）
     - 若驗證者惡意行為(例如提交無效區塊)，則會被懲罰，甚至沒收部分質押的 ETH（Slashing）
4. 區塊獎勵
   - 驗證者透過參與 PoS 來獲得獎勵，主要包括
     - 出塊獎勵(來自新鑄造的 ETH)
     - 交易手續費(但 PoS 下的手續費燃燒機制會減少發行量)
   - 若驗證者 離線或違規，則可能會被懲罰(例如損失部分 ETH)

#### 驗證者
以太坊會從這些驗證者中 隨機選擇 負責提議區塊與驗證交易

如何成為驗證者:
  1. 存入32ETH
  2. 運行驗證者節點，保持 24/7 在線
  3. 選擇運行方式
     - 自行架設節點
     - 使用 Staking 服務（如 Lido、Rocket Pool，這些允許小額質押）
     - 加入交易所的 Staking 計畫（如 Binance、Coinbase Staking）


#### Slot
- Slot（時間槽）是以太坊 PoS 中的一個時間單位，每個 Slot 持續 12 秒
- 在每個 Slot 中，一名驗證者(Validator) 會被隨機選中來 提議(propose)一個新的區塊(Block)
- 其他驗證者則負責 驗證與投票(Attestation)，確保區塊的正確性與共識
- 並非每個 Slot 都一定會產生區塊，有些 Slot 可能會「空缺(missed slot)」，導致該 Slot 沒有區塊


### 2025.03.13
Execution Layer(執行層)和Consensus Layer(共識層)共同合作來處理交易和維護區塊鏈的安全性
1. 交易的產生與廣播
   1. 用戶使用 MetaMask 或其他錢包發送一筆交易
   2. 交易會經過數字簽名(使用私鑰)，確保交易不可被篡改
   3. 交易被廣播到 Ethereum P2P 網絡，所有執行層的節點(Execution Clients)會將它加入交易池(Mempool)，等待打包
   
2. 交易的處理與執行(Execution Layer)
   1. 執行層的節點從交易池中選擇交易，並根據 Gas 費排序
   2. 節點執行EVM來執行交易
   3. EVM 計算交易的狀態變更(State Transition)
   4. 交易執行完成，產生Receipt
   
3. 區塊的提議與驗證(Consensus Layer)
   1. Slot 內的驗證者(Validator)被隨機選中為「提議者(Proposer)」來創建區塊
   2. 區塊提議者向執行層請求最新的交易狀態
      - 執行層返回「交易清單」與「狀態變更」，共識層將它封裝成區塊(Beacon Block)
   3. 提議者將區塊廣播到全網，其他驗證者開始驗證區塊的有效性
      - 這些驗證者不會重新執行交易，而是檢查區塊內的「狀態根（State Root）」是否正確。
      - 如果區塊通過驗證，大多數驗證者會投票同意這個區塊，並將其加入區塊鏈

4. 狀態的確認與最終確定性
   1. 當區塊被大多數驗證者接受後，交易正式生效，狀態變更（如帳戶餘額更新）會被寫入區塊鏈
   2. 檢查最終確定性(Finalization)
      - 以太坊 PoS 使用 Casper FFG（Finality Gadget），確保區塊不會被回滾。
      - 當一個區塊被「兩次 Checkpoint 確認」（約 12.8 分鐘），它就會被視為最終確定，無法被更改
   3. 用戶可以查詢區塊瀏覽器來確認交易成功

### 2025.03.15
#### Ethereum Testing and Security
1. Execution Layer Testing
   - EVM testing
     - State Testing: 驗證以太坊虛擬機（EVM）在執行智能合約時的狀態變化，確保狀態轉換符合預期
     - Fuzzy Diﬀerential State Testing: 透過模糊測試技術，隨機產生輸入來比較不同 EVM 實作之間的行為差異，找出潛在的漏洞或錯誤
     - Blockchain Testing: 模擬完整區塊鏈環境，測試區塊驗證、交易執行、手續費計算等機制的正確性
     - Blockchain Negative Testing: 針對異常狀況（如惡意交易、無效區塊）進行測試，確保系統能正確處理錯誤情境，避免安全風險
   - Execution APIs: 測試執行層提供的 API 是否符合規範，確保不同用戶端的 API 互通性及正確性
2. Consensus Layer Testing: 針對以太坊的 PoS（權益證明）共識機制進行測試，包括驗證者行為、區塊提議與最終確定性，確保共識協議的穩定性與安全性
3. Cross-Layer Testing
   - Execution + Consensus: 測試兩個層級之間的互動與同步機制，確保執行層與共識層協同運作正常
   - Hive: 一個自動化測試框架，支援多個以太坊用戶端的互通性測試，確保不同實作之間的一致性
   - Devnets: 用於測試新功能、硬分叉升級或協議變更的小型私有測試網，讓開發者驗證新版本的行為
   - Shadow-forks: 從主網或測試網擷取部分狀態並在獨立環境中進行測試，以驗證新功能的兼容性和穩定性
   - Testnets: 公共測試網，如 Goerli、Sepolia，用於測試智能合約、協議升級及應用程式部署
4. Ethereum Security: 透過各種測試技術發現與修復安全漏洞，包括智能合約審計、共識協議安全性分析、抗攻擊測試（如 Sybil 攻擊、防重放攻擊等），確保以太坊網路的安全性與穩定性

### 2025.03.16
#### Roadmap
- Merge: Better Proof of Stake: 將以太坊從 PoW（工作量證明）轉換為 PoS（權益證明），提高能源效率與安全性
  - 2022 年 9 月 The Merge 正式完成，主網與信標鏈（Beacon Chain）合併，淘汰 PoW
  - 以太坊的區塊由驗證者（Validators）產生，而非礦工（Miners）
  - 減少 99.95% 能耗，使以太坊變得更環保
    
- Surge: More data (availability) for rollups: 透過分片（Sharding）技術，降低 Rollup 成本，提升以太坊的交易吞吐量
  - 目前以太坊主鏈的交易處理能力有限，每秒僅 15-30 筆（TPS）
  - Danksharding 和 Proto-Danksharding（EIP-4844） 可透過 Blob data 提供更便宜的 Rollup 交易數據可用性
    
- Scourge: Less MEV downsides: 降低最大可提取價值（MEV，Maximal Extractable Value）帶來的負面影響，減少以太坊生態內的經濟不公平性
  - PBS（Proposer-Builder Separation） 設計可將區塊提議者與構建者分離，降低壟斷與操控風險
- Verge: Easier verification
  - 主要透過 Verkle Trees 來減少驗證數據存儲需求
    
- Purge: Simpler protocol: 透過更好的密碼學技術，讓驗證者可以更輕鬆地運行節點，降低區塊驗證成本
  - 目前以太坊節點需要儲存完整的區塊歷史記錄，佔據大量存儲空間
    
- Splurge: Miscellaneous goodies: 刪除過時或不必要的歷史數據，降低節點的運行成本，使網路更輕量化
  - Account Abstraction（AA）：讓用戶可以自定義錢包的簽名方式，提高安全性與可用性

### 2025.03.17
#### Ethereum Execution Layer Spec
什麼是Ethereum Consensus Specs: 是以太坊共識層的技術規範，用於定義 驗證者行為、區塊處理、狀態轉換、罰則 等規則。這些規範不是直接執行的程式碼，而是提供一個標準，讓所有以太坊客戶端（如 Prysm、Lighthouse、Teku）都能夠正確實作共識機


#### Ethereum Consensus Specifications
以太坊共識層規範（Ethereum Consensus Layer Specifications，簡稱「共識規範」）是 以太坊基金會 所維護的官方規範，負責定義以太坊 共識層（Consensus Layer, CL） 的運作方式。這部分與 執行層（Execution Layer, EL） 相輔相成，共同構成現代以太坊協議的基礎。

共識層的主要功能：
- 確保以太坊網絡在 Proof of Stake（PoS） 下正常運作
- 負責 驗證區塊、同步區塊、管理驗證者（Validators）
- 提供 最終性（Finality），確保區塊不會被輕易回滾
- 協調 驗證者獎勵和懲罰（Slashing）


### 2025.03.19
Reth 是什麼: Reth是一個用 Rust 實作的 以太坊（Ethereum）執行層（Execution Layer）客戶端，由 Paradigm 開發。它是一個高效能、模組化、開放原始碼的以太坊節點軟體，目標是提供更快、更安全、更易於維護的以太坊客戶端。

為什麼需要 Reth: 目前以太坊的執行層（Execution Layer）主要有以下幾個客戶端：

- Geth（Go Ethereum，最多節點使用）
- Nethermind（C#）
- Besu（Java）
- Erigon（Go，專注於存儲與效能）
- Reth（Rust）

Reth 的主要特點
- 高效能
  - Rust 的系統級語言特性，讓 Reth 具有更好的記憶體管理與執行效能。
  - 針對交易處理與區塊同步進行優化，比 Geth 更快。
- 模組化設計
  - Reth 的核心架構是模組化的，可以靈活地與不同的以太坊基礎設施（如共識層、存儲後端）整合。
  - 易於擴展與自訂，適合開發者社群參與。
- 對開發者友好
  - 使用 Rust 編寫，提供更好的型別安全性與錯誤處理機制，降低 Bug 風險。
  - 原始碼開放，任何人都可以參與開發，提升以太坊生態系統的多樣性。
- 更快的同步速度
  - 針對區塊數據存儲與同步進行優化，相比 Geth，能更快地加入網絡並完成區塊驗證。

Reth 在以太坊生態中的角色: Reth 主要負責 執行層（Execution Layer），處理：
- 交易執行
- 智能合約運行
- EVM 計算
- 狀態管理（State Management）

而它需要搭配 共識層（Consensus Layer） 客戶端（如 Prysm、Lighthouse、Teku、Nimbus）來運作，因為以太坊現在是 PoS（Proof of Stake）機制，執行層與共識層必須協同工作。

### 2025.03.20
#### Teku 共識客戶端
Teku 是由 ConsenSys 開發的一款 以太坊 2.0（信標鏈）共識客戶端，專門針對機構級別的需求設計，支持 驗證者管理、信標鏈運行、共識協議、與執行客戶端交互 等核心功能。Teku 以 Java 編寫，並基於 Apache 2.0 開源授權，是以太坊 2.0 主要的共識客戶端之一。

Teku 的特色
- 高可靠性：專為 機構和專業驗證者 設計，適合大型運營商。
- 安全性高：支持 遠端簽名、密鑰管理與監控工具（如 Prometheus）。
- 靈活性強：支持 單節點、多節點運行，可與多種執行客戶端（如 Geth、Erigon）搭配使用。
- 標準 API：符合 Ethereum JSON-RPC、REST API、Engine API，便於集成。

Teku 架構
1. P2P 網絡層
  - 使用 Libp2p 網絡協議與其他節點進行通信
  - 負責區塊同步、Gossip 區塊傳播、驗證者資訊交換
2. 信標鏈存儲（Beacon Chain Storage）
  - 儲存 信標鏈區塊、驗證者狀態、隨機性數據，確保歷史可追溯
  - 提供 SQLite、LevelDB 兩種存儲選項
3. 共識機制
  - Casper FFG（Finality Gadget）：確保區塊最終性
  - LMD-GHOST（Fork Choice Rule）：選擇權重最高的區塊作為主鏈
4. 驗證者管理
  - 支持 單節點多驗證者，可運行數千個驗證者
  - 內建 Slashing 監測，防止驗證者被雙重簽名處罰
5. Engine API（與執行客戶端交互）
  - 負責與 執行客戶端（如 Geth） 進行通信，獲取交易數據、提交區塊
  - 透過 JSON-RPC 和 REST API 提供外部控制接口

Teku 的工作流程
1. 啟動並同步信標鏈
  - 連接到 P2P 網絡，從其他節點獲取區塊數據
  - 檢查區塊的有效性，更新本地信標鏈存儲
2. 管理驗證者
  - 檢查驗證者的押注（Stake）狀態，確保符合 PoS 要求
  - 根據 LMD-GHOST 規則選擇最佳區塊進行投票
3. 與執行客戶端協作
  - 透過 Engine API 與執行客戶端同步交易數據
  - 驗證新交易，將合法的交易打包成區塊
4. 提交區塊與投票
  - 若節點被選為 提議者（Proposer），則負責提交區塊
  - 其他驗證者則對區塊進行 Attestation（投票），確保共識達成
5. 罰則與獎勵
  - 若驗證者離線、雙重簽名，會被 Slashing（罰款）
  - 若驗證者表現良好，則獲得 ETH 獎勵


### 2025.03.21
#### Ethereum DevOps

**1. 什麼是 Ethereum DevOps？**

Ethereum DevOps 團隊的核心職責是測試、部署與維護 Ethereum 網路的升級，確保新功能的推出不會影響區塊鏈的安全性與穩定性
在 Ethereum 開發與測試中，DevOps 的挑戰包括：
- 需要測試 超過 20 種不同的 Execution Clients（ELs）和 Consensus Clients（CLs）組合，確保它們相容
- 偵錯與溝通的困難度高，不同客戶端的問題可能影響整個網絡
- 可靠的測試方式尚在演進，過去僅針對單一客戶端進行測試，現在需要全面測試
- 升級影響層面廣，所有未來的升級都會繼承過去的複雜性，因此 DevOps 必須建構可重複使用的測試流程。


**2. DevOps 測試環境：Devnets（開發測試網）**

什麼是 Devnet？
- Devnet 是一個 Ethereum 的測試環境，用來模擬主網的行為，讓開發者能夠測試硬分叉或升級前的變更
- 內容包括
  - Execution Layer（EL）、Consensus Layer（CL） 和 驗證者（Validators）
  - 自訂測試環境，支援開發者測試新功能，不影響主網
  - 完全可控的驗證者集，允許測試極端情境（edge cases）
- Devnet 是公開的，社群開發者可以一起測試與回報錯誤

### 2025.03.23
#### Precompile
Precompile（預編譯合約）是一組特殊的智能合約，這些合約並不是用 Solidity 或 Vyper 編寫的，而是直接在以太坊客戶端（如 Geth、Nethermind、Besu）內部實作，並且擁有固定的合約地址（0x01 到 0x0A，EIP 擴展後可更多）

這些預編譯合約主要用於執行高計算成本的加密操作，如哈希、簽名驗證等，它們比一般的 Solidity 智能合約執行得更快且成本更低，因為它們直接運行在 EVM（Ethereum Virtual Machine）的內部，而不是透過 Solidity 編譯後的 EVM 字節碼（Bytecode）

1. Precompile 的特點
- 固定地址：這些合約的地址在所有以太坊網絡（如主網、測試網）都是一致的
- 原生 EVM 支援：直接由 EVM 內部執行，而不是由 Solidity 編寫的普通合約
- 更高效能：相較於 Solidity 編寫的等效合約，Precompile 執行速度更快，消耗更少 Gas
- 降低 Gas 成本：許多 Precompile 內部的操作如果用 Solidity 來寫，Gas 成本會高得多

2. Precompile 的優勢與限制
   - 優勢
     - 比 Solidity 智能合約更快：因為它們直接由 EVM 運行，而不是透過 Solidity 編譯後執行
     - 降低 Gas 成本：如果這些運算用 Solidity 來寫，成本會高很多，例如 ecrecover 在 Solidity 內部實作會非常昂貴
     - 標準化：所有以太坊客戶端都支持這些合約，不會因客戶端不同而影響結果。
   - 限制
     - 只能執行特定功能：Precompile 不能像 Solidity 那樣自由開發，僅限於內建的函數
     - 難以擴展：新增 Precompile 需要 Ethereum 改進提案（EIP）並獲得社群共識，例如 blake2f（EIP-152）
     - 與 Layer 2 兼容性不佳：某些 L2（如 Optimistic Rollup）可能不支持所有 Precompile，影響智能合約兼容性。

### 2025.03.24
#### Sharding and DAS

什麼是數據可用性抽樣(DAS): DAS 是一種輕量級驗證技術，允許節點在不下載整個區塊數據的情況下，確保區塊中的數據可用且完整

為什麼需要 DAS:
- 區塊鏈數據量過大：以太坊區塊鏈上的數據隨著時間增長，完整節點需要存儲大量數據，造成硬體負擔
- 輕量級節點無法儲存完整數據：傳統區塊鏈驗證需要下載整個區塊，這對資源有限的裝置（如手機、瀏覽器節點）不現實
- DAS 允許節點部分下載數據，但仍能驗證其完整性，大幅降低儲存需求並確保安全性

DAS 如何運作
- 區塊數據透過誤差校正碼（Erasure Coding）進行編碼，將數據轉換成冗餘區塊，確保即使部分數據遺失，也能透過數學方式恢復
- 節點隨機下載部分數據，並驗證其一致性。如果抽樣的數據是有效的，就可以推測整個區塊的數據是可用的
- 驗證者（Validators）透過隨機抽樣機制確保安全性，即使攻擊者試圖隱藏某些區塊數據，只要大多數節點仍然可存取數據，區塊仍可被驗證


什麼是 Danksharding: Danksharding 是以太坊 2.0 計畫的一部分，旨在透過「單一拍賣機制」來簡化分片設計，使以太坊的可擴展性更強
- 與傳統 Sharding 的不同點: 傳統分片（Sharding）：每個分片區塊鏈都有自己的「提議者（Proposer）」，需要跨分片通訊來確保數據一致
- Danksharding：整個網路只有一個提議者（即 Dankshard proposer），負責所有分片的數據提交，減少協調成本，提升效率

Danksharding + DAS 如何提升擴展性: 
- Danksharding 提供大規模的數據可用性（Data Availability）
- DAS 允許輕量級節點高效驗證數據
- 兩者結合可讓以太坊網絡支援更多 Rollups（如 Optimistic Rollup, ZK Rollup），提升交易吞吐量並降低 Gas 費用

### 2025.03.25
#### Verkle Trees
Verkle Tree（Verkle 樹）是一種改進版的 Merkle Patricia Tree（MPT），主要用於 區塊鏈數據存儲與驗證，目前是以太坊等區塊鏈研究的熱門技術之一。它的核心目標是減少數據存儲需求，並提升驗證效率，最終實現「無狀態（stateless）驗證」

為什麼需要 Verkle Trees: 目前的以太坊使用 Merkle Patricia Tree（MPT） 來管理帳本狀態（例如帳戶餘額、合約存儲等）。然而，MPT 有幾個主要缺點
- 數據大小太大：每次驗證時都需要傳遞大量 Merkle Proof（默克爾證明），導致節點存儲壓力大
- 存取速度慢：由於樹的深度較深，每次存取數據需要經過多層節點
- 無法支援「無狀態驗證」：目前的節點仍需存儲大量狀態數據，無法輕量運行
  
Verkle Trees 的運作方式: Verkle Tree 本質上仍是一種樹狀數據結構，但與 Merkle Tree 最大的不同是
- 使用向量承諾（Vector Commitments）
  - Merkle Tree 使用的是哈希函數來建立樹結構，而 Verkle Tree 則使用 向量承諾（Polynomial Commitments），如 Pedersen 承諾 或 KZG 承諾
  - 這讓 Verkle Tree 能夠將多個葉節點的哈希值合併成更短的證明，從而大幅減少存儲需求
- 更小的證明大小
  - 在 Merkle Tree 中，證明大小（Merkle Proof）會隨著樹的深度增加而增長，而 Verkle Tree 的證明大小可以保持固定（大約 150~200 bytes），即使樹變得非常大
  - 這對於無狀態驗證（Stateless Validation）非常重要，因為節點無需存儲完整狀態，只需驗證一個小型證明即可確認交易合法性
- 減少節點深度，提高查詢速度
  - MPT 每個內部節點最多有 16 個子節點，導致樹的深度較深，查詢時需要經過許多層節點
  - Verkle Tree 每個內部節點可以有 256 個子節點（或更多），這大幅減少了樹的深度，使得存取速度更快
 
Verkle Trees 的優勢:
- 更小的驗證數據 → 減少存儲需求，降低節點運行成本
- 更快的查詢速度 → 提升節點運行效率
- 支援無狀態驗證 → 讓節點無需存儲完整狀態，即可驗證交易
  
<!-- Content_END -->
