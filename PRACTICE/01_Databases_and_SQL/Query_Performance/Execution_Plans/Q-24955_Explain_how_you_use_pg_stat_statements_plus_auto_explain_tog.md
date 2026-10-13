---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you use pg_stat_statements plus auto_explain together to catch and diagnose intermittent slow plans in production.

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Diagnosing Query Regressions](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Diagnosing_Query_Regressions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively identify and diagnose intermittent slow query performance issues in a PostgreSQL database, leveraging pg_stat_statements combined with auto_explain can be extremely powerful. This approach helps by providing detailed execution plans and runtime statistics for queries that run infrequently but significantly impact the system's performance.

#### Triaging the Question

The core of this question revolves around using these two tools together to catch and diagnose intermittent slow query issues in a production environment. The key aspects being probed include:

Understanding how pg_stat_statements and auto_explain function.
Knowledge on when and why these tools are useful, especially for complex or infrequent queries.
Ability to interpret the data from both tools effectively.

#### Assumptions
You have a basic understanding of PostgreSQL’s internal query execution process.
The database is under regular monitoring with logging enabled.
You have administrative access to configure and manage pg_stat_statements and auto_explain.

#### First-Principles Solution
Step 1: Enabling pg_stat_statements

First, enable pg_stat_statements in your PostgreSQL configuration:

SQL
# Edit postgresql.conf
shared_preload_libraries = 'pg_stat_statements'
default_statistics_target = 1000 # Adjust based on your query patterns

# Restart the database if necessary

This module will gather detailed statistics about each executed statement, including execution time and resources used. However, pg_stat_statements alone might miss intermittent slow queries that don't run often enough to get enough samples.

Step 2: Configuring auto_explain

Next, configure auto_explain to log the query plans of certain statements:

SQL
# Edit postgresql.conf
auto_explain.log_min_duration = 500 # Log queries slower than this duration in milliseconds
auto_explain.log_format = 'text'
auto_explain.log_location = 'auto_explain.log'
auto_explain.sample_rate = 1.0 # Collect a full sample of each query

This configuration ensures that PostgreSQL will log the execution plan for any query running longer than half a second, providing insights into why these queries are slow.

Step 3: Analyzing Data with pg_stat_statements

Review the data collected by pg_stat_statements to identify queries that execute infrequently but run slowly. Focus on high CPU or I/O usage:

SQL
SELECT query, calls, total_time, rows_returned, plan_cache_hit_ratio
FROM pg_stat_statements
WHERE total_time > 1000 AND (calls < 10 OR plan_cache_hit_ratio < 95);
Step 4: Interpreting auto_explain Logs

Inspect the auto_explain.log to get detailed execution plans for these queries. Look for inefficiencies in the query plan, such as:

Poor index usage
Full table scans instead of indexed searches
Misplaced subqueries or joins
Step 5: Optimizing Queries and Indexes

Based on the findings from steps 3 and 4, identify opportunities to optimize:

Add missing indexes.
Refactor queries to use more efficient join patterns.

#### Cons
ider using EXPLAIN ANALYZE in a development environment before making changes.

#### Closing Summary

By combining pg_stat_statements and auto_explain, you can effectively catch and diagnose intermittent slow query issues, ensuring that your database remains performant even under varying workloads. This approach requires careful monitoring and configuration but offers significant benefits for performance tuning.

#### Common Mistakes/Follow-Ups
Failing to set appropriate thresholds in pg_stat_statements and auto_explain, leading to too much or too little data.
Not analyzing the collected data thoroughly, resulting in missed optimization opportunities.
Overlooking the need for regular maintenance of indexes and query structures.

**Comparison:**

| Column       | Type    |
|-------------|---------|
| query       | text     |
| calls      | integer   |
| total_time  | interval |
| rows_returned | integer  |
| plan_cache_hit_ratio | real    |

**Code:**

```sql
-- Enabling pg_stat_statements
shared_preload_libraries = 'pg_stat_statements'
default_statistics_target = 1000

-- Configuring auto_explain
auto_explain.log_min_duration = 500
auto_explain.log_format = 'text'
auto_explain.log_location = 'auto_explain.log'
auto_explain.sample_rate = 1.0
```

**Example input / output:**

```text
-- Example output of pg_stat_statements query
query: SELECT * FROM large_table WHERE id = $1;
calls: 3
total_time: 248.5ms
rows_returned: 0
plan_cache_hit_ratio: 67%
```

**Data:**

```sql
-- Sample pg_stat_statements query
SELECT query, calls, total_time, rows_returned, plan_cache_hit_ratio
FROM pg_stat_statements
WHERE total_time > 1000 AND (calls < 10 OR plan_cache_hit_ratio < 95);
```

**Flow:**

1. Rendering diagram…

