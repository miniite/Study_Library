---
type: "practice"
difficulty: "Deep"
---

# A long-running reporting transaction is bloating your PostgreSQL tables; how does MVCC cause that and how do you mitigate it?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Lock Holding and Optimistic Concurrency](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Lock_Holding_and_Optimistic_Concurrency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: Multi-Version Concurrency Control (MVCC) in PostgreSQL ensures data consistency during concurrent operations, but this feature can lead to bloating of tables when used by long-running reporting transactions. To mitigate this, we need to understand how MVCC works and apply techniques like VACUUM and optimizing the transaction management.

#### Triaging the Question
The question asks about a specific issue with PostgreSQL, where long-running reporting transactions are causing table bloat. The key concept here is understanding MVCC and its implications for maintaining data consistency in concurrent environments. The interviewer is probing your knowledge of how MVCC works, why it can lead to bloat, and what strategies can be employed to mitigate the issue.

#### First-Principles Solution

Understanding MVCC: In PostgreSQL, every table has multiple versions of rows due to transactions that modify data. Each transaction sees a snapshot of the database at the time it started (read timestamp). This allows for concurrent reads and writes without locking.
Bloating Mechanism: Long-running reporting queries can hold onto many read snapshots over time. These snapshots accumulate as new transactions occur, leading to more versions of rows in the table. Over time, this can significantly increase the size of the table.
Mitigation Strategies:
VACUUM: Regularly running VACUUM can reclaim space by removing old row versions that are no longer needed. However, this is a read-only operation and should be scheduled during off-peak times to avoid locking issues.
Analyze and Vacuum Analyze: Use these commands to analyze the table's statistics and perform vacuuming at the same time, optimizing query performance as well.
Optimizing Queries: Review and optimize reporting queries to minimize the number of row versions created. This might involve adding indexes, rewriting SQL for better performance, or using temporary tables.
Monitoring: Implement monitoring tools to track table growth and identify problematic transactions. Tools like pg_stat_database can provide insights into the bloat level.

#### Closing Summary
In summary, MVCC is a crucial feature in PostgreSQL that ensures consistency during concurrent operations but can lead to bloat if not managed properly, especially with long-running reporting queries. By understanding how MVCC works and applying techniques such as regular VACUUM, optimizing queries, and monitoring table growth, we can mitigate this issue effectively.

#### Common Mistakes/Follow-Ups

Misunderstanding VACUUM: Common mistakes include not scheduling VACUUM at the right time or over-relying on it without optimizing queries.
Ignoring Query Optimization: Optimizing queries to reduce bloat is often overlooked, leading to unnecessary bloat and performance issues.

**Comparison:**

```
Column	Type
id	integer
value	text
SQL
Copy
-- Example table schema
CREATE TABLE my_table (
    id serial PRIMARY KEY,
    value text
);
```

**Code:**

```sql
-- Example of VACUUM command
VACUUM ANALYZE my_table;

-- Monitoring table growth with pg_stat_database
SELECT * FROM pg_stat_database WHERE datname = 'my_db';
```

