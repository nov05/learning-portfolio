# 🟢 **Linearizable Write**  

Page 433 / 673  
```text
Strictly speaking, ZooKeeper provides linearizable writes, but reads
may be stale, since there is no guarantee that they are served from
the current leader [20]. etcd since version 3 provides linearizable
reads by default.
```

Linearizable write 指的是一次 write 成功后，系统已经对这个 write 的顺序和结果形成一致认知，后续操作不能再表现得像这个 write 没有发生。Stale read 则是 read 可能从尚未同步最新状态的 replica 读取，因此得到旧值。Stale read 本身不一定是问题，关键在于这个 read 的结果是否被用于需要当前状态才能正确做出的决定。对于只是展示信息、允许短暂不一致的场景，stale read 通常可以接受；涉及权限、锁、资源所有权、唯一性约束等 safety-sensitive 决策时，则需要更强的 consistency，例如 linearizable read。

```text id="38164"
Initial state:
x = 0

Client A:
write x = 1
→ success

Client B:
read x
→ must see x = 1

核心：
write 成功后，后续 linearizable read 不能再读到旧值 x = 0。
```

以 Google Docs 为例，用户编辑文档时，另一个用户的界面短暂显示旧内容通常没有问题，因为这是 presentation 层面的短暂延迟，之后可以同步到最新状态；类似地，一些实时 feed、状态展示也可以接受 stale read。但如果把 stale read 用于判断“这个用户现在是否有编辑权限”，或者判断“哪个节点现在持有这个 distributed lock”，就可能出问题：旧的权限状态可能导致未授权操作，旧的 lock 状态可能导致两个节点同时认为自己拥有资源。核心区别不是“数据能不能 stale”，而是 **stale data 是否被用于需要最新状态才能保证正确性的 decision**。
