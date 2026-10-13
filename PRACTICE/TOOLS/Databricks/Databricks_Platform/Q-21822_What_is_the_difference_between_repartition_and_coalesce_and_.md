---
type: "practice"
difficulty: "Deep"
---

# What is the difference between repartition and coalesce and when would you use each?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Databricks Query Performance](<../../../../TOOLS/Databricks/Databricks_Platform/Databricks_Query_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Repartition and coalesce are both functions used to manage partitioning in Databricks, which helps optimize data processing tasks by distributing them more efficiently across workers. Repartition re-partitions an RDD (Resilient Distributed Dataset) into a specified number of partitions, while coalesce reduces the number of partitions without repartitioning. Understanding when and why to use each can significantly impact performance.

Let's break down both functions from first principles:

What are Repartition and Coalesce?

Repartition is used to partition an RDD in such a way that you have full control over the number of partitions, which can be useful for achieving specific data distribution or reducing shuffling. It will shuffle all the data across the cluster.

Coalesce, on the other hand, reduces the number of partitions while minimizing shuffling and memory overhead. It is particularly useful when you need to reduce the number of partitions but don't necessarily want to repartition with a different number.

When to Use Each

Repartition:

You have an RDD that has too many or too few partitions, and you need control over the exact number of partitions.
You want to ensure data is distributed evenly for better load balancing across nodes.
You are performing operations like groupBy or join, where the resulting dataset needs a specific number of partitions.

Coalesce:

When you have too many partitions and need to reduce them while minimizing overhead.
If you’re doing large transformations that can handle a larger amount of data in each partition, reducing partitions can help optimize performance.
To decrease the number of shuffle stages, which can improve performance by reducing network traffic.

#### Triage
What's Being Probed

The interviewer is probing your understanding of:

The fundamental differences between repartition and coalesce.
When to use specific functions based on the context (e.g., when you need control over partition count vs. minimizing shuffling).
How these operations impact data distribution and performance.

#### Example Scenario
s

Repartition Example:

PYTHON
# Suppose we have an RDD with 10 partitions, but our subsequent operation requires 3 partitions.
data = spark.sparkContext.parallelize(range(10), 10)
repartitioned_data = data.repartition(3)

Coalesce Example:

PYTHON
# If we have 20 partitions and want to reduce them to 5.
coalesced_data = data.coalesce(5)

#### Closing Summary
+ Common Mistakes/Follow-Ups

Repartitioning is essential when you need control over the number of partitions, while coalesce can be used effectively for reducing partition counts with minimal overhead. Common mistakes include:

Using repartition without considering the actual impact on data shuffling.
Overusing coalesce, which could lead to increased memory usage and longer execution times if not properly managed.

#### Follow-ups
might involve discussing specific scenarios where these functions are critical or exploring how they interact with other Spark operations like map, reduceByKey, etc.

**Code:**

```python
# Example of using repartition
data = spark.sparkContext.parallelize(range(10), 10)
repartitioned_data = data.repartition(3)

# Example of using coalesce
coalesced_data = data.coalesce(5)
```

**Example input / output:**

```text
input_data = spark.sparkContext.parallelize(range(10), 10)
repartitioned_output = input_data.repartition(3).collect()
coalesced_output = input_data.coalesce(5).collect()

print(repartitioned_output)  # Output: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
print(coalesced_output)     # Output: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

**Flow:**

1. Repartition
2. Number of Partitions?
3. Shuffle and Repartition Data
4. Coalesce
5. Reduce Number of Partitions?
6. Minimize Shuffling, ReduceOverhead

