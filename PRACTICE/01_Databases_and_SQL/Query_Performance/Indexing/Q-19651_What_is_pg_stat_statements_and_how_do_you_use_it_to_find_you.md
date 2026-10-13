---
type: "practice"
difficulty: "Intermediate"
---

# What is pg_stat_statements and how do you use it to find your worst queries?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Index-only Scans](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index-only_Scans.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

pg_stat_statements is a powerful PostgreSQL extension that tracks execution statistics of all SQL statements executed by a database. By analyzing these stats, you can identify and optimize the performance bottlenecks in your queries. This tool helps backend engineers understand which queries are causing high resource usage or latency, thus enabling targeted optimizations.

#### Triaging the Question

The question asks about pg_stat_statements, an extension used to monitor query execution statistics in PostgreSQL databases. The core issue here is identifying and optimizing the worst-performing queries. Let's break down the assumptions and steps required:

#### Assumptions

You have a PostgreSQL database with pg_stat_statements enabled.
Your application logs many SQL statements, making it necessary to analyze query performance.

Key Objectives:

Identify inefficient queries that consume significant resources or cause high latency.
Optimize these queries for better performance without changing the underlying logic.

#### First-Principles Solution
Step 1: Enable pg_stat_statements

First, ensure you have the pg_stat_statements extension installed and enabled in your PostgreSQL database. You can install it via:

SQL
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

Once enabled, it starts collecting statistics on all SQL statements executed.

Step 2: Configure pg_stat_statements

Adjust the configuration settings to control which queries are tracked. For instance, you might want to limit the number of statements or set a minimum execution count:

SQL
-- Set the maximum number of distinct query texts to track
ALTER SYSTEM SET pg_stat_statements.max = '1000';

-- Set the minimum number of executions before tracking starts
ALTER SYSTEM SET pg_stat_statements.track = 'all';

After setting these parameters, restart your PostgreSQL service for changes to take effect.

Step 3: Query pg_stat_statements

Use the pg_stat_statements view to retrieve statistics:

SQL
SELECT * FROM pg_stat_statements LIMIT 10;

This command returns a list of recent queries with their execution counts, duration, and other relevant metrics. You can further filter this output using conditions like execution time or resource usage.

Step 4: Analyze Query Performance

Review the query results to identify poorly performing statements. Look for high calls, long total_time, or large shared_hit_ratio values as indicators of potential bottlenecks.

For example, a query with high calls and total_time might indicate frequent execution causing significant overhead:

SQL
SELECT sql_txt AS "Query", calls AS "Calls", total_time/1000::float8 AS "Total Time (s)", shared_hit_ratio
FROM pg_stat_statements
WHERE total_time > 5000 AND shared_hit_ratio < 0.9;
Step 5: Optimize Identified Queries

Based on the analysis, identify specific queries for optimization. Common strategies include:

Indexing: Ensure proper indexing to speed up data retrieval.
Query Refactoring: Modify the query structure or use different joins, subqueries, or CTEs (Common Table Expressions) to improve performance.
Partitioning Tables: Split large tables into smaller chunks based on certain criteria.

#### Closing Summary
and Common Mistakes

Using pg_stat_statements effectively can significantly enhance database performance by identifying and optimizing critical queries. However, here are some common mistakes:

Overlooking Low-Cost Queries: Ensure you filter out low-cost queries to focus only on those with significant overhead.
Misinterpretation of Metrics: Understand metrics like shared_hit_ratio, which indicates cache hit rates; a lower ratio might not always mean poor performance if the query is infrequent.
Ignoring Schema Changes: After optimizing, make sure to monitor whether the changes have had the desired effect and adjust further as needed.
Code Blocks

**Flow:**

1. ### Self-Check
2. This answer covers the core concepts, steps, and potential pitfalls of using `pg_stat_statements`. It is both comprehensive and practical for intermediate-level engineers.

