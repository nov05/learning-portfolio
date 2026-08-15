# **👉 Write Skew**

**Write skew**（写偏差）是并发事务中的一种异常：

> **两个事务读取了相同的一组数据/共同的业务条件，各自修改不同的记录，最终两个修改组合起来违反了原本应该满足的约束。**

典型结构：

```text
T1: 读条件 ──→ 修改 Row A
T2: 读条件 ──→ 修改 Row B

        ↓

两个事务互不修改同一行
        ↓
却共同破坏了一个跨行约束
```

例如规则：

> 至少有一个医生值班。

初始：

```text
Alice = on_call
Bob   = on_call
```

同时：

```text
T1: 看到有 2 人值班 → Alice 下班
T2: 看到有 2 人值班 → Bob 下班
```

最后：

```text
Alice = off
Bob   = off
```

**0 人值班，约束被破坏。**

关键特征就是：

1. **两个事务都读取了相关数据**
2. **两个事务修改的是不同的 row**
3. **单独看每个事务似乎都合法**
4. **两个事务一起提交后，违反了跨行/整体 invariant**

所以它叫 **write skew**，而不是简单的 lost update。

**Lost update：** 两个事务写同一条记录，互相覆盖。
**Write skew：** 两个事务写不同记录，但共同造成逻辑约束失效。

# 👉 **Scenario**

```text
Watch the Premise: If your code says IF (select count(*) ...) THEN (update ...), you are probably vulnerable to Write Skew. You need SELECT FOR UPDATE or serializable isolation.
```

这句话讲的是数据库并发控制里一个很重要的坑：**你检查的是“整体条件”，但更新的是“局部记录”**。

### 先看一个典型例子

假设有一个值班表：

```text
doctors
----------------
Alice   on_call = true
Bob     on_call = true
```

业务规则：

> **至少必须有一个医生 on call。**

现在 Alice 想下班：

```sql
IF (SELECT COUNT(*)
    FROM doctors
    WHERE on_call = true) > 1
THEN
    UPDATE doctors
    SET on_call = false
    WHERE name = 'Alice';
```

看起来完全合理：

> “现在有 2 个医生值班，所以我可以把自己设成 off call。”

但问题来了：**Alice 和 Bob 同时执行。**

---

## 两个 transaction 同时运行

```text
Transaction A (Alice)       Transaction B (Bob)

COUNT(*) = 2                COUNT(*) = 2
     ↓                           ↓
"可以下班"                    "可以下班"
     ↓                           ↓
Alice = false                Bob = false
     ↓                           ↓
COMMIT                       COMMIT
```

最后：

```text
Alice = false
Bob   = false
```

结果：

> **0 个医生值班。**

业务规则被破坏了。

---

## 这就是 Write Skew

关键在于：

**Alice 没有修改 Bob 的那一行。**

Bob 也没有修改 Alice 的那一行。

所以如果数据库只关注：

> “两个 transaction 有没有修改同一行？”

答案是：

> **没有。**

因此普通的 row-level locking 可能不会阻止它们同时提交。

但它们实际上修改了**同一个逻辑约束**：

```text
COUNT(on_call = true) >= 1
```

这就是 **write skew**。

---

# 为什么 `SELECT FOR UPDATE` 能解决？

你可以把：

```sql
SELECT COUNT(*)
FROM doctors
WHERE on_call = true;
```

改成锁住相关行：

```sql
SELECT *
FROM doctors
WHERE on_call = true
FOR UPDATE;
```

于是 Alice 执行：

```text
Alice transaction
      ↓
SELECT ... FOR UPDATE
      ↓
锁住 Alice + Bob
```

这时候 Bob 想执行同样的操作：

```text
Bob transaction
      ↓
SELECT ... FOR UPDATE
      ↓
        等待
        ↓
Alice COMMIT
        ↓
Bob 获得锁
        ↓
重新检查条件
```

于是 Bob 看到：

```text
COUNT(on_call = true) = 1
```

就不能下班了。

---

# 为什么 `SERIALIZABLE` 也可以？

因为 **serializable isolation** 的目标就是让并发执行的结果看起来像：

```text
A 先执行
↓
B 再执行
```

或者：

```text
B 先执行
↓
A 再执行
```

而不能出现某种并行交错导致一个**根本不可能由任何串行执行产生的结果**。

在这个例子里：

```text
A:
看到 2
→ A off

B:
看到 2
→ B off
```

这个结果：

```text
0 on-call
```

不可能通过合法的串行执行得到。

所以 serializable 实现应该检测/阻止这种并发冲突，通常让其中一个 transaction abort/retry。

---

## 最值得记住的一点

**Write skew 和普通的 lost update 不一样。**

### Lost update

两个 transaction：

```text
UPDATE 同一行
```

容易被 row lock / MVCC conflict 发现。

### Write skew

两个 transaction：

```text
读取相同的整体条件
        ↓
修改不同的行
```

例如：

```text
A 读：至少两个人值班
B 读：至少两个人值班

A 改 Alice
B 改 Bob
```

**它们写的是不同的 row，但违反的是同一个跨行 invariant。**

所以你看到的这句话：

> `IF (select count(*) ...) THEN (update ...)`

其实是在提醒：

> **“如果你的业务逻辑是先读取一个 aggregate/predicate，然后根据这个 predicate 修改某一行，你必须考虑并发 transaction 是否可能同时通过这个检查。”**

这就是为什么 **`SELECT FOR UPDATE` 或 `SERIALIZABLE`** 经常出现在解决 write skew 的讨论里。

一句话：

> **Row-level conflict 看的是“谁改了同一行”；write skew 要看的是“谁破坏了同一个业务规则”。**
