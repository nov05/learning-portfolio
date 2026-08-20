# 🟢 **Linearizability vs. Serializability**   

这样记：Linearizability 管的是“一个东西在现实时间里看起来像只有一个副本”，Serializability 管的是“几笔交易一起执行时，结果看起来像按某个顺序一笔一笔执行”。最简单的记忆句是：**Linearizability 看时间，Serializability 看交易。**

比如一个银行账户当前是 100。A 操作把它改成 80，A 已经返回成功以后，B 再读取账户，结果必须至少看到 80，这属于 Linearizability，因为 B 是在 A 完成以后才开始的，系统需要尊重这个现实时间上的先后关系。这里甚至没有 transaction 也可以讨论 Linearizability。

Serializability 则看这种情况：Transaction A 同时修改账户和订单，Transaction B 同时读取账户和订单。系统允许它们并发执行，但最后的结果需要等价于某一种串行执行，例如：`A → B` 或者 `B → A`。只要最终效果等价于某个合法的串行顺序，就满足 Serializability。这里关注的是 transaction 之间的读写冲突和执行结果。

我会用两个词记：

```text
Linearizability → real-time
Serializability  → transactions
```

压缩成一句话：Linearizability 问“前一个已经完成，后一个能不能装作没发生？”；Serializability 问“这些交易并发跑完以后，能不能看起来像排队一个个跑？”因此，看到题目时，如果重点是“操作 A 已经返回成功，操作 B 随后开始，却看不到 A 的结果”，先想到 Linearizability；如果重点是“多个 transaction 并发执行，结果是否等价于某个串行顺序”，先想到 Serializability。
