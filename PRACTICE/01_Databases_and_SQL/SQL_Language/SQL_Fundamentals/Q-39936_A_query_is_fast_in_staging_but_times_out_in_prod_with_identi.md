---
type: "practice"
difficulty: "Intermediate"
---

# A query is fast in staging but times out in prod with identical data volume. What environment differences (stats, config, parameterization, cache warmth) would you check first?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Replica Staleness](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Replica_Staleness.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a query performs well in staging but times out in production despite having similar data volumes, it often points to environmental differences between the two environments. As an intermediate database developer, I would start by checking several key areas such as statistics, configuration settings, parameterization, and cache warmth. Let's break down these steps:

Statistics (Indexes and Statistics Gathering): In staging, indexes might be more up-to-date or properly configured due to less frequent changes in data volume or schema. Production environments often experience different workload patterns that could lead to stale statistics.
Configuration Settings: There are several database configurations that can significantly impact query performance. For example, buffer pool sizes, connection limits, and query optimization settings.
Parameterization: Parameter sniffing is a common issue where the initial execution plan might not be suitable for all subsequent executions of a parameterized query due to different data distributions.
Cache Warmth: Cache hits can greatly influence performance. If staging has more frequent access patterns or different workload profiles, cache warmness could differ between environments.

Let's examine each in detail:

1. Statistics (Indexes and Statistics Gathering)

Staging Environment:

Check if the staging environment runs ANALYZE commands regularly to update statistics.
Verify that indexes are properly maintained or rebuilt as necessary.

Production Environment:

Look for signs of stale statistics by running DBCC SHOW_STATISTICS (for SQL Server) or equivalent commands to see if the query optimizer is using outdated information.
2. Configuration Settings

Staging Environment:

Confirm that buffer pool sizes, connection limits, and other critical settings are tuned appropriately.

Production Environment:

Check these configurations again but also consider production-specific needs like peak hours or high-concurrency scenarios. For example:
SQL
-- SQL Server Example
SELECT * FROM sys.configurations WHERE name IN ('min server memory', 'max server memory', 'cost threshold for parallelism')
3. Parameterization

Staging Environment:

Ensure that parameter sniffing is not an issue by using OPTION (RECOMPILE) or forcing a specific execution plan.

Production Environment:

Use SQL Server’s sys.dm_exec_query_plan to analyze the query plans and look for potential recompilations. Also, consider enabling Query Store if it's not already enabled:
SQL
-- Enable Query Store in SQL Server
ALTER DATABASE YourDatabase SET QUERY_STORE = ON;
4. Cache Warmth

Staging Environment:

Check if queries are executed more frequently during development and testing.

Production Environment:

Monitor cache hit ratios using commands like sys.dm_os_buffer_descriptors in SQL Server:
SQL
SELECT COUNT(*) AS [Total Buffers], SUM(CASE WHEN database_id = DB_ID('YourDatabase') THEN 1 ELSE 0 END) AS [YourDatabase Buffers]
FROM sys.dm_os_buffer_descriptors WHERE database_id > 2 AND database_id < 32767;

#### Closing Summary

By systematically checking these areas, we can identify and address the root causes of performance discrepancies between staging and production. This approach ensures that both environments are properly tuned for similar workloads.

#### Common Mistakes/Follow-Ups
Mistake: Focusing solely on the query code without considering environment differences.

#### Follow-Up
Question: How would you integrate these checks into a continuous integration process to ensure consistent performance across stages?

