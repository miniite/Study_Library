---
type: "practice"
difficulty: "Deep"
---

# What is the plan cache, how do you find the most expensive cached plans with sys.dm_exec_query_stats, and what does plan cache bloat look like?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Filtered Indexes and Plan Cache](<../../../../TOOLS/SQL_Server/T-SQL/Filtered_Indexes_and_Plan_Cache.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The plan cache in SQL Server stores compiled execution plans for queries. It helps improve performance by reducing the overhead of recompiling a query each time it is run. To find the most expensive cached plans, you can use sys.dm_exec_query_stats, which provides information about the resource usage and statistics for cached plans. Plan cache bloat refers to the situation where too many plans are stored in memory, potentially at the expense of more important ones.

To address this question step-by-step:

What is the Plan Cache?

The plan cache is a critical component of SQL Server’s internal architecture that stores compiled execution plans. These plans are created when a query is first executed and can be reused for subsequent executions with identical parameters or by reference to the query text, thus avoiding the overhead of recompilation.

Finding the Most Expensive Cached Plans

To find the most expensive cached plans, you need to use sys.dm_exec_query_stats. This dynamic management view (DMV) provides a variety of statistics about the execution plans in the plan cache. Specifically, it contains information such as the number of executions (total_worker_time), the time spent waiting for I/O operations (total_logical_reads, total_logical_writes, total_physical_reads), and more.

Here is an example SQL query to find the most expensive cached plans:

SQL
-- Find top 10 expensive queries by total worker time
SELECT
    qs.total_worker_time AS [Total CPU Time (ms)],
    qs.total_elapsed_time AS [Total Elapsed Time (ms)],
    qp.query_plan,
    qs.execution_count,
    st.text AS [SQL Statement]
FROM
    sys.dm_exec_query_stats qs
CROSS APPLY
    sys.dm_exec_sql_text(qs.sql_handle) st
CROSS APPLY
    sys.dm_exec_query_plan(qs.plan_handle) qp
ORDER BY
    qs.total_worker_time DESC
OFFSET 0 ROWS FETCH NEXT 10 ROWS ONLY;
Plan Cache Bloat

Plan cache bloat occurs when the plan cache becomes too large, leading to issues such as reduced memory available for other important operations or increased overhead due to excessive space consumption. This can be detrimental because it might cause the SQL Server instance to spend more time managing the plan cache rather than executing queries.

To identify and manage plan cache bloat:

Check Plan Cache Size: Use sys.dm_os_memory_clerks to get an overview of memory usage by different components, including the plan cache.
Analyze Plan Cache Content: Use sys.dm_exec_cached_plans to understand which plans are taking up space and whether they are still relevant.

Here is a sample query to check the size of the plan cache:

SQL
-- Get total size used by plan cache in KB
SELECT
    SUM(size_in_bytes) AS [Total Plan Cache Size (KB)]
FROM
    sys.dm_os_memory_clerks
WHERE
    name LIKE 'Plan%';

#### Common Mistakes
and Follow-Ups
Misinterpreting sys.dm_exec_query_stats: Sometimes, queries with higher execution counts may not be the most problematic if their average execution time is low.
Overlooking Memory Management: Plan cache bloat can also lead to memory pressure issues, causing SQL Server to perform additional page reads and writes, thus impacting performance.

#### Closing Summary

In summary, understanding and managing the plan cache is crucial for optimizing SQL Server performance. Using sys.dm_exec_query_stats helps identify the most expensive cached plans, while sys.dm_os_memory_clerks provides insights into memory usage. Plan cache bloat can be managed by monitoring both statistics and memory use, ensuring that critical queries remain efficient.

**Comparison:**

| Column        | Description                                          |
|---------------|------------------------------------------------------|
| total_worker_time | Total CPU time used by the query (in milliseconds).  |
| total_elapsed_time | Total elapsed time for the query execution (in milliseconds).  |
| query_plan     | XML representation of the cached plan.                |
| execution_count | Number of times the query has been executed.          |

**Code:**

```sql
-- SQL to find top 10 expensive queries
SELECT
    qs.total_worker_time AS [Total CPU Time (ms)],
    qs.total_elapsed_time AS [Total Elapsed Time (ms)],
    qp.query_plan,
    qs.execution_count,
    st.text AS [SQL Statement]
FROM
    sys.dm_exec_query_stats qs
CROSS APPLY
    sys.dm_exec_sql_text(qs.sql_handle) st
CROSS APPLY
    sys.dm_exec_query_plan(qs.plan_handle) qp
ORDER BY
    qs.total_worker_time DESC
OFFSET 0 ROWS FETCH NEXT 10 ROWS ONLY;

-- SQL to check plan cache size
SELECT
    SUM(size_in_bytes) AS [Total Plan Cache Size (KB)]
FROM
    sys.dm_os_memory_clerks
WHERE
    name LIKE 'Plan%';
```

