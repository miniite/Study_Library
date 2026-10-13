---
type: "practice"
difficulty: "Intermediate"
---

# How would you find the top 10 slowest queries hitting your production database right now?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Partitioning and Partition Pruning](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Partitioning_and_Partition_Pruning.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To identify the top 10 slowest queries hitting a production database, we need to gather and analyze query performance metrics. The primary steps involve setting up a monitoring system that captures query execution times and potentially other relevant metrics (like CPU usage, memory consumption, etc.). We'll focus on breaking down each step from first principles.

#### Triage

The question is essentially asking how to identify which SQL queries are causing the most significant delays in your database. The key areas being probed include understanding logging mechanisms, performance analysis tools, and possibly setting up alerts or dashboards for monitoring query performance.

#### Assumptions
We have access to a production environment with detailed logging enabled.
The database system supports query performance metrics (e.g., execution time).
There is a way to tag or categorize queries if necessary.
We can use available tools like SQL Server Profiler, pgBadger for PostgreSQL, or other APM tools.

#### First-Principles Solution

Capture Query Execution Time:

Ensure the database system logs query execution times and possibly additional metrics such as wait times, CPU usage, memory consumption.

Aggregate Metrics:

Collect all relevant data over a certain period (e.g., last 24 hours) to get an overview of recent performance issues.

Filter and Sort Queries:

Use a query to filter out the slowest queries based on execution time. For example, in PostgreSQL, you can use EXPLAIN or pg_stat_statements view.

Alerts and Dashboards:

Set up alerts for queries that exceed certain thresholds.
Create dashboards to visualize top 10 slowest queries over time.
Detailed Implementation

Let's assume we are using PostgreSQL, which has built-in tools like pg_stat_statements to track query performance. Here’s how you could implement this:

SQL
-- Enable pg_stat_statements if not already enabled
ALTER SYSTEM SET stat_statements TO ON;
RELOAD;

-- View top 10 slowest queries based on total time (sum of execution times)
SELECT
    query,
    calls,
    total_time / calls AS avg_time_per_call,
    total_time AS total_time
FROM
    pg_stat_statements
ORDER BY
    total_time DESC
LIMIT 10;

#### Closing Summary
+ Common Mistakes/Follow-Ups

To summarize, the key steps are enabling query logging, capturing metrics, filtering and sorting queries by execution time, and setting up monitoring tools. Common mistakes include not enabling logging in production environments or not having a proper dashboard to visualize the data.

For follow-ups:

#### Cons
ider implementing a real-time alerting mechanism for critical queries.
Explore other performance optimizations based on the identified slow queries.
Ensure that query performance is regularly monitored even after identifying initial issues.

**Comparison:**

| Query | Calls | Avg Time Per Call | Total Time |
|-------|-------|------------------|------------|
| SELECT ...                              | 2345  | 0.876            | 2090       |
| SELECT ...                              | 1234  | 1.25             | 1534       |

**Code:**

```sql
-- Example SQL query to find top 10 slowest queries in PostgreSQL
ALTER SYSTEM SET stat_statements TO ON;
RELOAD;

SELECT
    query,
    calls,
    total_time / calls AS avg_time_per_call,
    total_time AS total_time
FROM
    pg_stat_statements
ORDER BY
    total_time DESC
LIMIT 10;
```

**Example input / output:**

```text
Input:
SELECT query, calls, total_time / calls AS avg_time_per_call, total_time AS total_time
FROM pg_stat_statements
ORDER BY total_time DESC
LIMIT 10;

Output:
+----------------------------------------+-------+-----------------+-----------+
| query                                  | calls | avg_time_per_call| total_time|
+----------------------------------------+-------+-----------------+-----------+
| SELECT ...                             | 2345  | 0.876           | 2090      |
| SELECT ...                             | 1234  | 1.25            | 1534      |
... (Top 10 slowest queries)
```

**Data:**

```json
{
  "query": "SELECT ...,",
  "calls": 2345,
  "avg_time_per_call": 0.876,
  "total_time": 2090
}
```

**Flow:**

1. Enable pg_stat_statements
2. Collect Metrics
3. Filter and Sort Queries
4. Alerts and Dashboards
5. Implement Optimizations
6. Continuous Monitoring

