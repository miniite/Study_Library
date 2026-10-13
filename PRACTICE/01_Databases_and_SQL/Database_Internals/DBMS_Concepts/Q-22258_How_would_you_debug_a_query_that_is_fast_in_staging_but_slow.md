---
type: "practice"
difficulty: "Deep"
---

# How would you debug a query that is fast in staging but slow in production with the same schema?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1 · [↑ Partitioning and Partition Pruning](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Partitioning_and_Partition_Pruning.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When debugging a query that performs well in staging but poorly in production, it's crucial to systematically identify and address potential differences between these environments. The key steps involve comparing execution plans, resource utilization, data distribution, and potential performance overheads specific to the production environment.

First, triage the issue by understanding what the problem is: a query that works efficiently in staging but becomes slow or inefficient in production could be due to several factors such as differing execution environments (hardware, software versions), data skew, caching effects, or optimization settings. The core idea here is to methodically narrow down the causes.

Next, analyze the execution plans for both environments. Use tools like EXPLAIN statements to get detailed insights into how each environment optimizes and executes the query. Compare the cost estimates and actual row counts in the execution plans; look for differences in join order, index usage, or subquery evaluation strategies that might be affecting performance.

#### Cons
ider data distribution and skew: In production, the data may have different distributions than in staging, leading to skewed queries or poor partitioning choices. Analyze histograms, statistics, and any relevant metadata to understand how data is spread across tables and indexes.

Evaluate resource utilization: Check CPU usage, memory consumption, disk I/O, and network latency differences between environments. Use performance monitoring tools like top, iostat, netstat for Linux or similar utilities on Windows to gather detailed metrics.

Look into caching effects and optimization settings: Staging and production might have different caching behaviors due to varying cache sizes or policies, which can significantly impact query performance. Additionally, production systems often use more aggressive optimization settings like full table scans or parallelism that may not be present in the staging environment.

Finally, test and iterate: Implement changes based on your analysis and re-run the queries to observe improvements. This might involve adjusting indexes, rewriting queries, changing execution plans, or tuning database parameters.

In summary, debugging a slow query involves a comprehensive approach: comparing execution plans, understanding data distribution, monitoring resource utilization, considering caching effects, and iteratively testing adjustments.

**Code:**

```sql
-- Example EXPLAIN output comparison
EXPLAIN SELECT * FROM large_table WHERE column1 = 'value';
```

