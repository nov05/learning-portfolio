# 🟢 **Tips to learn distributed systems**

很多经典分布式系统概念，是在处理 **timeout、failure detector、retry、duplicate request、idempotency、consensus** 等 failure。学习分布式系统时，先建立 **failure taxonomy**，再看一个算法或机制在处理哪一种 failure。这样 **Raft、Paxos、2PC、replication、consistency、distributed transaction** 等概念会变得非常有逻辑和联系。

```text
问题                         对应机制
────────────────────────────────────────
消息可能丢失                 Retry
Retry 可能导致重复执行      Idempotency
节点可能突然死掉            Replication
节点恢复后状态可能过期      Log / WAL
网络可能断开                Partition handling
消息可能乱序                Sequence number
消息可能重复                Deduplication
多个节点同时修改            Concurrency control
节点存活状态无法确定        Failure detector
数据可能暂时不一致          Consistency model
多个节点需要达成一致        Consensus
```

```text
Node failures
├── crash
├── restart
├── slow / hung
└── Byzantine / corrupted

Network failures
├── packet loss
├── delay
├── duplication
├── reordering
├── partition
└── connection failure

Message failures
├── lost
├── duplicated
├── delayed
├── reordered
└── delivered after timeout

Timing
├── clock skew
├── clock drift
└── timeout 误判

Storage
├── write failure
├── stale read
├── corruption
└── data loss

Concurrency
├── race condition
├── concurrent update
├── deadlock
└── inconsistent state

Recovery
├── crash recovery
├── retry
├── replay
├── duplicate execution
└── failover
```
