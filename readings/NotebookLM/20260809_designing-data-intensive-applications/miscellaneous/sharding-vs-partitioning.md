# 🟢 **Sharding vs. Partitioning**  

Page 305 / 673  

```text
Consistent hashing is an approach to sharding that some systems use for rebalancing (see “Consistent hashing” on page 263).
```

👉 **ChatGPT:**

They’re related, but not quite the same.

### Partitioning

**Partitioning = splitting data into smaller logical pieces.**

A single database/table is divided into partitions, usually based on a key.

Example:

```text
Users table
├── Partition 1: user_id 1–1M
├── Partition 2: user_id 1M–2M
└── Partition 3: user_id 2M–3M
```

The partitions may still live on **the same database server**.

Common reasons:

* Faster queries by scanning only relevant partitions
* Easier maintenance/deletion of old data
* Better indexing/storage management
* Handling very large tables

### Sharding

**Sharding = distributing data across multiple independent database servers.**

```text
                Users
                  |
       ┌──────────┼──────────┐
       ↓          ↓          ↓
    Shard 1    Shard 2    Shard 3
    Server A   Server B   Server C
    users 1M   users 2M   users 3M
```

Each shard contains a subset of the data and typically has its own CPU, memory, storage, and database process.

Common reasons:

* Scale beyond one machine's storage
* Scale reads/writes across machines
* Increase availability/fault isolation
* Handle enormous datasets or traffic

### The key distinction

|                  | Partitioning                | Sharding                    |
| ---------------- | --------------------------- | --------------------------- |
| What is split?   | Data                        | Data                        |
| Across machines? | Not necessarily             | **Yes**                     |
| Scope            | Usually within one DB/table | Across DB instances/servers |
| Main goal        | Manage/query large datasets | Horizontal scaling          |
| Example          | Table → monthly partitions  | Users → servers by user ID  |

### An easy way to remember

**Partitioning is about dividing data.
Sharding is about distributing those divisions across machines.**

In fact, **sharding can be thought of as a form of horizontal partitioning where the partitions are placed on different database nodes.**

For example, a PostgreSQL database could have:

```text
             Users
               |
      Horizontal partitioning
        /       |       \
       /        |        \
   P1: A-H    P2: I-P    P3: Q-Z
```

If all three are on one server → **partitioning**.

If they're placed on three servers → **sharding**.

One subtlety: terminology varies by database system. Some systems use *partitioning* for both local and distributed data layouts, so the exact distinction depends on the technology.
