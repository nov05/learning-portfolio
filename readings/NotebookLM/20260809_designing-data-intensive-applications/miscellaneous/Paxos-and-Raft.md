

Paxos = Majority Agreement Protocol   
       （多数派达成一致协议） 

Raft  = Leader-Based Log Replication   
       （基于 Leader 的日志复制协议）   

Paxos = “投票达成共识”  
Raft = “选老大 + 老大同步日志”   

```text
                    分布式共识
                       │
             ┌─────────┴─────────┐
             ↓                   ↓
           Paxos                Raft
             │                   │
       理论经典/更抽象        工程化/更直观
             │                   │
        Multi-Paxos          Leader + Log
```

ChatGPT:

Raft → Paxos → Multi-Paxos → Zab → ZooKeeper/etcd  

分布式系统里的这条线，本质上是在讲**多个节点如何对一系列操作达成一致，并在部分节点故障时仍然保持一致性**。Raft 采用比较直观的 Leader 模型：集群先通过选举产生一个 Leader，客户端请求交给 Leader，Leader 把请求写入自己的日志，再复制给其他节点，只要多数节点确认，这条日志就可以提交，所有节点最终按照相同顺序执行这些日志；Paxos 解决的是更一般的分布式共识问题，它通过 Proposer、Acceptor 等角色让节点在网络延迟、消息丢失甚至部分节点宕机的情况下，对一个值达成一致，只不过它的表达方式比 Raft 抽象得多；Multi-Paxos 则是在 Paxos 的基础上解决“连续做很多次共识”的问题，通过选出一个相对稳定的 Leader，让它连续提出多个值，从而把一次次独立的共识组织成一条有序日志，这实际上就和 Raft 的 Leader + Log Replication 非常接近；Zab（ZooKeeper Atomic Broadcast）则是另一套用于实现复制状态机的协议，它通过 Leader 对事务进行排序并广播给其他节点，保证所有正常节点以相同顺序执行事务；在此基础上，ZooKeeper 把这种一致性机制封装成一个分布式协调服务，提供配置管理、命名、选主、分布式锁等能力，而 etcd 则采用 Raft，把 Raft 共识和复制日志封装成一个可靠的分布式 KV 存储，Kubernetes 等系统可以直接把它当作一致性的元数据存储。因此整条线可以看成：**Raft 是 Leader 驱动的日志复制，Paxos 是更抽象的共识，Multi-Paxos 把 Paxos 扩展成持续的日志复制，Zab 是 ZooKeeper 使用的原子广播协议，而 ZooKeeper 和 etcd 则是把这些共识机制真正变成可供其他分布式系统使用的基础设施。**  