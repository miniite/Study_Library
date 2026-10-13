---
type: "practice"
difficulty: "Deep"
---

# How does Adaptive Query Execution (AQE) change a plan at runtime and which three optimizations does it apply — when have you seen it help or hurt?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reducing Shuffle in PySpark](<../../../../TOOLS/Apache_Spark/PySpark/Reducing_Shuffle_in_PySpark.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Adaptive Query Execution (AQE) in PySpark is a dynamic optimization mechanism that refines query plans based on runtime statistics, allowing Spark to adaptively optimize queries as they execute. AQE can apply three primary optimizations: splitting large partitions, fusing shuffle maps, and adjusting hash partitioning. These optimizations are particularly valuable when working with complex data pipelines or large datasets.

The core idea behind AQE is that the initial query plan might not be optimal due to insufficient information about the data distribution at the time of execution. By collecting statistics during query execution, AQE can make informed decisions to improve performance dynamically.

#### Triaging the Question

What the Question is Asking:

The question focuses on understanding how AQE modifies a query plan in real-time and identifies three specific optimizations it applies.
It also seeks examples where these optimizations have helped or harmed performance.

#### Assumptions

We are dealing with scenarios involving PySpark.
We need to discuss both the theoretical benefits of AQE and practical experiences, highlighting cases where AQE improved or degraded query performance.

#### First Principles Solution
Splitting Large Partitions:

AQE can split large partitions into smaller ones if it detects that processing a single partition is inefficient. This is particularly useful for operations like join or group by, which become more manageable with smaller datasets.

Fusing Shuffle Maps:

Shuffle operations in Spark involve redistributing data across nodes, often leading to inefficiencies. AQE can fuse shuffle maps to reduce the number of shuffles, thereby improving performance by minimizing overhead.

Adjusting Hash Partitioning:

AQE can adjust the number of partitions used for hash-based operations based on runtime statistics. This is crucial for queries involving group by or join, where optimal partitioning can significantly impact performance.

Common Examples and Scenarios

#### Example
1: Splitting Large Partitions Suppose we have a query that involves joining two large datasets:

SQL
df1.join(df2, df1.id == df2.id)

Initially, both df1 and df2 are partitioned with many partitions. However, if one of the tables has a skewed distribution (e.g., a few keys dominate), AQE might recognize this during execution and decide to split these large partitions into smaller ones.

#### Example
2: Fusing Shuffle Maps Consider a scenario where we have multiple shuffle operations:

SQL
df1.groupBy('key').sum()

AQE can fuse these shuffle maps, reducing the number of shuffles from 3 to 1, which significantly improves performance by minimizing the overhead associated with shuffling data.

#### Example
3: Adjusting Hash Partitioning For a query involving:

SQL
df.groupBy('category', 'subcategory').count()

AQE might initially use more partitions but then adjust based on runtime statistics. If it detects that fewer partitions are sufficient, it can reduce the number of partitions to optimize performance.

When AQE Helps or Hurts

Helpful Scenarios:

Complex Joins: Large joins with skewed data distributions often benefit from partition splitting.
Multiple Shuffles: Complex queries involving multiple shuffle operations see significant improvements when AQE fuses them.
Dynamic Workload Changes: AQE is particularly useful in environments where the data distribution or workload changes frequently.

Harmful Scenarios:

Over-Partitioning: If AQE splits partitions excessively, it can lead to increased overhead without substantial performance gains.
Inconsistent Data Distribution: If data distributions change rapidly and unpredictably, AQE might struggle to keep up, leading to suboptimal plans.
Resource Contention: In environments with limited resources, excessive AQE activity could cause contention, slowing down the overall system.

#### Closing Summary

Adaptive Query Execution (AQE) in PySpark dynamically refines query plans based on runtime statistics. It applies optimizations like splitting large partitions, fusing shuffle maps, and adjusting hash partitioning to improve performance. While AQE is highly beneficial for complex queries with dynamic data distributions, it can also lead to suboptimal plans if the environment or workload changes rapidly.

#### Common Mistakes/Follow-Ups
Over-Splitting Partitions: Ensuring that partitions are neither too large nor overly split.
Under-Utilization of AQE: Recognizing situations where AQE is not necessary and avoiding unnecessary overhead.
Tuning AQE Settings: Adjusting AQE parameters to balance between dynamic optimization and system stability.

**Comparison:**

| Partition | Original Plan               | Optimized Plan                |
|-----------|-----------------------------|-------------------------------|
| df1       | 3 partitions (skewed)        | 6 smaller partitions          |
| df2       | 4 partitions (uniform)       | 8 smaller partitions          |

**Code:**

```python
# Example of a PySpark query with AQE enabled
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("AQEExample").getOrCreate()

df1 = spark.read.csv("path/to/file1")
df2 = spark.read.csv("path/to/file2")

result_df = df1.join(df2, df1.id == df2.id)
```

**Example input / output:**

```text
Initial Query:
df1.join(df2, df1.id == df2.id)

Optimized Plan (with AQE):
- Split large partitions in df1 and df2
- Fuse shuffle maps to reduce overhead
- Adjust hash partitioning for optimal performance

Result:
[
  {id: 1, value: 25},
  {id: 4, value: NULL}
]
```

**Data:**

```json
{
  "file1": {
    "id": [1, 2, 3],
    "value": [10, 20, 30]
  },
  "file2": {
    "id": [1, 4, 5],
    "value": [15, 25, 35]
  }
}
```

**Flow:**

1. No
2. Yes
3. Yes
4. No
5. Yes
6. No
7. Yes
8. No
9. Initial Query Plan
10. Is AQE Enabled?
11. Execute Plan as Is
12. Collect Runtime Statistics
13. Split Large Partitions?
14. Optimized Plan with PartitionSplitting
15. Fuse Shuffle Maps?
16. Optimized Plan with ShuffleMap Fusion
17. Adjust Hash Partitioning?
18. Optimized Plan with AdjustedPartitioning
19. Execute Optimized Plan

