---
type: "practice"
difficulty: "Expert"
---

# Explain how MVCC garbage / vacuum works in Postgres and why heap bloat happens. A table's disk usage keeps growing despite steady row count — walk through diagnosing whether autovacuum is falling behind and how to fix it.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ B-tree Concurrency and Latches](<../../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/B-tree_Concurrency_and_Latches.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand MVCC garbage / vacuum in PostgreSQL and heap bloat, let’s start by briefly summarizing what these concepts are about. MVCC (Multi-Version Concurrency Control) is a technique that allows multiple versions of rows to exist simultaneously in the database, enabling concurrent transactions without locks. Heap bloat refers to the phenomenon where disk usage for a table increases significantly despite maintaining a steady row count due to unused space from deleted or updated rows.

Now, let’s dive into the specifics. MVCC works by maintaining a version history of each row. Each time a row is modified, PostgreSQL creates a new tuple with the current state and marks the old one as "dead." This allows readers to see the most recent version while writers can apply changes without interfering with read operations.

#### Triaging the Question

The core issue here is diagnosing whether autovacuum (the process that manages MVCC garbage collection) is falling behind, leading to heap bloat. We need to understand why this happens and how to identify it, as well as steps to mitigate it. The primary assumptions are:

The database uses MVCC.
There is a steady state of inserts, updates, and deletes on the table.
Step-by-Step Solution
1. Understanding Heap Bloat

Heap bloat occurs because PostgreSQL does not automatically shrink or reuse space left by deleted rows. Even if you have a steady row count, the storage size can increase due to leftover dead tuples from previous operations. This is why periodic vacuuming is essential.

2. Identifying Whether Autovacuum Is Falling Behind

To diagnose whether autovacuum is falling behind, we need to check several metrics:

Free Space Map (FSM) Usage: The FSM tracks the availability of free space in the table. You can use pg_freespace or SELECT * FROM pg_freespace('your_table') to see how much space is being wasted.
Dead Tuple Count: Use VACUUM FULL VERBOSE; and observe if there are a significant number of dead tuples left after vacuuming. Alternatively, you can use pgstattuple or SELECT * FROM pgstattuple('your_table') to get detailed statistics.
3. Analyzing Autovacuum Logs

PostgreSQL logs autovacuum activities in the autovacuum.log file located typically in your PostgreSQL data directory. This log helps you understand how often and when autovacuum is running, and if it’s processing all necessary operations.

4. Manual Vacuuming

If you suspect that autovacuum is not keeping up, you can manually run VACUUM or ANALYZE commands to see the impact:

SQL
VACUUM your_table;

This will reclaim space and mark dead tuples for future removal.

5. Configuring Autovacuum

To ensure autovacuum operates efficiently, consider tuning its parameters in postgresql.conf:

autovacuum_vacuum_scale_factor: Adjust this to control how many rows need to be updated before autovacuum triggers.
autovacuum_vacuum_threshold: Set the minimum number of dead tuples before a vacuum is initiated.
autovacuum_freeze_max_age: Ensure that old transactions are frozen to avoid bloat.
6. Periodic Maintenance

Regularly run VACUUM FULL or CLUSTER (if necessary) on large tables to completely remove leftover space and reorder the heap for better performance.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, diagnosing and fixing heap bloat involves understanding MVCC, monitoring FSM usage, analyzing autovacuum logs, running manual vacuum operations, and adjusting autovacuum parameters. Common mistakes include relying solely on VACUUM without considering the need for ANALYZE, neglecting to check autovacuum logs, and not regularly tuning autovacuum settings.

For follow-ups, you might be asked about specific configurations, trade-offs between different vacuum methods (like VACUUM FULL vs. CLUSTER), or how these issues relate to other database management strategies.

**Code:**

```sql
-- Example SQL commands for manual vacuuming and analyzing tuple usage

VACUUM your_table;

SELECT * FROM pg_freespace('your_table');

SELECT * FROM pgstattuple('your_table');
```

