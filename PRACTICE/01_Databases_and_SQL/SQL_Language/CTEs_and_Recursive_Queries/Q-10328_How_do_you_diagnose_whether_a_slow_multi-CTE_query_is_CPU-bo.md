---
type: "practice"
difficulty: "Deep"
---

# How do you diagnose whether a slow multi-CTE query is CPU-bound in the recursion or I/O-bound in a downstream join?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Recursive CTEs and Materialization](<../../../../01_Databases_and_SQL/SQL_Language/CTEs_and_Recursive_Queries/Recursive_CTEs_and_Materialization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose whether a slow multi-CTE (Common Table Expression) query is CPU-bound in the recursion or I/O-bound in a downstream join, you need to understand both the nature of recursive CTEs and how they interact with other parts of your database. Let's break down the problem from first principles.

#### Triage
Understanding the Question

The question asks for a method to distinguish between CPU-bound (recursion) and I/O-bound (downstream join) issues in multi-CTE queries. The key here is to identify which part of the query execution process is causing performance bottlenecks, as this will inform how you optimize your query.

Step-by-Step Solution
Analyze the Query Structure: Break down the CTEs and joins involved in your query.
Use EXPLAIN Plan: Run EXPLAIN on your multi-CTE query to understand its execution plan, focusing on the costs of recursive steps vs. join operations.
Monitor Resource Usage: Use system monitors (e.g., pg_stat_activity, vmstat, iostat) to track CPU and I/O usage during query execution.
Test with Simplified Queries: Temporarily remove or modify parts of the CTEs and joins to isolate their impact.
Detailed Analysis

Recursive CTE Execution:

Recursive CTEs can be CPU-intensive if they involve complex logic or large datasets.
The recursion depth can also significantly increase CPU usage, especially with deep recursions or poor base cases.

Downstream Joins:

Downstream joins can be I/O-bound if the involved tables are large and require extensive data retrieval.
Poor indexing on join keys can exacerbate this issue, leading to higher disk I/O operations.
Using EXPLAIN

To diagnose CPU vs. I/O issues:

Run the EXPLAIN Command:

SQL
EXPLAIN ANALYZE WITH cte_1 AS (SELECT ...) , cte_2 AS (SELECT ...) SELECT * FROM cte_1 JOIN cte_2 ON ... ;
Focus on the execution plan costs for recursive steps and join operations.

Identify Costly Operations:

Look for high-cost recursive steps in the recursive section of the output.
Identify if there are significant costs associated with joins, indicating potential I/O bottlenecks.
Resource Monitoring

System Monitor Tools:

SQL
SELECT datname, usename, query_start, state, query FROM pg_stat_activity WHERE datname = 'your_database' ORDER BY state, query_start;
Check CPU usage with top, htop, or vmstat.
Monitor disk I/O with iostat or iotop.

Sample Data:

SQL
SELECT * FROM pg_stat_file('pg_wal') ;
Use this command to check the throughput of your database's write-ahead log, which can indicate I/O bottlenecks.
Testing with Simplified Queries

Simplify Recursive CTEs: Temporarily reduce the complexity or depth of recursive steps.

Isolate Joins: Remove or simplify downstream join conditions to see if performance improves.

#### Closing Summary

By analyzing the execution plan, monitoring resource usage, and testing with simplified queries, you can effectively diagnose whether a slow multi-CTE query is CPU-bound in recursion or I/O-bound in a downstream join. This approach ensures that you target optimizations where they are most needed.

#### Common Mistakes/Follow-Ups
Misinterpreting EXPLAIN Output: Not understanding the true costs of recursive steps vs. joins can lead to misdiagnosis.
Ignoring Full System Monitoring: Relying solely on query execution plans without considering overall system health can miss critical issues.
Overcomplicating Simplification Tests: Failing to isolate and simplify parts of the query effectively can make it harder to pinpoint bottlenecks.

**Comparison:**

```
CTE_1
(Recursive Step)
JOIN
Result
Input Tables
Output Tables
```

**Code:**

```sql
-- Example CTE Query
WITH cte_1 AS (SELECT * FROM large_table WHERE id > 0),
     cte_2 AS (SELECT * FROM small_table JOIN cte_1 ON small_table.id = cte_1.id)
SELECT * FROM cte_2;
```

**Example input / output:**

```text
-- EXPLAIN Output
EXPLAIN ANALYZE WITH cte_1 AS (SELECT * FROM large_table WHERE id > 0),
     cte_2 AS (SELECT * FROM small_table JOIN cte_1 ON small_table.id = cte_1.id)
SELECT * FROM cte_2;

-- Example Output:
"Hash Join  (cost=384.25..6937.69 rows=1000 width=15) (actual time=0.013..0.021 rows=2 loops=1)"
"  Hash Cond: small_table.id = cte_1.id"
"  ->  CTE Scan on cte_1  (cost=0.00..384.00 rows=500 width=10) (actual time=0.002..0.006 rows=2 loops=1)"
"  ->  Hash  (cost=384.00..384.00 rows=500 width=10) (actual time=0.009..0.009 rows=2 loops=1)"
"        Buckets: 1024  Batches: 1  Memory Usage: 9kB"
"        ->  Seq Scan on small_table  (cost=0.00..384.00 rows=500 width=10) (actual time=0.001..0.002 rows=2 loops=1)"
```

**Data:**

```json
{
  "large_table": {
    "columns": ["id", "name"],
    "rows": [{"id": 1, "name": "A"}, {"id": 2, "name": "B"}]
  },
  "small_table": {
    "columns": ["id", "value"],
    "rows": [{"id": 1, "value": 10}, {"id": 2, "value": 20}]
  }
}
```

**Flow:**

1. Run EXPLAIN ANALYZE
2. Monitor Resource Usage
3. Test with Simplified Queries
4. Analyze Results
5. Determine CPU vs I/O Bound

