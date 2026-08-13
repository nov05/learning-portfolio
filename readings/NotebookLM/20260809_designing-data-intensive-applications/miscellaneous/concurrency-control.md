* Page 393 / 673

```text
When writing multithreaded code on a single machine, we have fairly good tools for making it thread-safe: mutexes, semaphores, atomic counters, lock-free data struc! tures, blocking queues, and so on. Unfortunately, these tools don’t directly translate to distributed systems, because a distributed system has no shared memory—only messages sent over an unreliable network.
```

这段话是在对比**单机多线程**和**分布式系统**。单机上多个线程共享同一块内存，所以可以直接用各种同步原语协调线程之间的执行顺序。几个词可以先这样理解：

| 机制                           | 简单理解                | 主要解决什么         |
| ---------------------------- | ------------------- | -------------- |
| **mutex**                    | 一把锁，同一时间只允许一个线程进入   | 保护共享数据         |
| **semaphore**                | 一个“计数的通行证”          | 控制同时允许多少线程执行   |
| **atomic counter**           | 对计数器做不可分割的读改写       | 安全地修改共享计数      |
| **lock-free data structure** | 不依赖传统 mutex 的并发数据结构 | 高并发下减少锁竞争      |
| **blocking queue**           | 队列空了就等，有数据就取；满了就等   | 在线程之间传递任务并协调速度 |

### mutex

**Mutex（mutual exclusion）就是互斥锁。**

假设多个线程都要修改：

```text
balance = 100
```

如果两个线程同时执行：

```text
balance = balance - 10
```

就可能发生竞争。

Mutex 的思路就是：

```text
lock()

balance = balance - 10

unlock()
```

同一时刻只有一个线程能够拿到锁。

可以把它理解成**厕所钥匙**：只有一个钥匙，一个线程进去以后，其他线程必须等。

---

### semaphore

Semaphore 和 mutex 很像，但它不是只有“0/1”两个状态，而是可以有一个**计数值**。

例如有 3 个数据库连接：

```text
semaphore = 3
```

最多允许三个线程同时使用：

```text
Thread A → acquire → 2
Thread B → acquire → 1
Thread C → acquire → 0

Thread D → acquire → 等待
```

当 A 用完：

```text
Thread A → release → 1
```

D 就可以进去。

所以：

**mutex = 同时只能一个**

**semaphore = 同时最多 N 个**

Semaphore 很适合做**资源数量限制**，例如连接池、线程池、有限数量的设备等。

---

### atomic counter

Atomic counter 就是**原子计数器**。

例如：

```text
counter = 100
```

多个线程同时执行：

```text
counter++
```

普通的 `counter++` 实际上包含：

```text
read
+
1
write
```

两个线程同时操作就可能丢更新。

Atomic counter 则保证：

```text
counter++
```

这个操作作为一个整体完成，中间不会被另一个线程插进来。

所以多个线程可以安全地：

```text
atomic_increment(counter)
```

最终得到正确结果。

它特别适合：

```text
请求数量
连接数量
任务数量
reference count
统计计数
```

这类简单共享状态。

---

### lock-free data structure

Lock-free data structure 更进一步：**数据结构的并发操作不依赖 mutex。**

例如一个普通 queue：

```text
Thread A
   ↓
  lock
   ↓
 queue.push()
   ↓
 unlock
```

如果大量线程同时操作，就会产生锁竞争。

Lock-free queue 则利用 CPU 提供的 atomic 操作，例如 **CAS（compare-and-swap）**：

```text
当前值 = A

如果仍然是 A
    ↓
改成 B
```

如果另一个线程已经修改过：

```text
A → C
```

那么 CAS 失败，当前线程重新读取并尝试。

它的核心思想可以粗略理解成：

> **大家不拿一把公共的锁，而是通过原子操作竞争修改数据。**

它通常能降低锁竞争带来的开销，不过实现难度明显高很多，需要处理 ABA、memory ordering 等问题。

---

### blocking queue

Blocking queue 是非常实用的一种线程协调机制。

假设一个线程负责生产任务：

```text
Producer
   ↓
[ A ][ B ][ C ]
              ↓
           Consumer
```

Consumer 从 queue 里面取任务。

如果 queue 是空的：

```text
Consumer
   ↓
queue.empty
   ↓
等待
```

Producer 放进去一个任务：

```text
Producer
   ↓
queue.push(D)
   ↓
Consumer 被唤醒
```

反过来，如果 queue 有最大容量：

```text
[A][B][C][D]
```

已经满了，那么 Producer 再放任务时也可以阻塞等待。

所以 blocking queue 同时解决了两个问题：

**线程之间传递任务 + 控制生产速度和消费速度。**

这就是常见的 producer-consumer pattern。

---

把这几个东西放在一起看，就比较容易理解原文为什么强调 **distributed systems 没有 shared memory**。

单机上：

```text
Thread A ─┐
Thread B ─┼──→ Shared Memory
Thread C ─┘
```

所以可以直接：

```text
mutex
semaphore
atomic
queue
```

来协调。

分布式系统变成：

```text
Machine A                  Machine B

Thread A                   Thread B
   │                          │
   └────── network ──────────┘
```

中间没有共同的内存可以直接加锁。于是问题变成：

> **如果 A 想“锁住”某个资源，B 怎么知道？如果 A 获得锁以后机器挂了怎么办？如果网络消息丢了怎么办？如果 A 认为自己拿到了锁，但 B 也认为自己拿到了锁怎么办？**

这就是为什么分布式系统里面会出现 **distributed lock、consensus、leader election、distributed transaction** 等东西。

所以这段话真正想表达的是：**单机并发控制的工具很多都建立在“共享内存 + 可靠通信”这个前提上；一旦机器之间只能通过不可靠网络传消息，同样的问题就会突然困难很多。**
