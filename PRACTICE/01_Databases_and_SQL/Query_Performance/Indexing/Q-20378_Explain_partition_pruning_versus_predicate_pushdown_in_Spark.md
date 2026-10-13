---
type: "practice"
difficulty: "Deep"
---

# Explain partition pruning versus predicate pushdown in Spark: are they the same optimization, and where does each happen in the plan?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Partitioning and Data Skipping in Lakes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Partitioning_and_Data_Skipping_in_Lakes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Partition pruning and predicate pushdown are two distinct but closely related query optimizations used in Apache Spark to improve performance by reducing the amount of data processed during a query execution. While they operate at different stages of the query processing pipeline, both aim to reduce the computational load by leveraging metadata about the data being processed.

#### Triage
What’s Being Probed

The question is essentially asking you to differentiate between partition pruning and predicate pushdown in Spark. The interviewer wants to understand your grasp of how these optimizations work, their differences, and their respective places within a query execution plan. You should assume that the focus will be on both the technical details and practical implications.

#### First-Principles Solution

Partition pruning occurs during the planning phase of a query before any data is actually read from disk or memory. It involves filtering out partitions based on the partitioning scheme (e.g., range, hash) to only consider relevant partitions for processing. Predicate pushdown, on the other hand, happens at runtime when Spark reads data and processes it, pushing filter conditions down into storage layers such as HDFS or a database system.

Partition Pruning
What It Does: Partition pruning filters out irrelevant partitions based on metadata.
When It Happens: During query planning.
How It Works:
Partitioning Schemes: Typically used with columnar stores like Parquet, where data is organized in such a way that allows for efficient partition pruning.

#### Example
If a query has a WHERE clause filtering on a date column, and the data is stored by year, Spark can skip processing partitions from years outside the range of interest.
Predicate Pushdown
What It Does: Filters are pushed down into storage layers to reduce the amount of data transferred.
When It Happens: During execution.
How It Works:
Storage Layer Interaction: Spark integrates with storage systems that support predicate pushdown, such as HDFS or Hive.

#### Example
If a query has a WHERE clause filtering on a column in an external table, Spark can pass the filter directly to the storage layer, which then returns only relevant data.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, partition pruning and predicate pushdown are distinct optimizations. Partition pruning occurs during planning and filters out irrelevant partitions based on metadata, while predicate pushdown happens at runtime and reduces the amount of data processed by pushing filter conditions into storage layers. Understanding these differences is crucial for effective query optimization in Spark.

#### Common mistakes
include confusing these two optimizations or assuming they are the same. Be prepared to discuss real-world scenarios where one might be more beneficial than the other, such as partitioning schemes being ineffective when dealing with complex predicates that cannot be directly translated to partitions.

#### Follow-ups
could involve discussing best practices for choosing between them based on data characteristics and storage systems used.

**Code:**

```python
# Example code showing predicate pushdown in Spark

from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("Predicate Pushdown").getOrCreate()

data = [("Alice", 28, "NY"), ("Bob", 30, "LA"), ("Carol", 25, "SF")]
columns = ["name", "age", "location"]

df = spark.createDataFrame(data, columns)
df.write.format("parquet").save("/path/to/data")

# Query with predicate pushdown
filtered_df = df.filter(df.age > 26).select("name", "location")
filtered_df.show()
```

**Example input / output:**

```text
Input: df.filter(df.age > 26).select("name", "location")

Output:
+----+----------+
| name| location |
+----+----------+
| Carol|      SF  |
+----+----------+
```

**Flow:**

1. No
2. Yes
3. Query Execution Plan
4. Planning
5. Partition Pruning
6. Predicate Pushdown
7. Execution
8. Runtime
9. Storage Layer

