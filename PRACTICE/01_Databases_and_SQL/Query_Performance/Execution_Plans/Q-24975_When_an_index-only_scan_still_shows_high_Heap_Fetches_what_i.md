---
type: "practice"
difficulty: "Deep"
---

# When an index-only scan still shows high 'Heap Fetches', what is happening and how do you reduce them?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Query Rewrites and Plan Stability](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Query_Rewrites_and_Plan_Stability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When an index-only scan (IOS) still displays a significant number of "Heap Fetches," it indicates that even though the required data can be retrieved directly from the index, the query optimizer is fetching additional tuples from the heap storage. This scenario often points to suboptimal execution plans or issues with the way indexes are being utilized. To understand and resolve this issue, we need to break down the problem step-by-step.

First, let's understand what "Heap Fetches" means in PostgreSQL (the database system where this concept is most commonly discussed). Heap fetches occur when the query optimizer decides that it needs more data from the heap storage than can be provided by the index alone. This could happen due to several reasons such as missing or less selective indexes, poor query optimization, or misconfigurations.

To effectively tackle this problem, we will follow these steps:

Analyze the Execution Plan: Use EXPLAIN ANALYZE to get detailed information about the current execution plan.
Identify Missing or Unnecessary Indexes: Check if there are missing indexes that could be added to improve performance.
Optimize Index Usage: Ensure that existing indexes are being used efficiently and that no unnecessary scans are happening.
Check for Data Distribution and Selectivity: Verify the distribution of data in the database and how selective your queries are.

Let's dive into each step in detail:

Step 1: Analyze the Execution Plan

First, we need to inspect the execution plan with EXPLAIN ANALYZE. This will give us a detailed breakdown of what the query optimizer is doing. For example:

SQL
EXPLAIN ANALYZE SELECT * FROM my_table WHERE column_a = 'value';

This command will show us whether an index-only scan was performed and if heap fetches were necessary.

Step 2: Identify Missing or Unnecessary Indexes

If the execution plan shows that a full table scan is being performed despite having an index, it might be due to missing indexes. We can use tools like pg_stat_user_tables and pg_index to identify such issues:

SQL
SELECT indexname, indisunique, indkey::intarray
FROM pg_indexes WHERE tablename = 'my_table';

We should also check the statistics gathered by PostgreSQL using pg_stats. This helps us understand how selective our indexes are. Poorly selective indexes can lead to the optimizer choosing a full table scan.

Step 3: Optimize Index Usage

To ensure that index-only scans are being used effectively, we need to make sure that the query patterns align well with the existing indexes. If necessary, consider creating additional or more specific indexes based on common query patterns:

SQL
CREATE INDEX idx_my_table_column_a ON my_table(column_a);

Additionally, we can use hints like LEADING in the EXPLAIN statement to guide the optimizer:

SQL
EXPLAIN (ANALYZE, FORMAT JSON)
SELECT * FROM my_table WHERE column_a = 'value' LEADING (column_a);
Step 4: Check for Data Distribution and Selectivity

Lastly, we need to ensure that our data is distributed in a way that supports efficient indexing. High cardinality columns are typically better candidates for indexes than low-cardinality ones. We can use the ANALYZE command to update statistics on our tables:

SQL
ANALYZE my_table;

#### Closing Summary
and Common Mistakes

Reducing heap fetches involves a combination of understanding query patterns, optimizing index usage, and ensuring that data is well-distributed. Some common mistakes include ignoring the importance of statistics updates, over-indexing leading to increased maintenance overhead, and not considering the trade-offs between different types of indexes.

In conclusion, by systematically analyzing execution plans, identifying missing or redundant indexes, optimizing index usage, and ensuring proper data distribution, we can significantly reduce heap fetches and improve query performance.

#### Common mistakes
include:

Not updating statistics on tables
Over-indexing leading to increased maintenance costs
Ignoring the selectivity of indexes

**Comparison:**

| indexname | indisunique | indkey |
|-----------|------------|--------|
| idx_my_table_column_a | false | {101} |
| table_name | column_name | n_distinct | most_common_vals | most_common_freqs |
|------------|-------------|------------|------------------|--------------------|
| my_table   | column_a    | 50         | value1, value2   | 40%, 30%           |

**Code:**

```sql
-- Example SQL queries for analysis
EXPLAIN ANALYZE SELECT * FROM my_table WHERE column_a = 'value';
SELECT indexname, indisunique, indkey::intarray
FROM pg_indexes WHERE tablename = 'my_table';
ANALYZE my_table;
```

**Example input / output:**

```text
EXPLAIN ANALYZE SELECT * FROM my_table WHERE column_a = 'value';
-- Output:
-- Nested Loop (cost=0.56..21.57 rows=1 width=89)
--   -> Index Only Scan using idx_my_table_column_a on my_table (cost=0.28..14.04 rows=1 width=89)
--         Index Cond: (column_a = 'value'::text)
--   -> Bitmap Heap Scan on my_table (cost=3.59..7.64 rows=1 width=89)
--        Recheck Cond: (column_a = 'value'::text)
--        -> Bitmap Index Scan on idx_my_table_column_a (cost=0.00..3.59 rows=1 width=0)
--              Index Cond: (column_a = 'value'::text)

-- Worked example:
-- Heap Fetches: 2
```

**Data:**

```json
{
  "pg_indexes": [
    {"indexname": "idx_my_table_column_a", "indisunique": false, "indkey": [101]}
  ],
  "pg_stats": {
    "table_name": "my_table",
    "column_name": "column_a",
    "n_distinct": 50,
    "most_common_vals": ["value1", "value2"],
    "most_common_freqs": [0.4, 0.3]
  }
}
```

**Flow:**

1. Identify Execution Plan
2. Analyze with EXPLAINANALYZE
3. Check for Missing orUnnecessary Indexes
4. Create Additional Indexes ifNecessary
5. Optimize Query Patterns UsingHints
6. Update Statistics on Tables
7. Re-run Analysis and ValidateImprovements
8. Repeat if Needed

