---
type: "practice"
difficulty: "Intermediate"
---

# How would you diagnose why a specific INSERT is slow when the table itself is small, considering trigger overhead, foreign-key checks, and index maintenance, and how do you attribute time to each?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Materialized Views vs Incremental Aggregates](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Materialized_Views_vs_Incremental_Aggregates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose why an INSERT operation on a PostgreSQL table is slow, especially when the table is relatively small, we need to consider several factors including trigger overhead, foreign-key checks, and index maintenance. The key idea here is to systematically isolate these potential bottlenecks using performance profiling tools and analytical queries.

Firstly, let's triage what the question is probing: We are essentially asked to identify the root cause of slow insertions by examining various components that might be contributing to this slowness. This involves understanding how PostgreSQL manages triggers, foreign keys, indexes, and more during an INSERT operation. The goal is not just to find out why it’s slow but also to attribute specific times to each component.

#### Assumptions

We have access to the database logs and performance monitoring tools.
The table in question has relatively few rows, indicating that other factors like network latency or disk I/O are unlikely to be major contributors.

Step-by-step solution:

Check for Triggers:

PostgreSQL triggers can be a significant overhead during an INSERT if they perform complex operations.
Use the pg_trigger and pg_class system tables to find out which triggers are attached to this table.
SQL
SELECT tgname, tgtype
FROM pg_trigger
WHERE tgisinternal = false AND tgrelid = (SELECT oid FROM pg_class WHERE relname = 'your_table_name');

Examine Foreign-Key Checks:

Foreign-key constraints can cause delays during an INSERT if the referenced rows are not available or require additional operations.
Check the pg_constraint table to find out which foreign keys are defined on your table.
SQL
SELECT conname, contype
FROM pg_constraint
WHERE conrelid = (SELECT oid FROM pg_class WHERE relname = 'your_table_name');

Analyze Index Maintenance:

Inserts can be slow if the table has many indexes or if those indexes need to be updated frequently.
Use pg_stat_all_indexes and pg_index to check index usage and statistics.
SQL
SELECT idx_scan, idx_tup_read, idx_tup_fetch
FROM pg_stat_all_indexes WHERE tablename = 'your_table_name';

Profile with EXPLAIN ANALYZE:

Use EXPLAIN ANALYZE to get detailed execution plans and timing information for the INSERT operation.
SQL
EXPLAIN ANALYZE INSERT INTO your_table (column1, column2) VALUES ('value1', 'value2');

Use Performance Monitoring Tools:

Utilize PostgreSQL’s built-in pg_stat_activity and pg_stat_database to monitor ongoing operations.
Use tools like pgBadger, pgTAP, or even pg_stat_statements to analyze query performance over time.

Attributing Time to Each Component:

Once we have profiling data, we can calculate the relative contribution of each component.
SQL
-- Example calculation in pseudo-code
SELECT
  (time_trigger - min(time_total)) AS time_spent_on_triggers,
  (time_fk_check - min(time_total)) AS time_spent_on_fks,
  (time_index - min(time_total)) AS time_spent_on_indexes
FROM (
  SELECT
    EXTRACT(EPOCH FROM (end_time - start_time)) AS time_total,
    EXTRACT(EPOCH FROM COALESCE((SELECT end_time - start_time FROM pg_stat_activity WHERE query LIKE '%your_table_name%' AND state = 'triggering'), 0)) AS time_trigger,
    EXTRACT(EPOCH FROM COALESCE((SELECT end_time - start_time FROM pg_stat_activity WHERE query LIKE '%your_table_name%' AND state = 'waiting on fk constraint')), 0) AS time_fk_check,
    EXTRACT(EPOCH FROM COALESCE((SELECT end_time - start_time FROM pg_stat_activity WHERE query LIKE '%your_table_name%' AND state = 'indexing')), 0) AS time_index
  FROM (SELECT NOW() - interval '1 minute' as start_time, NOW() as end_time)
) subquery;

#### Closing Summary
By systematically examining triggers, foreign-key checks, and index maintenance through PostgreSQL’s system tables, EXPLAIN ANALYZE, performance monitoring tools, and detailed profiling, we can accurately diagnose the cause of slow INSERT operations. This approach helps in not only identifying but also attributing specific times to each contributing factor.

#### Common Mistakes

Focusing solely on hardware issues like disk I/O or CPU usage without considering software-level optimizations.
Not using EXPLAIN ANALYZE effectively to get detailed insights into query execution plans.
Overlooking the importance of PostgreSQL’s built-in performance monitoring tools and external utilities.

**Code:**

```sql
SELECT tgname, tgtype
FROM pg_trigger
WHERE tgisinternal = false AND tgrelid = (SELECT oid FROM pg_class WHERE relname = 'your_table_name');

SELECT conname, contype
FROM pg_constraint
WHERE conrelid = (SELECT oid FROM pg_class WHERE relname = 'your_table_name');

SELECT idx_scan, idx_tup_read, idx_tup_fetch
FROM pg_stat_all_indexes WHERE tablename = 'your_table_name';

EXPLAIN ANALYZE INSERT INTO your_table (column1, column2) VALUES ('value1', 'value2');
```

**Example input / output:**

```text
Example output of EXPLAIN ANALYZE:
INSERT 0 1
Time: 4.386 ms

Detailed timing information:
rows_time
2.570 ms (triggering)
1.595 ms (waiting on fk constraint)
0.217 ms (indexing)
```

