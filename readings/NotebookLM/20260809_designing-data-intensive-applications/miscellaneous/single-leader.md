Page 223 / 673 

```text
Single-leader replication is very widely used. It’s a built-in feature of many relational
databases, such as PostgreSQL, MySQL, Oracle Data Guard [3], and SQL Server’s
Always On availability groups [4]. It is also used in some document databases (such
as MongoDB and DynamoDB [5]), message brokers such as Kafka, replicated block
devices such as DRBD, and some network filesystems. Many consensus algorithms—
such as Raft, which is used for replication in CockroachDB [6], TiDB [7], etcd, and
RabbitMQ quorum queues (among others)—are also based on a single leader and
automatically elect a new leader if the old one fails (we will discuss consensus in more
detail in Chapter 10).
```