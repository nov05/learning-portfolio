# 🟢 **Error Handling**   


Page 312 / 673 

```text
Errors will inevitably happen, but many software developers prefer to think only
about the happy path rather than the intricacies of error handling. For example,
popular object-relational mapping (ORM) frameworks such as Rails ActiveRecord
and Django don’t retry aborted transactions—the error usually results in an exception
bubbling up the stack, so any user input is thrown away, and the user gets an error
message. This is a shame, because the whole point of rolling back transactions is to
enable safe retries.
```

<br>  

ChatGPT:  

具体业务场景的组合几乎不可能穷尽，但可以提炼出有限的 **pattern types**。不同业务虽然完全不同，例如下单、转账、预订，底层仍可能落入相似的 failure patterns，再针对这些 patterns 设计对应的机制。这样学习分布式系统时，不需要枚举所有具体场景，而是识别反复出现的 pattern，再理解它对应的解决机制。

```text
下单 → 扣库存 → 扣款 → 发货
转账 → 扣款 → 入账
预订 → 锁定资源 → 支付 → 确认
```

```text
Pattern                     典型机制
────────────────────────────────────────
Multi-step dependency       Saga / workflow
Partial completion          Compensation
Retry + duplicate           Idempotency
Concurrent updates          Concurrency control
Multiple replicas           Replication
Cross-node agreement        Consensus
Atomic cross-service work   2PC
Out-of-order events         Ordering / sequence
```

