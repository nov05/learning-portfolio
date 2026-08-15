Based on Chapter 8 of the sources, here are the five most critical engineering trade-offs regarding transactions and concurrency control.

### 1. Isolation: Read Committed vs. Serializability
**Naive approach:** **Read Uncommitted**, where a transaction can read data that has been modified by another transaction but not yet committed.
*   **→ Problem:** Leads to **dirty reads** (seeing partially updated or aborted data) and **dirty writes** (overwriting uncommitted changes), causing permanent data corruption or nonsensical user views.
*   **→ Solution:** **Read Committed isolation**, which uses row-level locks to prevent dirty writes and keeps old versions of objects to prevent dirty reads.
*   **→ Benefit:** Effectively prevents the most basic and dangerous race conditions with very low performance overhead.
*   **→ Cost:** Remains vulnerable to **read skew** (nonrepeatable reads), **lost updates**, and **write skew**.
*   **→ When to use:** Standard web applications where minor temporary inconsistencies (like a slightly stale count) are acceptable in exchange for high performance.
*   **→ When NOT to use:** Financial systems of record or integrity-critical systems where strict invariants (e.g., "account balance must never be negative") must hold across multiple objects.

---

### 2. Concurrency Control: Pessimistic (2PL) vs. Optimistic (SSI)
**Naive approach:** Standard **Snapshot Isolation** (MVCC).
*   **→ Problem:** While fast, it is prone to **write skew** and **phantoms**, where a transaction makes a decision based on a premise that changes before it commits (e.g., two doctors both quit their shift because they both see two doctors are currently on call).
*   **→ Solution:** **Serializable Snapshot Isolation (SSI)**, which runs transactions optimistically but tracks read and write sets to detect serialization conflicts.
*   **→ Benefit:** Provides full **Serializability** (the strongest isolation guarantee) without the massive blocking overhead of pessimistic locking.
*   **→ Cost:** If a conflict is detected, the transaction must be **aborted and retried** by the application, wasting CPU and potentially causing "retry storms" under high load.
*   **→ When to use:** Systems with low-to-moderate contention that require absolute correctness and protection against all race conditions.
*   **→ When NOT to use:** Extremely high contention workloads where the high abort rate would destroy throughput; in those cases, pessimistic **Two-Phase Locking (2PL)** may be more stable despite its latency.

---

### 3. Distributed Integrity: Dual Writes vs. Two-Phase Commit (2PC)
**Naive approach:** **Dual writes**, where the application code explicitly writes to two different systems (e.g., a database and a search index) independently.
*   **→ Problem:** One write succeeds while the other fails (violating atomicity), or concurrent writes arrive at the two systems in different orders, leading to permanent state divergence.
*   **→ Solution:** **Two-Phase Commit (2PC)**, a protocol where a coordinator ensures that all participating nodes either atomically commit or all abort.
*   **→ Benefit:** Guarantees **atomicity** across multiple database nodes, shards, or even heterogeneous systems.
*   **→ Cost:** Significant performance penalty due to multiple network round trips and forced disk fsyncs; it is also a **blocking protocol** where a coordinator failure can leave nodes "in-doubt" and holding locks indefinitely.
*   **→ When to use:** Critical multi-shard operations within a single database system (Internal 2PC) where performance is optimized by the vendor.
*   **→ When NOT to use:** Integrating heterogeneous systems (e.g., MySQL and Kafka) via **XA transactions**; the operational fragility and performance cost are usually too high. Use **idempotence** and **request IDs** instead.

---

### 4. Lost Update Prevention: Application Logic vs. Atomic Operations
**Naive approach:** **Read-modify-write** cycle in application code (Read value → calculate new value → write back).
*   **→ Problem:** If two clients do this concurrently, one update will silently overwrite (clobber) the other, leading to **Lost Updates**.
*   **→ Solution:** **Atomic write operations** provided by the database (e.g., `UPDATE ... SET value = value + 1`) or automatic **lost update detection** in the storage engine.
*   **→ Benefit:** Simplifies application code and avoids the overhead of manual locking or full serializability.
*   **→ Cost:** Atomic operations are limited to what the database supports (e.g., basic math); detection requires the application to handle transaction aborts and retries.
*   **→ When to use:** Simple counters, account balance updates, or appending elements to a list.
*   **→ When NOT to use:** Complex business logic that requires multi-object checks (e.g., "is there another figure at this board position?"); these require **explicit locking** (`SELECT FOR UPDATE`) or **Serializability**.

---

### 5. Throughput: Interactive Transactions vs. Stored Procedures
**Naive approach:** **Interactive transactions**, where SQL statements are sent one by one from the application to the database over the network.
*   **→ Problem:** Network latency becomes a bottleneck, especially in **Serial Execution** systems where a single slow transaction stalls all other processing.
*   **→ Solution:** Encapsulating the entire transaction logic in a **Stored Procedure** that executes locally on the database server.
*   **→ Benefit:** Enables **actual serial execution** on a single thread, reaching extremely high throughput by eliminating network/disk I/O from the critical path.
*   **→ Cost:** Stored procedure code is notoriously difficult to debug, test, version control, and monitor compared to standard application code.
*   **→ When to use:** High-throughput OLTP systems where transactions are small, fast, and the active dataset fits in memory.
*   **→ When NOT to use:** Long-running analytical queries or large-scale multi-shard transactions that would block the single-threaded execution loop.