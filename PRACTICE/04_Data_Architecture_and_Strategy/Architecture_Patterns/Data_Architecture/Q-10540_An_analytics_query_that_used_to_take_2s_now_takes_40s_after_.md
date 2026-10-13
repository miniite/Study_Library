---
type: "practice"
difficulty: "Deep"
---

# An analytics query that used to take 2s now takes 40s after your lake grew 10x. Walk me through diagnosing whether the fix is partitioning, compaction, file sizing, clustering, or an engine change — and how you prevent regression at scale.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Small Files and Compaction](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Small_Files_and_Compaction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Diagnosing Performance Degradation in a Data Warehouse

When faced with a sudden performance degradation from 2 seconds to 40 seconds after scaling up your data lake by 10x, diagnosing the root cause requires a deep understanding of various data architecture principles and techniques. This problem can be approached through several key steps: profiling, analysis, and optimization. Here's how you would systematically diagnose whether the issue lies in partitioning, compaction, file sizing, clustering, or an engine change.

#### Triage

The question is probing your ability to analyze complex data storage and processing systems, identify bottlenecks, and implement efficient optimizations at scale. Key assumptions include:

The query has not changed significantly.
The performance issue started after the data lake was scaled up by 10x.
You are using a modern data warehouse system (e.g., Apache Iceberg, Delta Lake).
Step-by-Step Solution
1. Profiling and Analysis

The first step is to gather detailed profiling information about the query execution. Use tools like EXPLAIN plans, SQL tracebacks, and performance monitoring dashboards provided by your data warehousing platform.

#### Example
Using Apache Iceberg’s Explain Plan

SQL
EXPLAIN EXTENDED SELECT * FROM my_table;
2. Identify the Bottleneck

Based on profiling data, identify where the query is spending most of its time. Common bottlenecks include:

I/O operations (read/write latency)
CPU and memory usage
Network latency

For instance, if the EXPLAIN plan shows that a large fraction of time is spent on reading files or scanning data, it might point towards file-level issues.

3. Evaluate Partitioning

Partitioning can significantly improve query performance by reducing the amount of data scanned. Check if your table uses partitioning and how effective it is:

#### Example
Checking Partitioning

SQL
SHOW PARTITIONS FROM my_table;

If no partitions are used or if they are not fine-grained enough, consider adding more partitions based on relevant columns.

4. Assess Compaction

Compaction can improve data locality by merging smaller files into larger ones, reducing the number of reads required for a query. Check the compaction settings and logs:

#### Example
Checking Compaction Status

SQL
SHOW COMPACTIONS FROM my_table;

If compaction is not enabled or configured properly, enabling it might help.

5. File Sizing

Large files can lead to poor performance due to excessive shuffle operations. Optimal file size depends on the data processing framework and query patterns:

#### Example
Setting File Size

SQL
ALTER TABLE my_table SET FILE_SIZE = 100MB;

Monitor the impact of different file sizes using A/B testing.

6. Cluster Columns

Clustering can improve join performance by grouping related rows together, reducing the number of disk seeks required:

#### Example
Clustering a Table

SQL
ALTER TABLE my_table CLUSTER BY (customer_id);

Test the effect of clustering on query time and overall system load.

7. Engine Change

If all else fails, consider changing to a more efficient engine or optimizing existing engines through configuration changes or upgrades.

#### Example
Migrating to a More Efficient Engine

SQL
CREATE TABLE new_table LIKE my_table;
COPY INTO new_table FROM my_table;
Preventing Regression at Scale

To prevent future performance regressions:

Automated Profiling: Implement automated profiling tools that run periodically and alert you to potential issues.
Change Management: Use a rigorous change management process for any schema or engine changes, including thorough testing in staging environments.
Capacity Planning: Regularly update capacity plans based on actual usage patterns and performance data.

#### Closing Summary

By systematically analyzing the query execution, identifying bottlenecks, and applying optimizations such as partitioning, compaction, file sizing, clustering, or a change to the engine, you can address the root cause of performance degradation. Preventing future regressions involves continuous monitoring, automated profiling, and robust change management practices.

#### Common Mistakes
Focusing on one optimization without understanding the full context.
Ignoring the data distribution and query patterns when applying optimizations.
Overcomplicating solutions without validating their impact.

**Comparison:**

| Partitioning Strategy | Effect on Query Performance |
|----------------------|------------------------------|
| Coarse-grained       | Slower queries               |
| Fine-grained         | Faster queries               |

**Code:**

```sql
-- Example SQL for profiling and diagnostics
EXPLAIN EXTENDED SELECT * FROM my_table;
SHOW PARTITIONS FROM my_table;
SHOW COMPACTIONS FROM my_table;
ALTER TABLE my_table CLUSTER BY (customer_id);
```

**Example input / output:**

```text
Input:
- EXPLAIN EXTENDED SELECT * FROM my_table;

Output:
+----------------------------------------------------------------+
| id | statement | type       | table           | partitions      |
+----------------------------------------------------------------+
| 1  | SELECT ...| SIMPLE     | my_table        | customer_id=1   |
+----------------------------------------------------------------+

Input:
- SHOW PARTITIONS FROM my_table;

Output:
+------------+
| partition  |
+------------+
| customer_1 |
| customer_2 |
+------------+

Input:
- SHOW COMPACTIONS FROM my_table;

Output:
+------------------------+-------------+
| compaction_id          | status      |
+------------------------+-------------+
| 12345                  | completed   |
+------------------------+-------------+
```

**Data:**

```json
{
  "table": {
    "name": "my_table",
    "schema": [
      {"column": "customer_id", "type": "integer"},
      {"column": "transaction_time", "type": "timestamp"}
    ]
  },
  "partitioning": ["customer_id"],
  "compaction_status": "Enabled"
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Profiling and Diagnostics
6. Identify Bottleneck
7. Partitioning?
8. Optimize Partitioning
9. Compaction Enabled?
10. Migrate to Efficient Engine
11. Adjust File Sizing

