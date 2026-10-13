---
type: "practice"
difficulty: "Expert"
---

# Under high write load your PostgreSQL is slowing down. How do you diagnose the root cause?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Finding Expensive Queries](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Finding_Expensive_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a PostgreSQL database that's slowing down under high write load, the first step is to understand the current state of the system and identify the bottlenecks. This involves a systematic approach to diagnose the root cause, leveraging both tools and in-depth knowledge of PostgreSQL internals.

Triaging the Problem

The question essentially asks how we can systematically diagnose why a PostgreSQL database is slowing down under high write load. The key aspects the interviewer will be probing include understanding the performance metrics, identifying specific bottlenecks, analyzing query patterns, and considering configuration settings and resource limitations.

#### Assumptions

You have access to the PostgreSQL instance.
You are familiar with monitoring tools like pg_stat_statements, EXPLAIN output, and system monitoring tools such as top or htop.
Step-by-Step Solution

Check System Resource Utilization:

Use system-level monitoring tools (e.g., top, htop) to check CPU, memory, and disk usage.
Monitor the PostgreSQL backend processes with tools like ps aux | grep postgres to see if any queries are consuming excessive resources.

Analyze Performance Metrics:

Use pg_stat_activity to monitor active connections and their states.
Examine pg_stat_statements for slow query analysis, focusing on the most time-consuming statements.

Review Query Performance:

Analyze individual queries using the EXPLAIN command to understand their execution plans.
Look for inefficient query patterns or missing indexes that could be causing delays.

Check Disk I/O and WAL Settings:

Use pg_waldump to check the write-ahead log (WAL) settings and ensure they are optimized for your workload.
Monitor disk I/O performance with tools like iostat or iotop.

Tune Configuration Parameters:

Adjust PostgreSQL configuration parameters such as work_mem, maintenance_work_mem, max_wal_size, etc., to better handle the write load.

#### Cons
ider using extensions like pg_repack or pg_partman for managing large tables.

Examine Connection Management:

Ensure connection pooling is properly configured and that connections are not idle for too long.
Check for any deadlocks or locks by examining pg_locks.

Log Analysis:

Review PostgreSQL log files (postgresql.log) for error messages, warnings, or performance-related issues.

Hardware Considerations:

Evaluate the need for additional hardware resources like more RAM, faster disks, or better CPU architecture.

#### Cons
ider sharding if the database schema allows it, to distribute the write load across multiple instances.

#### Closing Summary

By systematically analyzing system and application-level metrics, tuning PostgreSQL configurations, and possibly considering hardware upgrades, we can effectively diagnose and address the root cause of slow performance in a PostgreSQL database under high write load. Common mistakes include not thoroughly understanding query patterns or overlooking system resource limitations.

#### Common Mistakes/Follow-Ups
Focusing solely on indexing without considering other performance bottlenecks.
Ignoring system-level resource constraints such as CPU and memory limits.
Overlooking the importance of monitoring tools like pg_stat_statements for slow query detection.
Not evaluating the need for hardware upgrades or schema optimizations.

**Code:**

```sql
-- Example EXPLAIN output to analyze a problematic query
EXPLAIN SELECT * FROM large_table WHERE column = 'value';
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Monitor System Resources
6. Are there obvious resourcebottlenecks?
7. Analyze Query Performance
8. Tune PostgreSQLConfiguration Parameters
9. Tune Indexes and UseExtensions
10. Review ConnectionManagement
11. Check WAL Settings and DiskI/O
12. Consider Hardware Upgradesor Sharding
13. Implement Adjustments andMonitor Again
14. Problem Solved?
15. K

