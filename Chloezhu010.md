---
timezone: UTC+1
---

> 请在上边的 timezone 添加你的当地时区(UTC)，这会有助于你的打卡状态的自动化更新，如果没有添加，默认为北京时间 UTC+8 时区


# 你的名字

1. 自我介绍
    - Chloe，ETHPanda Core team，prev EIP Fun project lead
    - 去年参加了第一期 EPF study group，对以太坊底层协议研发开始上瘾，去年10周 study group 的笔记也可以作为参考：https://hackmd.io/@chloezhux/epfsg_notes ， 目前我对 protocol network/ light client 比较关注和感兴趣
    - 底层协议的信息量巨大，前沿领域也在不断发展，需要一遍遍不断学习，so here I am~
2. 你认为你会完成本次残酷学习吗？
    - 一定
3. 你的联系方式（推荐 Telegram）
    - TG: https://t.me/chloe_zhu
    - Twitter：https://x.com/Chloe_zhuX

## Notes

<!-- Content_START -->

### 2025.03.12

本周参与 Portal Summit，主要针对轻客户端及与执行层客户端 integrate 的讨论，笔记整理中
- 讨论主题包括
   - Portal Network 深入介绍
   - 执行层客户端集成 Portal 进展
   - Portal State Network 研发进展介绍
   - Discv5 协议改进讨论
   - EIP 4444 执行层客户端实现进展
   - EIP 7745 深入介绍
- 参考链接
    - Portal network deep dive by Piper：https://docs.google.com/presentation/d/16lscCmWZfbxDXgdubV4alRjqnEGczFc2KAsPXgbvqCs/edit#slide=id.gf912b6efc8_0_95

### 2025.03.13
- Update Portal Summit discussion notes: https://hackmd.io/DWsDCFooT-u7skmgAb-uSA?view

### 2025.03.14
- Finished Portal Summit discussion notes: https://hackmd.io/DWsDCFooT-u7skmgAb-uSA?view

### 2025.03.15
#### Ben Edgingon's talk on Gasper
- Video link: https://www.youtube.com/watch?v=cOivWPEBEMo&t=346s
- Slides link: https://docs.google.com/presentation/d/1mSn8JUfY88HvcCauLBkKRuy3f6YFlV9VcJptav0Ef24/edit?usp=sharing
<details>
<summary>Gasper Notes</summary>
    
    - Recap 
        - The problem
            - Ethereum prioritize liveness over safety, which means it can be forkful
        - Solution: the fork choice rule
            - Converge the block tree to blockchain
        - Historical issues with Ethereum PoS fork choice: 11:03
            - Recent holesky pectra upgrade struggle
        - Specs
            - class AttestationData: https://github.com/ethereum/annotated-spec/blob/master/phase0/beacon-chain.md#attestationdata
                - LMD GHOST vote: for the block
                    - depend on attestations received via gossip
                    - need real-time decision
                    - https://github.com/ethereum/annotated-spec/blob/master/phase0/fork-choice.md
                - FFG vote: for the 2 checkpoints
                    - depend on attestion in blocks
                    - part of block and epoch processing
                    - https://github.com/ethereum/annotated-spec/blob/master/phase0/beacon-chain.md
            - class Store: https://github.com/ethereum/annotated-spec/blob/master/phase0/fork-choice.md
                - each node's view of the network
        - Events
          - LMD GHOST fork choice is event driven
          - handlers
              - on_tick()
              - on_block()
              - on_attestation()
              - on_attester_slashing()
          - always ready to output a best head block via a call to get_head()
    - LMD GHOST
        - latest message driven
            - rely only on attestation
            - validators attest to what they believe to be the best head in the current slot
            - only the most recent attestation from each validtor counts
        - GHOST
            - greedy heaviest-observed sub-tree algo
        - Key points
            - timescale: 12s slot-based
            - goal:
                - used by the block proposer to decide on which branch to build its block
                - used by attesters to choose which branch to attest to
            - heuristic: which branch is least likely to be orphaned in future?
            - based on
                - weighting the votes received in attestations
                - a max of only 3.125% of the votes are fresh
            - properties
                - liveness: always output a viable head block on which to build
                - safety: no useful guarantees
        - Ignore undesirable blocks
            - most in on_block()
        - Proposer boost
            - balancing attack in 2020
            - solution: proposer boost
                - extra weight in get_weight() within the 4s at the start
                - get_proposer_score()
                - can also fork out late blocks
        - Slashing mechanism
    - Casper FFG
        - timescale: epoch-based 32 slots (6.4 min)
        - goal: confer finality on the chain
        - heuristic: 2-phase commit based on agreement among validators having at least 2/3 of the state
        - based on: weight the source adn target votes received in attestation contained in blocks
        - properties
            - plausible liveness: cannot get into a stuck state
            - accountable safety: finalize conflicting checkpoints comes at enormous cost
        - checkpoints
            - rely on seeing votes from the whole validator set
            - accounting done at each epoch end transition
            - checkpoint: the block at the first slot of the epoch
        - 2-phase commit: justification
        - Casper commandment: no double vote, no surround vote
        - Slashing in Casper FFG
            - deliver economic finality
    - Gasper: the combination
        - Apply casper ffg's fork choice: follow the chain containing the justified checkpoint of the greatest height
        - Issues
            - block tree filtering: resolve potential finalisation deadlock issue
            - unrealised justification and finalization
    - Single slot finality
    
</details>

### 2025.03.16
#### Pavel's talk on EVM
- Video: https://www.youtube.com/watch?v=gYnx_YQS8cM&t=907s
- Slides: https://docs.google.com/presentation/d/1_6tKfzWexxCe9og0c-_Qv0BfgSfqdlusXAgkStq3oY8/edit?usp=sharing
<details>
<summary>EVM Notes</summary>

- Ethereum state transition
    - Tx and State
    - State and Accounts
        - State = collection of accounts
        - address => account
        - Account
            - balance (eth account): 256-bit nbr
            - nounce: 64-bit nbr
            - code: bytes
            - storage (key-value): 32-byte, 32-byte
        - Commitments
    - Accounts duality
        - EOA (people account): balance, nonce
        - Contracts (passive code): balance, code, storage, nonce
- What's VM
    - system VM vs process VM
        - system VM: not relevant
        - process/ app VM: managed runtime env
            - eg. JVM, .NET, webassembly
        - classic programming lang
            - lang, compiler, different hw/ OS architecture
        - managed programming lang
            - add VM btw compiler and different hw/ OS architecture
    - stack-based vs register-based VM
        - stack-based
            - infinite stack
            - short instructions: bytecode
            - eg. JVM, .NET, wasm, EVM
        - register-based
            - infinite registers
            - longer instruction
            - eg. Dalvik VM, Lua VM
- EVM
    - Features
        - bytecode
        - stack-based
        - big stack items
        - no validation
        - many memories
        - exotic instructions
    - 256-bit values
    - EVM interpreter steps
        - fetch next instruction (if exists)
        - stack underflow
        - stack overflow
        - gas cost calculation + out-of-gas check
        - do actual work
    - EVM instructions overview
        - [evm.codes](https://www.evm.codes/)
- EVM unique features
  - Internal calls
        - What's internal call: an internal call refers to a function call made from one smart contract function to another within the same execution context. These calls do not create new transactions but occur within the same transaction that triggered the original contract execution
        - Instructions that invoke other contracts
            - Call
            - Delegatecall
            - StaticCall
        - Args
            - address
            - gas
            - value
            - input
        - Results
            - return data
            - remaining gas
    - EVM memories
        - stack: instruction operands
        - memory: main volatile memory
            - limited by gas 
        - calldata: input data
            - read only
        - returndata: output from sub-calls
            - read only
        - storage: persistent storage
    - Gas metering: 52:03
        - execution is limited by gas units
        - on all levels:
            - internal calls
            - tx：mostly by wallets
            - block: gas limit discussion
        - instruction cost some gas: constant, complex formula
- EVM object format
- 

</details>
<!-- Content_END -->
