# 🟢 **Lost Updates**  

Page 323 / 673  
```text
Several other interesting kinds of conflicts can occur between concurrently writing
transactions. The best known of these is the lost update problem, illustrated in
Figure 8-1 with the example of two concurrent counter increments.

The lost update problem can occur if an application reads a value from the database,
modifies it, and writes back the modified value (the read-modify-write cycle men!
tioned earlier). If two transactions do this concurrently, one of the modifications
can be lost, because the second write does not include the first modification. (We
sometimes say that the later write clobbers the earlier write.)
```

<br>  

Page 324 / 673  
```text
Unfortunately, ORM frameworks make it easy to accidentally write code that per!
forms unsafe read-modify-write cycles instead of using atomic operations provided
by the database [51, 52, 53]. This can be a source of subtle bugs that are difficult to
find by testing.
```

ChatGPT:

**ORM (Object-Relational Mapping)** 是一层把程序里的对象映射到关系数据库表的抽象层。它让开发者可以通过对象操作数据库，ORM 再生成对应的 SQL。问题在于，这种抽象很容易把一个本应由数据库原子完成的操作写成 **Read → Modify → Write**，并发情况下就可能产生 race condition。

```text id="3q8v2n"
程序对象                     数据库
────────────────────────────────────
User object        →        users row
Order object       →        orders row
obj.save()         →        UPDATE
obj.delete()       →        DELETE
```

例如两个请求同时给账户扣 $10：

```text id="5c1m7x"
A: Read balance = 100
B: Read balance = 100

A: balance -= 10
B: balance -= 10

A: Write balance = 90
B: Write balance = 90

最终：90
正确：80
```

而数据库可以直接执行：

```text id="h7p3qd"
UPDATE account
SET balance = balance - 10
WHERE id = ...;
```

这里的关键区别是：**前者把状态计算放在应用层，后者把操作交给数据库作为一个原子更新处理。**

```text id="2k6m9r"
ORM                         典型风险
────────────────────────────────────────
Object → Read               读取旧状态
Modify                      并发修改
Write                       覆盖其他更新
```

<br>  

Page 325 / 673  
```text
PostgreSQL’s repeatable read, Oracle’s
serializable, and SQL Server’s snapshot isolation levels automatically detect when
a lost update has occurred and abort the offending transaction. However, MySQL/
InnoDB’s repeatable read isolation level does not detect lost updates [30, 43]. Some
authors [38, 40] argue that a database must prevent lost updates in order to qualify
as providing snapshot isolation, so MySQL does not provide snapshot isolation under
this definition.
```