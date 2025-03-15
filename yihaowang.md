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

<!-- Content_END -->
