---
type: "practice"
difficulty: "Intermediate"
---

# You need to know whether a specific index is worth its maintenance cost; how do you combine pg_stat_user_indexes idx_scan counts with the write volume on the table to make a drop decision?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Materialized Views vs Incremental Aggregates](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Materialized_Views_vs_Incremental_Aggregates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To determine whether a specific index in PostgreSQL is worth its maintenance cost, we need to balance the idx_scan count from pg_stat_user_indexes with the write volume (i.e., DML operations) on the table. The core idea here is that an index which gets frequently scanned but rarely updated can be justified, whereas an infrequently used or heavily written index might not pay off in terms of performance benefits.

The question is probing your understanding of how indexes affect query performance and how to measure this impact against their maintenance costs. You'll need to consider the balance between read and write operations, as well as other factors like table size and selectivity.

First, let's start with some assumptions:

The idx_scan count indicates how often the index is being used for reading data.
The write volume (inserts, updates, deletes) measures the frequency of DML operations on the table.
Step-by-Step Solution

Collect Data: Retrieve the necessary statistics using SQL queries from pg_stat_user_indexes, pg_stats, and possibly pg_locks.

SQL
-- Example query to get idx_scan counts for a specific index
SELECT
    relname AS table_name,
    indexrelname AS index_name,
    idx_scan
FROM
    pg_stat_user_indexes
WHERE
    schemaname = 'public' AND
    indexname = 'your_index_name';

-- Example query to get write volume
SELECT
    relname AS table_name,
    n_tup_ins AS tuples_inserted,
    n_tup_upd AS tuples_updated,
    n_tup_del AS tuples_deleted
FROM
    pg_stat_user_tables;

Calculate Index Benefits: Determine the benefits of using the index by analyzing idx_scan and comparing it to the table size.

Evaluate Maintenance Cost: Estimate the cost associated with maintaining the index, such as space overhead and disk I/O during write operations.

Decision Criteria:

If the index is heavily used for reads (idx_scan > 10,000) but rarely written to (writes < 1% of table size), it may be worth keeping.
Conversely, if the index sees minimal use and frequent writes, consider dropping or optimizing it.

Risk Assessment: Consider potential risks such as lock contention during write operations that might impact performance negatively.

#### Example
Calculation

Let's say we have a table orders with 1 million rows. The index idx_order_customer_id has an idx_scan count of 20,000 and writes to the table are occurring at a rate of 500 tuples per second.

Read Utilization: 20,000 scans / 1,000,000 rows = 2% read utilization
Write Volume: 500 tuples/sec * 86400 sec/day = 43.2M writes/day

Given these numbers:

The index is moderately utilized for reads.
However, the write volume suggests that frequent updates could be a concern.

#### Conclusion
and Common Mistakes

In this case, while the read benefits are significant, the write load might make it less attractive to keep. You should consider dropping the index and re-evaluating after some time or when the write patterns change significantly.

#### Common mistakes
include:

Ignoring write volume entirely.
Failing to balance between high idx_scan counts and frequent writes.
Overlooking other factors like table size, selectivity, and lock contention.

By carefully analyzing both read and write activities, you can make an informed decision about the index's value.

**Comparison:**

| Column | Description                |
|--------|----------------------------|
| table_name | Name of the table           |
| index_name | Name of the index          |
| idx_scan   | Number of scans using this index  |
| Column | Description                         |
|--------|-------------------------------------|
| table_name | Name of the table              |
| tuples_inserted    | Number of rows inserted         |
| tuples_updated     | Number of rows updated          |
| tuples_deleted     | Number of rows deleted          |

**Example input / output:**

```text
-- Example input -> output for idx_scan count
Input:
SELECT
    relname AS table_name,
    indexrelname AS index_name,
    idx_scan
FROM
    pg_stat_user_indexes
WHERE
    schemaname = 'public' AND
    indexname = 'your_index_name';

Output:
+---------------+-----------------+-----------+
| table_name    | index_name      | idx_scan  |
+---------------+-----------------+-----------+
| orders        | idx_order_customer_id | 20000     |
+---------------+-----------------+-----------+

-- Example input -> output for write volume
Input:
SELECT
    relname AS table_name,
    n_tup_ins AS tuples_inserted,
    n_tup_upd AS tuples_updated,
    n_tup_del AS tuples_deleted
FROM
    pg_stat_user_tables;

Output:
+---------------+------------------------+------------------------+--------------------+
| table_name    | tuples_inserted        | tuples_updated         | tuples_deleted     |
+---------------+------------------------+------------------------+--------------------+
| orders        | 432000                 | 58000                  | 1200               |
+---------------+------------------------+------------------------+--------------------+
```

**Data:**

```

-- Sample data for idx_scan count SELECT relname AS table_name, indexrelname AS index_name, idx_scan FROM pg_stat_user_indexes WHERE schemaname = 'public' AND indexname = 'your_index_name';

CODE
Copy

```sql
-- Sample data for write volume
SELECT
    relname AS table_name,
    n_tup_ins AS tuples_inserted,
    n_tup_upd AS tuples_updated,
    n_tup_del AS tuples_deleted
FROM
    pg_stat_user_tables;
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Get idx_scan count
6. idx_scan > 10,000?
7. Evaluate write volume
8. Decision: Keep index if readutilization is high and writesare low
9. idx_scan < 1,000?
10. Drop the index or consideralternative storage options
11. Evaluate other factors liketable size, selectivity, lockcontention

