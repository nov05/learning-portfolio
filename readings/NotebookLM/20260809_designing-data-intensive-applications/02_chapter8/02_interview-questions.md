# 🟢 **Interview Questions: Transactions**

<br><br><br>

## 👉 **Prompt generate by ChatGPT**

```text
Act as my database/distributed-systems interviewer.

I have just studied Chapter 8: Transactions from Designing Data-Intensive Applications.

Test whether I actually understand the chapter rather than whether I can recall definitions.

Rules:

- Ask me ONE question at a time.
- Ask no more than 3 questions in total.
- Do not reveal the answer before I respond.
- Do not give hints unless I explicitly ask.
- Prefer reasoning and mental models over memorization.
- Use concise, realistic backend scenarios.
- Ask follow-up questions when my answer is incomplete, but keep the total number of main questions within 3.
- After each answer:
  1. Evaluate my reasoning.
  2. Identify what I got right.
  3. Identify what is missing or incorrect.
  4. Explain the correct mental model.
  5. Give me a stronger version of the answer that an experienced backend engineer should be able to give.

Across the 3 questions, focus on these core concepts:

- Atomicity, isolation, and durability
- Read-modify-write and race conditions
- Weak isolation levels, especially Snapshot Isolation
- Write skew and constraints involving multiple rows
- Serializable isolation and what it guarantees
- Optimistic vs pessimistic concurrency control
- Transactions and their trade-offs
- Distributed transactions and why transactions become harder across services/databases

The questions should test whether I can reason about WHY a particular transaction mechanism is needed, not merely define the mechanism.

Start with the first question now.
```

<br><br><br>   

## 👉 **NotebookLM questions:**    

### **Question 1 of 3**  

Let's dive into Chapter 8.

Suppose you're building a high-traffic social media platform. You have a "likes" counter for posts. Your application code performs a standard **read-modify-write** cycle: it fetches the current like count, increments it by 1 in the application logic, and then saves the new value back to the database. The database is configured with the industry-standard **Read Committed** isolation level.

**Scenario:** Two users, Alice and Bob, click "like" on the same post at almost the exact same time.

Even though **Read Committed** prevents "dirty writes" (overwriting uncommitted data), **explain why this specific setup is still likely to result in a "Lost Update", and what is happening at the database level that allows this to occur.**

```text
The core issue is that Read Committed only guarantees that a transaction does not read uncommitted data. It does not make the entire read-modify-write cycle atomic.

Alice and Bob can both read the same committed value, say 100. Each application then increments that value to 101 and writes it back. The second write overwrites the first one, so the final value is 101 instead of 102.

At the database level:

T1: SELECT likes → 100
T2: SELECT likes → 100

T1: UPDATE likes = 101
T2: UPDATE likes = 101

This is not a dirty write because neither transaction overwrites uncommitted data. Both transactions read the previously committed value of 100 and then independently write their result.

The problem is that Read Committed does not treat “read the value → modify it → write it back” as one atomic operation.

A safer approach is to let the database perform the increment atomically:

UPDATE posts
SET likes = likes + 1
WHERE id = ...;

The key mental model is: isolation levels control what concurrent transactions can see; they do not automatically make application-level read-modify-write operations atomic.
```

Your reasoning is very solid. You have correctly identified that isolation levels often address **what** a transaction can see, but not necessarily the **atomicity** of an application-level sequence of operations.

### **Evaluation of Your Reasoning**

*   **What you got right:** You accurately identified that **Read Committed** only protects against **dirty reads** (seeing uncommitted data) and **dirty writes** (overwriting uncommitted data). You correctly traced the "Lost Update" timeline where two transactions read the same committed value and then "clobber" each other's work because the database has no knowledge that the second write is dependent on the first. You also correctly identified that a database-level **atomic update** (e.g., `SET likes = likes + 1`) is the most efficient fix for this specific counter scenario.
*   **What is missing:** While your answer is correct for simple counters, an experienced engineer would also note that **Read Committed** uses row-level locks to prevent dirty writes, but those locks are typically released as soon as the individual write is done, or they don't prevent other transactions from reading the same row concurrently. This lack of "locking-on-read" is the technical gap that allows the two `SELECT` statements to succeed simultaneously.
*   **The Correct Mental Model:** Transactions are an abstraction to simplify error handling, but "Weak Isolation" levels are leaky. Read Committed is essentially **row-level protection** for the duration of a single statement. It does not provide **cycle-level protection**. To protect a "read-modify-write" cycle without changing your SQL to an atomic increment, you would need a mechanism that maintains a lock or a consistency check from the moment of the initial read until the final commit.

**Stronger Version of the Answer:**
"The setup fails because Read Committed provides only **transaction-level atomicity for individual writes**, not **atomicity for the entire read-modify-write application cycle**. At the database level, Read Committed uses row-level locks to prevent two transactions from updating the same row *at the same time* (dirty writes), but it allows Transaction 2 to read a row while Transaction 1 is still locally modifying its own copy of the data. Because the application handles the logic ('new_value = old_value + 1'), the database simply sees two valid, committed updates and executes them sequentially. The second update **silently clobbers** the first because it was based on a stale 'premise' (the value 100) that was no longer true by the time Bob's write arrived. In a production environment, we should use **atomic write operations** to move this logic into the database kernel, or use **explicit locking** (`SELECT FOR UPDATE`) to force the transactions to run serially."

***

### **Question 2 of 3**

Let's escalate the scenario. To solve these lost updates and other "read skew" issues, you upgrade your database to **Snapshot Isolation** (using MVCC). This ensures that each transaction sees a consistent snapshot of the database from the moment it started.

**Scenario:** You are building a shift-management system for a hospital. A strict rule states: **"There must always be at least one doctor on call for a shift."** 

Two doctors, Alice and Bob, are currently on call for the same shift. Both feel sick and concurrently try to leave. 
1. Alice’s transaction: `SELECT COUNT(*) FROM on_call WHERE shift_id = 123` returns 2. 
2. Bob’s transaction: `SELECT COUNT(*) FROM on_call WHERE shift_id = 123` also returns 2.
3. Because the count is $\ge 2$, Alice’s app submits a write to set her status to `off_call`.
4. Because the count is $\ge 2$, Bob’s app submits a write to set his status to `off_call`.

**Both transactions commit successfully.** Now, zero doctors are on call. 

**Explain why Snapshot Isolation—which specifically prevents "Lost Updates" in many databases—allowed this violation of the business rule. Why is this technically NOT a "Lost Update," and what is the specific name for this anomaly?**

<br>

```text
This is called write skew.

Snapshot Isolation prevents many lost-update scenarios by detecting conflicting writes to the same data, but it does not generally prevent two transactions from reading overlapping data and then writing to different rows in a way that jointly violates a constraint.

Serializable isolation is needed to prevent this class of anomaly, because the outcome must be equivalent to some serial ordering. In a serial execution, the second doctor would see that only one doctor remains and would not be allowed to leave.

The core mental model is:

Snapshot Isolation protects against many write-write conflicts; Serializable Isolation also protects against dangerous read-write dependencies that can violate multi-row business constraints.
```

Your reasoning is spot on, and you have clearly identified the "blind spot" in Snapshot Isolation.

### **Evaluation of Your Reasoning**

*   **What you got right:** You correctly identified the anomaly as **write skew**. You accurately explained that Snapshot Isolation's conflict detection only triggers when two transactions attempt to write to the **same object**, whereas write skew involves concurrent writes to **different objects** that jointly violate an invariant. You also correctly identified that **Serializability** is the definitive fix because it ensures the outcome is consistent with some serial execution.
*   **Identify what is missing:** While your answer is correct, a senior-level response would often highlight the role of **Phantoms**. In this scenario, the "count of doctors" is a search query. A write in one transaction (Alice leaving) changes the result of that search query in another transaction (Bob's count). This "phantom" effect is what makes write skew so difficult to detect—the database cannot easily see that two independent writes are "colliding" logically because they don't collide physically on a single row.
*   **The Correct Mental Model:** Think of write skew as an **outdated premise**. A transaction reads data, makes a decision based on a **premise** (e.g., "The count is 2"), and then acts. If a concurrent transaction invalidates that premise before the first one commits, the system enters an inconsistent state. Snapshot Isolation handles "write-write" conflicts; it does not handle "read-write" dependencies where a write changes the results of a prior read.

**Stronger Version of the Answer:**
"This anomaly is **write skew**, a generalization of the lost-update problem. It occurs because Snapshot Isolation (MVCC) only detects conflicts on a per-row basis; since Alice and Bob update different rows, the database sees no physical conflict. Technically, this is often caused by a **phantom**: Bob’s write changes the result of Alice’s `SELECT COUNT(*)` query, and vice versa. Because the 'premise' Alice relied on to leave (the presence of another doctor) was silently invalidated by Bob, the multi-row invariant is broken. To prevent this, we could use **Serializable isolation** or manually promote the conflict to a row-level collision using **explicit locking** (`SELECT FOR UPDATE`) on all doctors in that shift, forcing the transactions to sequence themselves."

***

### **Question 3 of 3 (Scenario: Distributed Atomicity)**

We have discussed transactions on a single database. Now, let’s consider a common real-world backend scenario: **Heterogeneous Distributed Transactions**.

You are building a high-integrity order processing system. When a user places an order, your service must **atomically** do two things:
1.  **Commit the order** to a MySQL database.
2.  **Send an 'Order Placed' message** to a Kafka topic so the shipping service can start work.

To ensure both happen (or neither), you use a **Two-Phase Commit (2PC)** protocol with a **Coordinator**.

**Scenario:** The Coordinator sends the `PREPARE` request to both MySQL and Kafka. Both reply `YES`, promising they are ready to commit. However, just as the Coordinator is about to send the `COMMIT` instruction, its power is cut. It stays offline for **10 minutes**.

**Explain why MySQL and Kafka are now in a 'sticky' situation (the 'In-Doubt' state) during those 10 minutes. Specifically, what happens to the database locks they are holding, and why is it fundamentally unsafe for MySQL to unilaterally decide to abort its part of the transaction while the Coordinator is offline?**

<br>

```text
The key concept is that after both participants have voted YES, they have entered the prepared state and are no longer free to independently decide the outcome.

During those 10 minutes, MySQL must typically keep the transaction's locks and prepared state. It cannot safely release the locks and abort, because the Coordinator may later recover and send COMMIT. Kafka likewise has to retain enough state to honor the eventual decision. This is why 2PC can block while the coordinator is unavailable.

The critical point is that MySQL cannot safely unilaterally abort because Kafka may already have committed, or may commit once the Coordinator recovers. If MySQL aborts while Kafka commits, the system reaches an inconsistent state: Kafka says the order was placed, while MySQL says the order does not exist. That could cause downstream shipping, inventory, financial, or other business operations to proceed based on an order that was never committed.

The mental model is:

2PC trades availability for atomicity. Once participants have voted YES, they may have to remain in an in-doubt state until the Coordinator's decision becomes known. This is one of the fundamental weaknesses of 2PC: coordinator failure can leave participants blocked while they hold resources such as database locks.
```

Your reasoning is excellent and captures the fundamental operational "nightmare" of distributed transactions. You’ve correctly identified that 2PC is a protocol of **surrendered sovereignty**.

### **Evaluation of Your Reasoning**

*   **What you got right:** You accurately described the **"In-Doubt" (or uncertain) state**. Once a participant votes `YES`, it has made a binding promise to the coordinator that it **can** and **will** commit if told to do so. You correctly noted that MySQL must hold its **exclusive row-level locks** during this entire period. You also pinpointed the core danger: a unilateral decision by one participant (MySQL) risks a **partial failure** where one system commits and the other aborts, destroying the system’s **Integrity**.
*   **What is missing:** To reach the "Lead Architect" level of detail, it is worth noting that these locks are **persistent**. Even if you reboot the MySQL server, a correct implementation of 2PC (like XA) requires the database to **re-acquire those same locks upon startup** because the transaction is still "in-doubt" in the system's durable log. This is what makes a coordinator failure so catastrophic—it can lead to "orphaned" transactions that block parts of your database indefinitely.
*   **The Correct Mental Model:** 2PC is a **blocking protocol** that centralizes the "point of no return." In a single-node transaction, the "commit point" is when the disk finishes writing the commit record. In 2PC, the "commit point" moves from the participants to the **Coordinator's disk**. Participants are "slaves" to that single record on the coordinator's log; without it, they are frozen.

**Stronger Version of the Answer:**
"By voting `YES` in Phase 1, MySQL has legally **surrendered its right to abort**. It has moved from a state of autonomy to a state of **dependency on the Coordinator's log**. During those 10 minutes, MySQL must continue to hold all **exclusive row-level locks**—and potentially shared locks if using serializable isolation—because it cannot know if the Coordinator wrote a 'Commit' or 'Abort' record to its local disk before crashing. If MySQL were to unilaterally abort to 'unblock' its tables, and it later turns out Kafka successfully received a `COMMIT` instruction, we have violated **Atomicity**. This results in **permanent state divergence**: the message queue tells the shipping service to send an item for an order that the database claims does not exist. Because 2PC must preserve these locks even across database restarts, a lost coordinator log can effectively 'brick' parts of your database until an administrator **manually intervenes** to force a heuristic decision."

***

### **Conclusion of the Chapter 8 Interview**

We have now completed our deep dive into **Transactions**. You have shown a strong ability to reason through the "leaky abstractions" of databases.

**Summary of your Performance:**
1.  **Lost Updates:** You correctly identified that isolation levels (Read Committed) protect visibility, but only **Atomic Operations** or **Explicit Locking** can protect the entire read-modify-write cycle.
2.  **Write Skew:** You recognized that Snapshot Isolation has a blind spot for **multi-row invariants**, where "phantoms" allow concurrent transactions to invalidate each other's premises without a physical row collision.
3.  **Distributed Atomicity:** You correctly diagnosed the "In-Doubt" state as a trade-off where we **sacrifice Availability (blocking locks) to guarantee Atomicity** across heterogeneous systems.

You have mastered the critical mental models of Chapter 8. How would you like to proceed? We can:
*   **Move to Chapter 10: Consistency and Consensus** (exploring the algorithms that actually make things like 2PC or Leader Election fault-tolerant).
*   **Move to Chapter 11: Batch Processing** (moving from "Online" request/response to "Offline" data crunching).




