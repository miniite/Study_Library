---
type: "practice"
difficulty: "Deep"
---

# A downstream BI dashboard on Databricks SQL got 10x slower after a partitioning change. How do you decide whether to add liquid clustering, revert, or re-model?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Databricks Query Performance](<../../../../TOOLS/Databricks/Databricks_Platform/Databricks_Query_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a 10x slowdown in a downstream BI dashboard on Databricks SQL following a partitioning change, we need to understand the root cause and evaluate potential solutions. The primary goal is to balance performance with maintainability and scalability. Let's break this down from first principles.

Firstly, when dealing with such an issue, we must consider whether the problem lies in how data is partitioned, indexed, or physically stored. The core question is: Are we effectively utilizing Databricks SQL capabilities to optimize query performance? To do this, we need to explore three potential approaches: adding liquid clustering, reverting the change, or re-modeling the database schema.

Triaging the Issue
Understanding the Change: The partitioning change might have introduced a less optimal distribution of data across partitions.
Performance Metrics: We should review metrics such as query latency, number of partitions scanned, and overall resource utilization (CPU, memory) to identify bottlenecks.
Dashboard Behavior: Investigate if specific queries or types of analysis are causing the slowdown.
Step-by-Step Solution

Review Partitioning Strategy:

Evaluate the current partitioning strategy to see if it aligns with query patterns and data access frequency.
Check if the new partitioning introduced a less efficient distribution, such as too many small partitions or uneven data skew.

Analyze Query Performance:

Use Databricks SQL’s EXPLAIN feature to understand how queries are being executed.
Identify which parts of the query execution plan are causing the most delay and resource consumption.

#### Cons
ider Liquid Clustering:

What is Liquid Clustering?: Liquid clustering automatically reorganizes data in a partitioned table based on query patterns, which can improve performance by reducing the number of partitions scanned.
Evaluation Criteria: Determine if liquid clustering could benefit from our current query patterns and data access. It’s particularly useful when queries frequently scan large numbers of partitions.

Revert or Re-model:

Reverting: If liquid clustering does not provide sufficient improvement, consider reverting to the previous partitioning strategy.
Re-modeling: Evaluate if a different schema or partitioning approach could better align with query patterns and data access needs. This might involve changing how data is distributed across partitions.

#### First-Principles Solution

To decide on the best course of action, follow these steps:

Benchmark Current Performance:
Record baseline metrics for comparison.
Implement Liquid Clustering (if feasible):
Enable liquid clustering and monitor performance improvements.
Analyze Results:
Compare new metrics with the baseline to assess if liquid clustering has helped.
Revert or Re-model:
If liquid clustering does not improve performance, revert to the previous partitioning strategy or consider a re-model.

#### Closing Summary
and Common Mistakes

In summary, our approach involves a systematic evaluation of the current partitioning strategy, using tools like EXPLAIN to understand query execution, and leveraging techniques such as liquid clustering. The key is to balance optimization with maintainability and scalability. Common mistakes include not thoroughly analyzing query patterns or prematurely implementing changes without proper evaluation.

**Comparison:**

```
Metric	Baseline	After Partition Change
Partitions Scanned	100	500
Query Latency (ms)	500	5000
```

**Code:**

```sql
-- Example of Databricks SQL EXPLAIN output
EXPLAIN SELECT * FROM my_table WHERE date >= '2023-01-01';
```

**Example input / output:**

```text
Baseline Performance:
- Partitioning strategy: date, region
- Partitions scanned: 100
- Query latency: 500ms

After Partition Change:
- New partitioning strategy: date, hour, region
- Partitions scanned: 500
- Query latency: 5000ms
```

**Data:**

```json
{
  "partition_columns": ["date", "region"],
  "partitions_scanned": 50,
  "query_latency_ms": 8000,
  "cpu_utilization_percentage": 70,
  "memory_utilization_mb": 4000
}
```

**Flow:**

1. Review Partitioning Strategy
2. Analyze Query Performance
3. Liquid Clustering
4. Evaluate Results
5. Revert or Re-model
6. Choose Best Course of Action

