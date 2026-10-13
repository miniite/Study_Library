---
type: "practice"
difficulty: "Deep"
---

# Prod latency spiked at 2am and slow_log shows a query that was fast yesterday now doing a full scan — walk me through how you'd root-cause a plan flip in InnoDB.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ MySQL Performance Diagnosis](<../../../../TOOLS/MySQL/MySQL_Platform/MySQL_Performance_Diagnosis.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a query that used to be fast suddenly starts performing poorly and causes latency spikes, it’s crucial to understand what might have changed between its previous execution and the current one. A common culprit is a plan flip in InnoDB, where a more efficient execution plan was replaced with a less optimal one. To root-cause this issue, we need to analyze various aspects of the query's behavior, environment, and MySQL’s internal workings.

#### Triage
Understanding the Question

The core question revolves around identifying why a previously optimized query now triggers a full table scan (a Plan Flip), which is indicated by spikes in production latency. The focus should be on understanding the factors that might have caused this change in execution plan, such as changes in data distribution or schema modifications.

#### Assumptions
We are dealing with a MySQL database using InnoDB storage engine.
The slow_log provides detailed information about the query and its execution plan.
There were no recent schema changes or updates to the application code that could have impacted the query's performance.
Step-by-Step Solution
1. Review the Query Log

Start by examining the slow_log entry for this specific query. Look at the following details:

The exact SQL statement.
Any conditions, joins, and indexes used in the query.
The full execution plan provided by MySQL (which can be found using EXPLAIN).
2. Check Data Distribution

Evaluate whether there have been significant changes in data distribution that could impact the query's performance. This includes:

Are there new records being inserted or updated frequently?
Has there been a large influx of data, causing the table to grow significantly?
3. Analyze Index Usage

Determine if the indexes used by the query are still effective. Consider the following questions:

Is the index that was previously used no longer covering the needed columns?
Have any new records or data types been added that might make existing indexes less efficient?
4. Check for Lock Contention

InnoDB uses row-level locking, and excessive locking can degrade performance. Analyze whether there is a high level of lock contention:

Are there more transactions running concurrently than before?
Is the query holding locks longer than usual?
5. Monitor MySQL Configuration Parameters

Review relevant MySQL configuration parameters that could influence plan selection:

innodb_buffer_pool_size: Ensure enough buffer pool size to cache frequently accessed data.
query_cache_size: If enabled, check if it’s effectively caching results.
optimizer_switch and other optimization-related settings.
6. Use Performance Schema

Utilize the Performance Schema (if enabled) for more detailed insights:

Monitor events_waits_summary_by_event_name to see which waits are frequent.
Check file_iops_global_by_event if there’s a lot of I/O operations.

#### Closing Summary
and Common Mistakes

In summary, a plan flip can be caused by changes in data distribution, index usage, schema modifications, or configuration settings. To diagnose the issue effectively:

Start with detailed analysis from the slow_log.
Verify data distribution and index effectiveness.
Monitor for lock contention and transactional overhead.
Adjust MySQL configurations as necessary.

#### Common mistakes
include not thoroughly reviewing the query’s execution plan or assuming that recent changes are unrelated to performance issues. Always consider all possible factors before concluding the root cause.

**Comparison:**

```
CUSTOMERS
first_name
VARCHAR(50),
last_name
VARCHAR(50),
email
VARCHAR(100)
```

**Code:**

```sql
-- Example of a slow_log entry
SELECT * FROM customers WHERE last_name = 'Smith';
```

**Example input / output:**

| Query | Old Plan (Fast)                     | New Plan (Slow)                       |
|-------|------------------------------------|--------------------------------------|
| EXPLAIN SELECT * FROM customers WHERE last_name = 'Smith' | Using index; Range checked 2 rows; Using where; Found 1 row; Using filesort |
| EXPLAIN SELECT * FROM customers WHERE last_name = 'Smith' | Full table scan; Checking 10,000 rows; Using temporary; Using filesort          |

**Data:**

```plaintext
-- Sample data schema
CREATE TABLE customers (
    id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100)
);
```

**Flow:**

1. digraph QueryPlanFlips {
2. rankdir=LR;
3. subgraph cluster_0 {
4. label="Plan Flipping Process";
5. slow_log -> analyze_query -> check_data_distribution;
6. slow_log -> analyze_index_usage;
7. analyze_query -> review_optimizer_switches;
8. analyze_index_usage -> monitor_lock_contention;
9. check_data_distribution -> adjust_buffer_pool_size;
10. monitor_lock_contention -> optimize_concurrency;
11. }
12. }

