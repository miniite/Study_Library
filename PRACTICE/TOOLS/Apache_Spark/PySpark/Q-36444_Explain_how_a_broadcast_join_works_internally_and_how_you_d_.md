---
type: "practice"
difficulty: "Intermediate"
---

# Explain how a broadcast join works internally and how you'd decide the threshold for broadcasting a dimension table in production.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reducing Shuffle in PySpark](<../../../../TOOLS/Apache_Spark/PySpark/Reducing_Shuffle_in_PySpark.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Broadcast joins are an optimization technique used in PySpark to efficiently handle large datasets by reducing the amount of shuffling and data movement during operations like joins. In essence, broadcast joins work by bringing a smaller table (the "broadcasted" or "small" dataset) into memory on all nodes where the larger table is being processed. This allows for faster join operations since each node can perform the join in-memory without needing to shuffle large amounts of data across the network.

#### Triaging the Question

The question asks you to explain how broadcast joins work internally and to decide a threshold for broadcasting a dimension table in production. The key points here are understanding the internal mechanics, identifying when it's appropriate to use this technique, and determining an optimal size threshold for the smaller dataset.

#### First-Principles Solution
Internal Mechanics

A broadcast join works by first persisting (saving in memory) the smaller table in a broadcast variable on every node. When performing the join operation, PySpark looks up the smaller table's values directly from memory instead of shuffling and joining over the network. This is particularly useful when one table is significantly smaller than the other and can fit into memory on all nodes.

Choosing the Threshold

To decide whether to broadcast a dimension table in production, consider several factors:

Table Size: The size of the dimension table relative to the fact table.
Memory Constraints: Available memory per node and across the cluster.
Join Frequency: How often this join operation is performed.
Performance Impact: Whether the current method (full outer join) or other optimizations are sufficiently slow.

Here’s a step-by-step approach:

Estimate the Size: Measure the size of both tables to understand if one can fit into memory on all nodes. Typically, broadcast variables should be less than 100MB.
Cluster Memory: Calculate the total available memory across your cluster and ensure that it can accommodate multiple copies of the smaller table.
Performance Baseline: Measure the performance of your current join operation (e.g., full outer join). Compare this with the expected performance gain from a broadcast join.
Dynamic Thresholds: Depending on the variability in data, you might want to implement dynamic thresholds where the decision is made at runtime based on the size of the tables.

#### Example
Code
PYTHON
# Example code for performing a broadcast join
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("Broadcast Join Example").getOrCreate()

small_table = spark.read.csv("path/to/small_table.csv", header=True)
large_table = spark.read.csv("path/to/large_table.csv", header=True)

# Check if the small table can fit into memory
if small_table.rdd.memoryUsage() < 100e6: # Less than 100MB
    broadcasted_small_table = spark.sparkContext.broadcast(small_table)

    joined_df = large_table.join(
        broadcasted_small_table.value,
        on="key_column",
        how="left_outer"
    )
else:
    joined_df = large_table.join(
        small_table,
        on="key_column",
        how="left_outer"
    )

joined_df.show()

#### Common Mistakes
and Follow-Ups
Underestimating Memory: Not accounting for the memory overhead of broadcast variables.
Overhead of Broadcast Join: Ignoring the fact that broadcasting a large table can also cause performance issues due to increased heap usage.
Inconsistent Thresholds: Using static thresholds without considering dynamic changes in data size.

#### Closing Summary

Understanding how broadcast joins work and setting appropriate thresholds is crucial for optimizing PySpark jobs. By carefully estimating the size of tables, understanding memory constraints, and measuring performance gains, you can make informed decisions that significantly improve job efficiency and reduce costs.

**Comparison:**

| Table | Column | Data Type |
|-------|--------|----------|
| small_table | key | Integer |
| small_table | val | String |
| large_table | key | Integer |
| large_table | fact_val | String |

**Code:**

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("Broadcast Join Example").getOrCreate()

small_table = spark.read.csv("path/to/small_table.csv", header=True)
large_table = spark.read.csv("path/to/large_table.csv", header=True)

# Check if the small table can fit into memory
if small_table.rdd.memoryUsage() < 100e6: # Less than 100MB
    broadcasted_small_table = spark.sparkContext.broadcast(small_table)

    joined_df = large_table.join(
        broadcasted_small_table.value,
        on="key_column",
        how="left_outer"
    )
else:
    joined_df = large_table.join(
        small_table,
        on="key_column",
        how="left_outer"
    )

joined_df.show()
```

**Example input / output:**

```text
+---+----+
| key| val|
+---+----+
|   1|  aaa|
|   2|  bbb|
|   3|  ccc|
+---+----+

+---+----+
| key| fact_val|
+---+---------+
|   4|       xxx|
|   5|       yyy|
|   6|       zzz|
+---+---------+
```

**Data:**

```plaintext
path/to/small_table.csv
path/to/large_table.csv
```

**Flow:**

1. Yes
2. No
3. Check Table Size
4. Less Than 100MB?
5. Broadcast Small Table
6. Perform Normal Join
7. Join with Large Table
8. Show Result

