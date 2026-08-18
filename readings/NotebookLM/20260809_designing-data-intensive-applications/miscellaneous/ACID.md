# 🟢 **Atomicity, Consistency, Isolation, Durability**

通俗记忆法：

```text
Atomicity   → 全不全
Consistency → 合不合法
Isolation   → 互不互相干扰
Durability  → 掉电还在不在

例如银行转账就是经典的 ACID 示例：钱不能凭空消失（A/C），并发转账不能互相看到半成品（I），成功提交后不能因为数据库重启就消失（D）。 
```

<br>  

Page 306 / 673  
**Isolation**  

```text
Isolation in the sense of ACID means that concurrently executing transactions are
isolated from each other; they cannot step on each other’s toes. The classic database
textbooks formalize isolation as serializability, which means that each transaction can
pretend that it is the only transaction running on the entire database. The database
ensures that when the transactions have committed, the result is the same as if
they had run serially (one after another), even though in reality they may have run
concurrently [13].

However, serializability has a performance cost. In practice, many databases use
forms of isolation that are weaker than serializability—that is, they allow concurrent
transactions to interfere with each other in limited ways. Some popular databases,
such as Oracle, don’t even implement it (Oracle has an isolation level called “serializable”, 
but it actually implements snapshot isolation, which is a weaker guarantee than
serializability [10, 14]). This means that some kinds of race conditions can still occur.
We will explore snapshot isolation and other forms of isolation in “Weak Isolation
Levels” on page 288.
```

<br>  

Page 307 / 673  
**Replication and Durability**   

```text 
Historically, durability meant writing to an archive tape. Then it was understood as
writing to a disk or SSD. More recently, it has been adapted to mean replication.
Which implementation is better?

In practice, no one technique can provide absolute guarantees. There are only various
risk-reduction techniques—including writing to disk, replicating to remote machines,
and backups—and they can and should be used together. As always, it’s wise to take
any theoretical “guarantees” with a healthy grain of salt.
```

<br>  

Page 309 / 673  
Figure 8-2. Violating isolation: one transaction reads another transaction’s uncommitted
writes (a “dirty read”)   

Page 314 / 673   
Figure 8-4. No dirty reads: user 2 sees the new value for x only a$er user 1’s transaction
has committed  

Page 316 / 673  
Figure 8-5. With dirty writes, con!icting writes from di&erent transactions can be mixed
up  

<br>  

Page 359 / 673 
Table 8-1. Summary of anomalies that can occur at various isolation levels ✅    