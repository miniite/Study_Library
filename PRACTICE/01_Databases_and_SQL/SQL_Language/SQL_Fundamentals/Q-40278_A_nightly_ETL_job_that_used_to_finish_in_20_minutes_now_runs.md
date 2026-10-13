---
type: "practice"
difficulty: "Expert"
---

# A nightly ETL job that used to finish in 20 minutes now runs for 4 hours after a schema change. Walk me through how you would isolate whether the regression is a bad plan, missing statistics, lock contention, or data-volume growth.

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Incremental Rollups](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Incremental_Rollups.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose why an ETL job that used to finish in 20 minutes now takes 4 hours after a schema change, we need to systematically isolate potential issues such as a bad query plan, missing statistics, lock contention, or data-volume growth. The key is to start with first-principles and methodically narrow down the root cause.

First, let's triage the problem by understanding what could be affecting performance:

A bad query plan might result in inefficient execution.
Missing or outdated statistics can lead SQL engines to choose suboptimal plans.
Lock contention could be causing delays, especially if there are high concurrency issues.
Data-volume growth might exacerbate existing bottlenecks.

Next, we'll go through the steps to diagnose and resolve each potential issue:

Query Plan Analysis:

Run EXPLAIN or equivalent command on critical queries in the ETL job to understand their execution plans.
Check if new indexes are being used or if old ones are still in place.
Look for high-cost operations, such as full table scans, which might indicate inefficiency.

Statistics Review:

Verify that statistics for tables and indexes involved in the ETL job are up to date.
Use ANALYZE or equivalent commands to refresh statistics if necessary.
Ensure that histograms and other statistical information are present and accurate, as they guide query optimizers.

Lock Contention Analysis:

Monitor lock wait events using tools like SHOW ENGINE INNODB STATUS; for MySQL.
Check the lock wait graph in monitoring tools like Prometheus or Grafana to see if there are frequent waits during the ETL run.
Look at the transaction isolation level and whether it might be causing unnecessary locks.

Data Volume Growth Impact:

Estimate the amount of data being processed before and after the schema change.
Run performance tests with synthetic data similar in volume to the current job to isolate the impact of increased data volume.

Performance Metrics and Logs:

Collect and analyze performance metrics such as query latency, CPU usage, I/O operations, and memory usage during the ETL process.
Review error logs for any SQL statements that might be failing or timing out.

Parallelism and Concurrency:

Assess if parallel execution settings are appropriate given the current data volume and schema changes.
Tune session variables like innodb_parallel_read_threads and max_connections to ensure efficient use of resources.

Cluster and Hardware Capacity:

Verify that cluster capacity is sufficient for the new workload, considering factors like disk I/O, network bandwidth, and CPU/GPU capabilities.

#### Cons
ider scaling out or up if necessary to handle increased load.

By systematically analyzing these areas, we can identify which factor(s) are contributing to the performance degradation. Let's walk through a hypothetical example to illustrate:

Hypothetical Example

Step 1: Query Plan Analysis

Run EXPLAIN on a critical query:
SQL
EXPLAIN SELECT * FROM large_table WHERE some_column = 'value';
Observe that the plan is doing full table scans instead of using an index. This suggests a bad plan.

Step 2: Statistics Review

Check statistics for large_table:
SQL
SHOW TABLE STATUS LIKE 'large_table';
Notice outdated or missing histograms, leading to suboptimal query plans.
Run ANALYZE TABLE large_table; to update statistics.

Step 3: Lock Contention Analysis

Monitor lock wait events:
SQL
SHOW ENGINE INNODB STATUS;
Identify frequent wait times and adjust transaction isolation levels or use SET TRANSACTION ISOLATION LEVEL READ COMMITTED;.

Step 4: Data Volume Growth Impact

Run tests with synthetic data:
Simulate the ETL job on a subset of increased data.
Compare performance with before the schema change.

By following these steps, we can isolate and address the root cause of the performance regression. Common mistakes include skipping detailed analysis in favor of quick fixes or assuming only one factor is responsible for the degradation.

