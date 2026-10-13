---
type: "practice"
difficulty: "Deep"
---

# Explain how Adaptive Query Execution in Spark 3 can switch a sort-merge join to a broadcast join at runtime and coalesce shuffle partitions.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Index Tuning in Practice](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Adaptive Query Execution (AQE) in Apache Spark 3 is a powerful feature that dynamically optimizes query execution by switching between different strategies, such as sort-merge joins to broadcast joins, based on runtime statistics and characteristics of the data. This optimization can significantly improve performance when dealing with skewed data or small datasets. In this answer, we'll delve into how AQE works in detail, focusing on its ability to switch join types and coalesce shuffle partitions.

Firstly, let's break down what AQE is probing for:

Runtime Performance Monitoring: AQE tracks runtime statistics of the query execution.
Dynamic Strategy Selection: It selects the most efficient execution strategy based on these statistics.
Partition Management: AQE can dynamically manage and coalesce shuffle partitions to reduce memory usage and improve performance.

#### Triaging the Question

AQE in Spark 3 is designed to adaptively choose between different join strategies at runtime, switching from sort-merge joins to broadcast joins when it detects that the latter would be more efficient. This involves monitoring the size of data being processed, detecting skew, and making decisions about partitioning.

#### First Principles Solution
Sort-Merge Join: Typically used for large datasets where both partitions are too big to fit in memory. It sorts each input partition separately and then merges them based on a common key.
Broadcast Join: Used when one of the data sets is small enough to be loaded into memory (broadcasted). The smaller dataset is sent to every executor, reducing the need for shuffling large datasets.
Scenario: Sort-Merge Join to Broadcast Join

#### Cons
ider a scenario where we have two datasets, A and B, with keys that are distributed across multiple partitions. Initially, AQE might decide a sort-merge join is necessary because both datasets are too large:

SQL
val df1 = spark.read.parquet("path/to/df1")
val df2 = spark.read.parquet("path/to/df2")

df1.join(df2, "key").show()

However, during the execution, AQE observes that df2 is small enough to fit into memory on all executors. It then decides to switch the join strategy from sort-merge to broadcast:

SQL
spark.conf.set("spark.sql.adaptive.enabled", true)
val df1 = spark.read.parquet("path/to/df1")
val df2 = spark.read.parquet("path/to/df2")

df1.join(df2, "key").show()
Coalescing Shuffle Partitions

Coalescing shuffle partitions is a related optimization that AQE can perform to reduce the number of shuffle partitions. By default, Spark creates as many shuffle partitions as there are executors. However, in cases where too many small partitions cause excessive shuffling overhead, AQE can coalesce these partitions into fewer ones.

SCALA
val df1 = spark.read.parquet("path/to/df1").repartition(10)  // Create 10 shuffle partitions
val df2 = spark.read.parquet("path/to/df2").repartition(5)   // Create 5 shuffle partitions

spark.conf.set("spark.sql.adaptive.enabled", true)
val result = df1.join(df2, "key")

AQE can then coalesce these partitions to a more optimal number:

SCALA
result.coalesce(3).show()

#### Closing Summary
and Common Mistakes/Follow-Ups

Adaptive Query Execution in Spark 3 is a sophisticated feature that dynamically optimizes query execution by switching join strategies and managing shuffle partitions based on runtime statistics. This can lead to significant performance improvements, especially when dealing with skewed data or small datasets.

#### Common mistakes
include not enabling AQE (by setting spark.sql.adaptive.enabled to true) or misconfiguring the threshold conditions under which it operates. Understanding these nuances is crucial for leveraging AQE effectively in real-world scenarios.

For follow-ups, you might want to explore:

How AQE handles different types of joins and their trade-offs.
The impact of AQE on memory management and task scheduling.
Best practices for tuning Spark configurations to work well with AQE.

**Comparison:**

| Partition Count | Impact on Performance |
|-----------------|-----------------------|
| Too many       | Increased shuffling   |
| Optimal        | Reduced overhead      |
| Too few        | Underutilized resources |

**Code:**

```scala
// Example enabling AQE
spark.conf.set("spark.sql.adaptive.enabled", true)
val df1 = spark.read.parquet("path/to/df1")
val df2 = spark.read.parquet("path/to/df2")

// Join operation with AQE enabled
val result = df1.join(df2, "key").show()
```

**Example input / output:**

```

+---+----+-----+ | id|name|value| +---+----+-----+ | 1| A| 10| +---+----+-----+

CODE
Copy

### Example of Coalescing Shuffle Partitions
```scala
// Original DataFrame with multiple partitions
val df1 = spark.read.parquet("path/to/df1").repartition(10)
val df2 = spark.read.parquet("path/to/df2").repartition(5)

// Enabling AQE and showing the result after coalescing
spark.conf.set("spark.sql.adaptive.enabled", true)
val result = df1.join(df2, "key")
result.coalesce(3).show()
```

**Data:**

```json
{
  "df1": {
    "schema": ["id", "name", "value"],
    "data": [[1, "A", 10], [2, "B", 20], [3, "C", 30]]
  },
  "df2": {
    "schema": ["key", "type", "amount"],
    "data": [["A", "Type1", 50], ["D", "Type2", 60]]
  }
}
```

**Flow:**

1. Yes
2. No
3. Start Query
4. Collect Runtime Stats
5. Is Data Skewed?
6. Choose Broadcast Join
7. Perform Sort-Merge Join
8. Optimize Coalesced Partitions
9. Continue with Sort-Merge Join
10. Show Results
11. End Query Execution

