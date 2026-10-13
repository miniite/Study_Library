---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between narrow and wide transformations in Spark?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Spark DataFrames and Lazy Evaluation](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Spark_DataFrames_and_Lazy_Evaluation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Narrow and wide transformations are two fundamental concepts in Apache Spark, particularly when discussing Data Pipelines like those often used in AIOps (Artificial Intelligence for IT Operations). In a nutshell, narrow transformations operate on each partition of the data independently and return results of the same size as the input, while wide transformations involve combining data from multiple partitions into a single result. This distinction is crucial for understanding how tasks are executed in Spark.

To clarify what the question is probing: the interviewer wants to assess your understanding of Spark's fundamental operations and how these operations affect task execution and optimization within a pipeline. Assumptions here include familiarity with basic Spark concepts, understanding of data partitioning, and awareness of performance implications. Let’s break down each type from first principles.

Narrow Transformations

Narrow transformations in Spark operate on each partition independently and return results that are the same size as their input partitions. Examples include filter, map, and flatMap. These operations do not require any shuffle or data redistribution, making them efficient for large-scale processing tasks.

#### Example
PYTHON
# Spark Transformation Example (Narrow)
from pyspark import SparkContext

sc = SparkContext.getOrCreate()
data = [1, 2, 3, 4, 5]
rdd = sc.parallelize(data)

result = rdd.filter(lambda x: x > 2).collect()  # Returns [3, 4, 5]
Wide Transformations

Wide transformations, on the other hand, involve combining data from multiple partitions into a single result. This often necessitates shuffling data across nodes, which can significantly impact performance. Examples include groupByKey, reduceByKey, and join. Since these operations require redistributing data, they are generally less efficient than narrow transformations.

#### Example
PYTHON
# Spark Transformation Example (Wide)
from pyspark import SparkContext

sc = SparkContext.getOrCreate()
data1 = [(1, 2), (3, 4)]
rdd1 = sc.parallelize(data1)

data2 = [2, 4]
rdd2 = sc.parallelize(data2)

result = rdd1.join(rdd2).collect()  # This requires a shuffle operation

#### First-Principles Solution

Understanding the difference is essential for optimizing Spark jobs. Narrow transformations are generally preferred because they can be executed in parallel without shuffling data, making them faster and more scalable. Wide transformations should be used judiciously to avoid bottlenecks caused by excessive shuffling.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, narrow transformations operate independently on each partition, while wide transformations require redistributing data across nodes. Choosing the right transformation can greatly impact performance in Spark jobs. Common mistakes include overusing wide transformations and not leveraging broadcast variables or in-memory caching to mitigate shuffling costs.

For follow-ups, you might be asked about specific use cases where wide transformations are necessary, or how to optimize for both memory usage and job execution time.

**Comparison:**

| Transformation Type | Description | Example |
|---------------------|-------------|---------|
| Narrow              | Operates on each partition independently, returns results of the same size as input. | `filter`, `map`, `flatMap` |
| Wide                | Combines data from multiple partitions into a single result, requires shuffling. | `groupByKey`, `reduceByKey`, `join` |

**Code:**

```python
# Narrow Transformation Example (Filter)
from pyspark import SparkContext

sc = SparkContext.getOrCreate()
data = [1, 2, 3, 4, 5]
rdd = sc.parallelize(data)

result_narrow = rdd.filter(lambda x: x > 2).collect()  # [3, 4, 5]

# Wide Transformation Example (Join)
from pyspark import SparkContext

sc = SparkContext.getOrCreate()
data1 = [(1, 2), (3, 4)]
rdd1 = sc.parallelize(data1)

data2 = [2, 4]
rdd2 = sc.parallelize(data2)

result_wide = rdd1.join(rdd2).collect()  # Requires a shuffle
```

**Example input / output:**

```text
# Narrow Transformation Example (Filter)
from pyspark import SparkContext

sc = SparkContext.getOrCreate()
data = [1, 2, 3, 4, 5]
rdd = sc.parallelize(data)

result_narrow = rdd.filter(lambda x: x > 2).collect()  # Output: [3, 4, 5]

# Wide Transformation Example (Join)
from pyspark import SparkContext

sc = SparkContext.getOrCreate()
data1 = [(1, 2), (3, 4)]
rdd1 = sc.parallelize(data1)

data2 = [2, 4]
rdd2 = sc.parallelize(data2)

result_wide = rdd1.join(rdd2).collect()  # Output: Requires a shuffle
```

**Data:**

```json
{
    "narrow_transformations": ["filter", "map", "flatMap"],
    "wide_transformations": ["groupByKey", "reduceByKey", "join"]
}
```

**Flow:**

1. Optimizations
2. Narrow Transformations
3. Filter
4. Map
5. Wide Transformations
6. GroupByKey
7. ReduceByKey
8. JOIN
9. Use Broadcast Variables
10. Caching in Memory
11. D

