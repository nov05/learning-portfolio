# 🟢 **Google Cloud Spanner quorum realization**   

ChatGPT:

现实中的大规模分布式系统不会让整个系统的所有节点一起做 quorum，而是先把数据切成大量独立的 **shard / split / Paxos group**，每个 group 只维护自己的少量副本。Google Spanner 就是一个很直接的现实例子：Spanner 会把数据库拆成很多 **split**，每个 split 是一段连续的 key range；每个 split 再复制到多个 failure domain，由 Paxos 管理这一组副本。([Google Cloud Documentation][1])

例如 Google 公开资料里，一个普通的 **regional Spanner** 配置有 **3 个 read-write replicas**，分别位于不同的 zone；一次写入需要 **2/3** 的 voting replicas 同意，也就是 quorum = 2。([Google Cloud Documentation][2])

```text
                    Spanner Database
                           │
             ┌─────────────┼─────────────┐
             ↓             ↓             ↓
          Split A        Split B        Split C
             │             │             │
          Paxos          Paxos          Paxos
           Group          Group          Group
             │
       ┌─────┼─────┐
       ↓     ↓     ↓
      R1     R2     R3
       │     │     │
      Zone1 Zone2 Zone3

             Write
               ↓
          R1 + R2  ✓
          quorum = 2/3
```

所以某个用户的数据如果落在 **Split A**，一次写入只需要在 Split A 的这 3 个副本之间完成 quorum，并不会让整个 Spanner 集群参与。

Google 的大规模部署还会把副本扩展到多个 region。Google 公开的资料中，常见的 multi-region 配置至少有 **5 个 replicas**，分布在 **3 个或更多 regions**；Google 的工程论文也明确说，Spanner 常见部署使用 **5 replicas**，这样最多可以失去 **2 个 datacenters** 仍然形成 quorum。([Google Research][3])

```text
                         Spanner
                            │
              ┌─────────────┴─────────────┐
              │                           │
          Region A                    Region B
        ┌──────┬──────┐             ┌──────┬──────┐
        ↓      ↓      ↓             ↓      ↓
       R1     R2                       R3     R4
        │      │                       │      │
      Zone   Zone                    Zone   Zone
              │
              └──────────────┐
                             ↓
                         Region C
                            R5
                         Witness

                 5 voting replicas
                      ↓
                  quorum = 3
```

这里的 **R5 witness** 不保存完整数据，但可以参与投票；Google 当前的 multi-region 配置使用 read-write、read-only 和 witness replica 来分别承担写入、读取和 quorum 投票等职责。([Google Cloud Documentation][1])

更重要的是，**5 个副本只是一个数据 group 的副本，不是整个 Google Docs 或整个 Google 数据库只有 5 个节点**。系统可以有海量 split：

```text
Global Data
│
├── Split 1
│    └── 5 replicas → quorum 3
│
├── Split 2
│    └── 5 replicas → quorum 3
│
├── Split 3
│    └── 5 replicas → quorum 3
│
├── Split 4
│    └── 5 replicas → quorum 3
│
├── ...
└── Split N
     └── 5 replicas → quorum 3
```

这些 split 再分布到大量机器和 region 上。Spanner 还会根据负载动态拆分、移动数据；Google 的设计本身就是 **shared-nothing + 大量独立的数据分片**。([Google Cloud Documentation][4])

所以前面“quorum 很昂贵”的直觉是对的：**真正昂贵的是一个写操作需要跨副本通信**，尤其副本分布在不同 region 时会增加网络延迟。Google 也明确指出，多 region 配置的 quorum 跨地域通信会增加 write latency；因此系统把 quorum 限制在每个数据 group 的少数 voting replicas 中，同时增加 read-only replicas 来扩展读取能力，而不扩大 write quorum。([Google Cloud Documentation][1])

这就是大规模系统的基本形态：

```text
                         Global System
                              │
              ┌───────────────┼───────────────┐
              ↓               ↓               ↓
             US            Europe            Asia
              │               │               │
          many splits     many splits     many splits
              │
        ┌─────┼─────┐
        ↓     ↓     ↓
      Group  Group  Group
        │
     3 / 5 replicas
        │
     quorum = 2 / 3
          or 3 / 5
```

**不是“Google 有 100 万台机器，所以 quorum 是 50 万台”；而是“Google 有大量数据 group，每个 group 自己有 3～5 个左右的 voting replicas，quorum 只发生在这个很小的范围内”。**

[1]: https://docs.cloud.google.com/spanner/docs/replication?utm_source=chatgpt.com "Replication  |  Spanner  |  Google Cloud Documentation"
[2]: https://docs.cloud.google.com/spanner/docs/instance-configurations?authuser=7&utm_source=chatgpt.com "Regional, dual-region, and multi-region configurations  |  Spanner  |  Google Cloud Documentation"
[3]: https://research.google.com/pubs/archive/44686.pdf?utm_source=chatgpt.com "High-Availability at Massive Scale: Building"
[4]: https://docs.cloud.google.com/spanner/docs/whitepapers/life-of-reads-and-writes?authuser=2&utm_source=chatgpt.com "Life of Spanner Reads & Writes  |  Google Cloud Documentation"
