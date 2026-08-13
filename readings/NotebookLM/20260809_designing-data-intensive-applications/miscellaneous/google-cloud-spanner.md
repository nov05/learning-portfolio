
* Page 390 / 673

有一些相当知名的非 Google 公司在生产环境使用 **Cloud Spanner**。比较典型的是：

* **Uber**：Uber 的 Fulfillment Platform 使用 Cloud Spanner，主要看中它的全球分布、外部一致性和高可用能力。Uber 曾公开介绍过从原有数据库架构迁移到 Spanner 的过程。([Reddit][1])
* **Palo Alto Networks**：现在使用 Spanner 构建 **Access Graph**，把用户身份、权限和活动关系放在一个大规模图结构中，同时利用 Spanner 的关系数据能力。Google 2026 年的案例中有详细介绍。([Google Cloud][2])
* **DANA**：印尼大型电子钱包和金融服务平台，用 Spanner Graph 做反洗钱（AML）分析，需要在大规模交易数据上实时寻找账户之间的复杂关系。([Google Cloud][2])
* **ShareChat / Glance**：Google Cloud 的合作伙伴资料显示，这两个印度大型社交/内容平台曾从 DynamoDB、Cosmos DB、Cassandra 等系统迁移到 Spanner。([Google Cloud][3])
* **Storj**：Google Cloud 的合作伙伴资料提到，Storj 从 CockroachDB 迁移到了 Spanner。([Google Cloud][3])

如果从**分布式系统学习价值**来说，我觉得 Uber 是很好的例子。因为它不是“某个 SaaS 公司拿 Spanner 存点普通业务数据”，而是涉及真正的大规模、全球分布式业务，对 **external consistency、事务、分区、复制、延迟和扩展**这些问题都有实际需求。

另外有个容易混淆的地方：**很多公司使用的是 Cloud Spanner，并不意味着它们在使用 Google 内部那个原始 Spanner 系统。** Google 内部 Spanner 是 Google 自己的基础设施；Cloud Spanner 是 Google 把这套技术产品化之后提供给外部客户的托管服务。

如果你是在学分布式系统，我觉得 **Uber + Spanner** 这个案例很值得看，因为它能把课本里的“为什么需要 Spanner”直接和现实系统连接起来。

[1]: https://www.reddit.com/r/googlecloud/comments/pygln0?utm_source=chatgpt.com "Uber using External Consistency database Google Cloud Spanner"
[2]: https://cloud.google.com/blog/products/databases/customers-see-real-world-success-with-multi-model-spanner?utm_source=chatgpt.com "Customers see real-world success with multi-model Spanner | Google Cloud Blog"
[3]: https://cloud.google.com/blog/products/databases/unlocking-the-power-of-spanner?utm_source=chatgpt.com "10 Google Cloud partners to unlock the power of Spanner | Google Cloud Blog"

<br><br><br>  

---   

Uber 这个案例很适合用来理解 **Spanner 到底解决什么问题**。因为 Uber 自己公开讲过这次架构迁移，而且它遇到的恰好就是分布式系统里很典型的矛盾：**规模已经大到必须分布式，但业务状态又不能接受数据之间经常不一致。** ([Uber][1])

Uber 的这个系统叫 **Fulfillment Platform**，可以粗略理解成“把一次 Uber 行程/订单真正执行起来的核心平台”。它处理的不只是乘客点击叫车，还包括司机开始行程、给司机匹配 offer、司机位置变化后重新计算可服务产品等大量状态变化。Uber 当时描述的规模是每天数十亿数据库事务、数百万并发用户、每月数十亿 trips，覆盖一万多个城市。([Uber][1])

关键在于，Uber 原来的架构实际上更接近 **NoSQL + 高可用优先**。它使用 Ringpop 一类的分布式架构，把数据分散到不同集群。这个设计很适合大规模扩展，但它选择了 CAP 里面偏向 **AP** 的路线：网络分区的时候尽量保持可用，牺牲一部分一致性。

问题一旦落到 Uber 的业务状态上，就很麻烦。

假设一次行程涉及：

```text
Trip
  ↓
Driver
  ↓
Offer
  ↓
Product / Order
```

一次业务操作可能需要同时修改多个实体。原来的系统使用 **Saga** 一类机制协调这些修改。假设：

```text
修改 Trip       ✓
修改 Driver     ✓
创建 Offer      ✗
```

那么系统就需要做 compensating action，把前面已经成功的操作回滚或者补偿。Uber 自己总结得很直接：这种模式让开发人员必须考虑大量失败和补偿路径，而且某些 Saga 失败以后可能留下不一致状态，需要人工干预。([Uber][1])

这就是 Uber 为什么开始重新考虑数据库。

他们评估过 **CockroachDB、FoundationDB、sharded MySQL 和 Cloud Spanner**。最终 Spanner 满足了他们要求的事务一致性、水平扩展和较低运维负担。([Uber][1])

这里 Spanner 给 Uber 的最大变化其实不是“数据库变快了”，而是：

> **把一部分原来由 Uber 应用层自己负责的分布式一致性问题，交给数据库解决。**

原来应用大概需要想：

```text
写 A
 ↓
写 B
 ↓
写 C
 ↓
如果 C 失败怎么办？
 ↓
补偿 A
 ↓
补偿 B
 ↓
如果补偿又失败怎么办？
```

换成 Spanner 后，可以把相关操作放进一个数据库事务：

```text
BEGIN

UPDATE Trip ...
UPDATE Driver ...
INSERT Offer ...

COMMIT
```

Spanner 提供跨行、跨表事务，而且 Uber 使用的是它的 **external consistency**。也就是说，即使事务实际上在多个服务器、多个数据中心上执行，系统仍然让事务的效果表现得像按照某个严格的全序依次执行。([Uber][1])

这就是 Spanner 最有意思的地方：**它不是简单地把 MySQL 做成分布式版本，而是试图让“全球分布式”和“传统关系数据库事务”同时存在。**

不过 Uber 并没有因此把所有东西都交给 Spanner。这里反而非常值得注意。Uber 当时是一个 **hybrid cloud** 环境，很多 Fulfillment 应用还运行在自己的数据中心，Spanner 在 GCP。于是：

```text
Uber On-Prem
     │
     │ network
     ↓
Cloud Spanner
     │
 ┌───┴────┐
Region A  Region B
```

这带来了一个很现实的问题：**Spanner 的强一致性解决了数据库一致性，却没有消除物理网络延迟。**

Uber 甚至发现一个事务里面如果有很多次 Spanner read/write，而请求又来自自己的 on-prem 数据中心，那么每一次 RPC 都会产生网络延迟。因此他们自己做了 **transaction coalescing**，把事务里的多个操作尽量合并到一次 RPC 中。([Uber][1])

还有一个很漂亮的地方是他们对 **strong read 和 stale read** 的处理。

多 region Spanner 里面，如果一定要求最新数据，就需要承担跨复制状态同步带来的成本；有些读取却根本不需要最新状态。于是 Uber 会把：

```text
Strong read
    ↓
Leader region

Stale read
    ↓
Replica region
```

这样把不同类型的读取分开，让 replica 也得到充分利用。([Uber][1])

甚至他们还做了 **on-prem cache**。因为 Fulfillment 的 workload 很偏 read-heavy，把所有读取都送到 GCP Spanner，既增加 latency，也增加 networking cost。所以他们在自己的数据中心做缓存，同时利用 Spanner 的 snapshot/时间语义来维持所需要的一致性边界。([Uber][1])

这里就能看到一个很重要的结论：

**Spanner 并没有让分布式系统的问题消失，它改变的是问题所在的层次。**

原来 Uber 需要大量考虑：

```text
数据分片
一致性
事务失败
Saga
补偿
集群管理
故障恢复
```

采用 Spanner 后，很多事情变成：

```text
数据库负责：
    一致性
    复制
    分布式事务
    故障恢复
    集群管理

Uber 负责：
    schema
    key design
    workload
    network
    cache
    cost
```

而这也是 Uber 选择 Spanner 最核心的价值。它不是说 **“Spanner 比 NoSQL 更高级”**，而是 Uber 的业务到了一个阶段以后，**“为了追求极致可用性而让应用自己处理一致性”的工程成本已经太高了**。于是他们愿意把一部分可用性/延迟上的成本交给 Spanner，换取更强的事务语义和更简单的应用逻辑。Uber 自己的文章也明确说，他们重新设计 Fulfillment 时，把 consistency、resilience 和 availability 一起作为核心要求。([Uber][1])

所以如果把这个案例压缩成一句话，我会这样理解：

> **Uber 原来是在用分布式应用逻辑弥补数据库一致性的不足；用了 Spanner 以后，则让数据库承担更多分布式一致性责任，Uber 自己专注于业务逻辑和大规模网络/缓存优化。**

这也是为什么 Spanner 在分布式系统课程里很值得学：它实际上把 **CAP、分布式事务、复制、时间、故障处理、SQL、水平扩展** 这些原本分散的概念，全部集中到了一个真实生产系统里。

[1]: https://www.uber.com/us/en/blog/building-ubers-fulfillment-platform/?utm_source=chatgpt.com "Building Uber’s Fulfillment Platform for Planet-Scale using Google Cloud Spanner"

