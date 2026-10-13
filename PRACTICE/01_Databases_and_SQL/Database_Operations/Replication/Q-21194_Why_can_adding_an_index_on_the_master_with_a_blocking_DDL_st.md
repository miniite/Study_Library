---
type: "practice"
difficulty: "Deep"
---

# Why can adding an index on the master with a blocking DDL stall all replicas, and how do tools like pt-online-schema-change help?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Schema Changes and Failover](<../../../../01_Databases_and_SQL/Database_Operations/Replication/Failback_After_Failover/Schema_Changes_and_Failover.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why adding an index on the master with a blocking DDL stalls all replicas, we need to delve into the mechanics of database replication in a master-slave architecture and explore how tools like pt-online-schema-change can help mitigate this issue.

#### Triaging the Question

The core question is about the behavior of database indexes during DDL operations on a master node and their impact on replica nodes. Specifically, we are asked to explain why blocking DDLs (like CREATE INDEX) can stall all replicas and how tools like pt-online-schema-change address this problem.

Key Assumptions:
The database is in a master-slave replication setup.
Replicas receive updates from the master through binary logs.
Blocking DDL operations block new transactions on the master, potentially causing delays or stalls in replica synchronization.

#### First-Principles Solution
Why Blocking DDLs Stall Replicas

When you execute a blocking DDL operation like CREATE INDEX on the master, it temporarily blocks further write operations until the index creation is complete. During this time:

Write Operations Stalled: Any new writes to the database will be blocked, preventing any updates from reaching the replicas.
Binary Log Backlog: While these writes are stalled, the binary logs (which contain all changes made by transactions) continue to accumulate on the master but cannot be replicated immediately.

When a replica attempts to apply these accumulated binary log events:

If it encounters an operation that requires data not yet available in its local state (because of blocking write operations), the replica will stall or fail.
The replica may need to wait until the master has processed enough transactions to catch up, leading to significant delays.
Impact on Replicas

Replicas are designed to be asynchronous and eventually consistent. However, a blocking DDL can cause temporary consistency issues:

Stalled Replication: Replicas might stall as they await the completion of the DDL operation.
Out-of-Sync State: If replicas have not caught up with the master by the time the DDL completes, they will be out of sync.
Tools like pt-online-schema-change

To mitigate these issues, tools like pt-online-schema-change are designed to perform schema changes in a way that minimizes downtime and reduces the impact on replica synchronization. Here’s how it works:

Pre-Create Index: The tool creates an empty index first and then gradually populates it with data from the master.
Row-by-row Data Transfer: Instead of blocking write operations, pt-online-schema-change processes each row individually, ensuring that writes can continue on the master during the operation.
Parallel Execution: It handles multiple rows in parallel using a worker thread model, reducing the overall impact and duration of the DDL.
How pt-online-schema-change Works

Here’s a high-level flow:

Pre-create Index:

SQL
ALTER TABLE t ADD INDEX (idx_col) WITH PARSER parser_name;

Data Migration:

Start a background worker thread to insert data into the new index.
The main thread handles ongoing queries and writes, ensuring minimal disruption.

Finalize Index Creation:

SQL
ALTER TABLE t ADD INDEX (idx_col);

This approach ensures that the replica can continue processing binary logs without being stalled by the DDL operation on the master.

#### Closing Summary

In summary, adding a blocking index to the master can stall all replicas due to write operations and binary log accumulation. Tools like pt-online-schema-change address this issue by performing schema changes in an asynchronous manner that minimizes impact on the replica nodes. They ensure data integrity while allowing writes to continue, thus maintaining eventual consistency.

#### Common Mistakes
and Follow-Ups
Overlooking Blocking Operations: Developers might overlook that DDLs can block critical operations.
Ignoring Replication Lag: The delay in applying binary logs can lead to significant replication lag, impacting application availability.
Inadequate Schema Management Tools: Using manual methods for schema changes without considering the impact on replicas.

For further discussion, one could explore how different database systems handle DDLs and whether there are alternative tools or strategies that can be employed.

