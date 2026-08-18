# 🟢 **Snapshot Isolation**   

Page 318 / 673   
```text
Snapshot isolation [38] is the most common solution to this problem. The idea is that
each transaction reads from a consistent snapshot of the database—that is, it sees all
the data that was committed in the database at the start of that. Even if the data is
subsequently changed by another transaction, each transaction sees only the old data
from that particular point in time.

Snapshot isolation is a boon for long-running, read-only queries such as backups and
analytics. It is very hard to reason about the meaning of a query if the data on which
it operates is changing at the same time as the query is executing. When a transaction
can see a consistent snapshot of the database, frozen at a particular point in time, it is
much easier to understand.

Snapshot isolation is a popular feature: variants of it are supported by PostgreSQL,
MySQL with the InnoDB storage engine, Oracle, SQL Server, and others, although
the detailed behavior varies from one system to the next [30, 42, 43]. Some databases,
such as Oracle, TiDB, and Aurora DSQL, even choose snapshot isolation as their
highest isolation level. Cloud data warehouses such as BigQuery frequently use snapshot 
isolation as well, as it provides a point-in-time view of the database for analytical
queries.
```

<br>  
