
ChatGPT:

ScyllaDB 的创业背景很有代表性。它源于一批长期做 Linux、KVM、虚拟化和高性能系统软件的工程师。核心人物包括 Avi Kivity 和 Dor Laor，两人曾在 Qumranet 共事。Qumranet 后来被 Red Hat 收购，他们进入 Red Hat 继续工作。2012 年，两人离开 Red Hat，创办 Cloudius Systems。这个阶段形成了两项重要资本：一套很深的 systems engineering 能力，以及一个来自 KVM、Linux、Red Hat 和开源社区的人才网络。Scylla 后来的早期团队，很大程度上就建立在这套技术网络上。

Cloudius 最初的方向其实是 OSv，一个面向云计算和虚拟机环境重新设计的操作系统。团队希望从更底层重新思考操作系统，让应用在云环境中获得更高性能。为了验证 OSv，他们开始拿真实应用做 benchmark。测试过程中出现了一个很有意思的结果：很多应用在 OSv 上性能提升明显，Cassandra 却没有。团队继续往下分析，发现 Cassandra 本身的分布式设计已经相当成熟，真正限制性能的部分更多来自实现层，包括 Java runtime、线程模型、内存管理、I/O，以及传统服务器软件对现代多核 CPU、SSD、网络设备的利用方式。

这个发现改变了 Cloudius 的产品方向。Cassandra 已经完成了市场教育，数据模型、API、生态、部署方式和客户群都已经存在。团队可以把问题集中到 implementation：**保留 Cassandra 已经被市场验证的产品形态，重新设计底层实现，让数据库充分利用现代硬件。**这形成了 Scylla 的基本产品逻辑：相同的 workload，通过更高效的实现获得更高吞吐、更低延迟，同时减少所需服务器数量。对于企业客户，这种性能优势可以直接转化成基础设施成本优势。

为了实现这个目标，团队开始使用 C++ 重写 Cassandra，并开发 Seastar。Seastar 是一个面向高性能 server application 的 C++ framework，核心设计包括异步 I/O、shared-nothing、shard-per-core、NUMA awareness 等。它把 CPU core、memory、network、storage 等资源更直接地交给应用管理。Scylla 建立在这套技术之上，从执行模型到底层 I/O 都围绕现代多核硬件重新设计。所以 Scylla 的技术来源其实可以一直往下追到 Linux 和 KVM：KVM 让团队积累虚拟化和内核级系统经验，OSv 让他们进一步研究云环境下的系统性能，Cassandra benchmark 暴露出新的问题，Seastar 提供底层解决方案，最后形成 ScyllaDB。

```text id="8c6k3u"
Linux / KVM
      ↓
虚拟化、CPU、内存、I/O
      ↓
Qumranet
      ↓
Red Hat
      ↓
Cloudius Systems
      ↓
OSv
      ↓
真实应用 benchmark
      ↓
发现 Cassandra 性能瓶颈
      ↓
Seastar
      ↓
重新实现 Cassandra
      ↓
ScyllaDB
```

人员组织同样值得关注。Scylla 早期很难按照普通互联网公司的招聘方式组建，因为产品需要大量非常底层的能力：kernel、CPU architecture、NUMA、networking、storage、C++、异步编程、distributed systems。Avi Kivity 本身就是 KVM 的核心创造者之一，创始团队又来自 Qumranet、Red Hat 和 Linux 社区，所以他们能够直接从已有技术网络中寻找合适的人。一个 KVM 工程师认识 Linux kernel 工程师，后者又认识 storage、networking 或 compiler 方向的人，这种社区关系会不断扩散。早期公司的组织资本因此很大一部分来自创始人的技术声誉和开源社区关系。

这种背景也解释了为什么 Scylla 的创业路径和普通 SaaS startup 差异很大。公司早期最稀缺的资源不是产品经理、销售团队或者增长团队，而是能够把数据库做到极致的系统工程师。产品方向确定以后，核心技术团队先建立起来，再逐步补充产品、销售、市场、企业服务等职能。对于这种基础设施公司，早期几十名工程师的技术密度可能比团队规模本身重要得多。

资本进入以后，投资逻辑开始从技术问题转向产业经济学。Scylla 面对的市场已经存在：Cassandra、Dynamo-style distributed database、NoSQL 都已经有大量用户。它需要证明的是性能和成本优势。数据中心 workload 不断增长，CPU、SSD、网络和内存硬件快速发展，软件对硬件资源的利用效率越来越重要。如果数据库能够让一台机器承担原来几台机器的 workload，客户得到的不只是 benchmark 上更高的 QPS，还包括服务器采购、电力、机房空间、网络以及运维成本的下降。

因此 Scylla 后来的投资人里出现了 Western Digital、Samsung、Qualcomm 等产业资本，也有传统 VC。产业资本尤其容易理解这套逻辑：硬件性能持续提升，软件基础设施需要跟着重新设计，能够把硬件性能释放出来的软件具备很高的产业价值。Scylla 早期融资的用途也逐渐从纯技术研发扩展到 enterprise 产品、销售和市场。

开源则解决了数据库创业最重要的一个信任问题。数据库承载核心数据，客户不会轻易把生产 workload 迁移到一个完全陌生的产品。Scylla 采用 open-source 模式，让工程师可以自己下载、测试、benchmark，再逐渐进入生产环境。Cassandra compatibility 又进一步降低迁移成本：用户原来使用 Cassandra 的数据模型和接口，可以较低成本尝试 Scylla。这样技术优势能够通过社区传播进入真实生产环境，再通过 Enterprise 功能、商业支持和 Cloud 服务完成商业化。

这里还有一个很重要的产品策略：**Scylla 没有要求市场重新学习一种数据库。**客户原来理解 Cassandra 的数据模型、读写模式、replication 和 operational model，只需要重新评估底层实现的性能。这样产品创新主要发生在系统内部，用户体验上的变化相对有限。这对于基础设施产品非常重要，因为数据库迁移本身就具有很高风险，任何额外的学习和改造都会增加 adoption 成本。

后来 Seastar 本身也开始产生独立价值。它原本服务于 Scylla，却逐渐被其他高性能基础设施项目采用，包括 Redpanda、Ceph Crimson 等。这里出现了一种基础设施创业里很有意思的现象：**一个具体产品逼出了底层技术，底层技术又开始成为平台。**

所以从创业史角度看，Scylla 可以拆成几个连续阶段：

```text id="6v7j0p"
已有的系统工程能力
        │
        ↓
KVM / Linux / Red Hat 人才网络
        │
        ↓
     Cloudius
        │
        ↓
       OSv
        │
        ↓
真实 workload benchmark
        │
        ↓
发现 Cassandra 的性能缺口
        │
        ↓
判断：
Cassandra 的市场和产品形态已经成熟
底层 implementation 仍有巨大优化空间
        │
        ↓
   C++ + Seastar
        │
        ↓
     ScyllaDB
        │
        ├─────────────┐
        ↓             ↓
 Cassandra 用户     开源社区
        │             │
        └──────┬──────┘
               ↓
          生产环境验证
               ↓
         VC + 产业资本
               ↓
     Enterprise / Cloud
               ↓
          商业化扩张
```

如果把它放到更大的技术创业框架里，Scylla 的故事体现了一种很典型的路径：**创始团队先拥有极深的底层技术能力，在真实工程环境里发现性能瓶颈，再寻找已经存在的市场需求，把底层能力转化成产品优势；开源负责降低技术验证和用户采用成本，VC 负责扩大研发、销售和市场规模，企业服务和云服务负责最终变现。**

这里最值得注意的地方，是**产品机会并非从商业市场分析中凭空产生，而是在工程实践里被发现的**。团队原本研究 OSv，benchmark Cassandra 时发现异常，深入分析以后发现了新的技术缺口。Cassandra 已经证明市场存在，Scylla 只需要证明自己的实现能够把同一类 workload 做得更快、更便宜。技术能力、已有市场、迁移路径、商业价值在这里形成了非常紧密的闭环。

这也是为什么 Scylla 很适合用来理解基础设施创业：**论文、技术、工程团队、开源社区、已有产品、VC、产业资本和商业客户并不是几个独立环节，它们会沿着一个具体的技术机会逐渐连接起来。**
