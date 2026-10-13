---
type: "practice"
difficulty: "Deep"
---

# You partitioned a Delta table by event_date and hour and now writes take 3x longer than reads warrant — how do you reason about whether the hour sub-partition is worth it?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Concurrent Partition Overwrites](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Concurrent_Partition_Overwrites.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with partitioning and bucketing in a Delta table, particularly when we observe that writes are taking significantly more time compared to reads, it's crucial to step back and understand the underlying mechanics of these operations. In this case, we're looking at a scenario where partitioning by event_date and sub-partitioning by hour has led to increased write times. The key is to reason through the performance implications from first principles.

To determine whether the hour sub-partition is worth it, let's break down the problem:

Understanding Partitioning and Bucketing:

Partitioning splits data based on a partitioning column (in this case, event_date). This allows for faster read operations by filtering out partitions that are irrelevant to the query.
Bucketing further divides the data within each partition using a hash function. The number of buckets can be adjusted to balance write and read performance.

#### Assumptions
and Observations:

We know that writes take 3x longer than reads, which suggests an imbalance in the workload.
event_date is used as the primary partitioning column. This is efficient for filtering by date ranges but might not be enough to distribute write operations evenly across partitions.

Reasoning from First Principles:

When performing a write operation, data needs to be appended to the appropriate sub-partition based on hour.
If writes are slow, it could indicate that some hours have more data than others, leading to hotspots or uneven distribution.
Alternatively, the overhead of writing to multiple buckets for each hour might be causing delays.

Step-by-Step Analysis: a. Check Data Distribution:

SQL
SELECT event_date, hour, COUNT(*) as count
FROM table
GROUP BY event_date, hour;
This query will help us understand if there are significant differences in the number of rows per hour. Hotspots can indicate sub-optimal write performance.

b. Analyze Write Operations:

SQL
EXPLAIN SELECT * FROM table WHERE event_date = '2023-10-01';
Use EXPLAIN to understand the execution plan and identify any bottlenecks in the write process.

c. Bucketing Effectiveness:

If bucketing is enabled, we need to ensure that it's configured optimally.
SQL
DESCRIBE DETAIL table;
Check if the number of buckets per partition is appropriate for the workload. Too many or too few buckets can affect performance.

#### Conclusion
and Next Steps:

If the distribution shows uneven write loads, consider re-balancing data or adjusting the bucketing strategy.
Ensure that the bucketing configuration aligns with the expected write patterns.

#### Closing Summary
and Common Mistakes:

The hour sub-partition might not be necessary if it's causing more overhead than benefit during writes.
Always validate assumptions by checking actual data distribution and performance metrics.

**Code:**

```sql
-- Example SQL queries for analysis
SELECT event_date, hour, COUNT(*) as count
FROM table
GROUP BY event_date, hour;

EXPLAIN SELECT * FROM table WHERE event_date = '2023-10-01';

DESCRIBE DETAIL table;
```

