---
type: "practice"
difficulty: "Deep"
---

# A job's shuffle-heavy stage keeps failing with FetchFailedException. Walk me through the root causes and your recovery/tuning path.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Spark Skew and Spill](<../../../../TOOLS/Apache_Spark/Spark_Optimization/Spark_Skew_and_Spill.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a Spark job that fails due to FetchFailedException in a shuffle-heavy stage, the key is to identify and address skew in data distribution or improper partitioning. This failure typically occurs when tasks are unable to fetch required data from other nodes due to skewed data partitions.

The goal of this recovery/tuning path is to ensure balanced data distribution across partitions and optimize shuffling for better performance. We'll start by triaging the issue, then move through a series of steps to diagnose and resolve it.

#### Triage
Understanding FetchFailedException
Understanding the Exception: FetchFailedException in Spark typically indicates that a task is unable to fetch data from another node due to network issues or skewed data distribution.

#### Assumptions
The cluster setup and configuration are correct.
Network connectivity between nodes is stable.

#### First-Principles Solution
Identify Skew in Data Distribution:
Skewed data can cause some tasks to handle disproportionately large amounts of data, leading to failures.
Analyze Partitioning Strategy:
Review the partitioning strategy used in your dataset. Common strategies include hash partitioning and range partitioning.
Optimize Shuffling:
Adjust the number of partitions based on the data size and available resources.
Use Bucketing:
If possible, apply bucketing to ensure more balanced data distribution.
Step-by-Step Solution

Check Data Distribution:

Use spark.sql.shuffle.partitions or set it in your Spark configuration.
Run a simple query like df.printSchema() and inspect the partition keys to see if there is skew.

Increase Number of Partitions:

Temporarily increase the number of partitions to see if this alleviates the issue.
PYTHON
spark.conf.set("spark.sql.shuffle.partitions", 100)

Use Bucketing for Skewed Data:

If you identify specific keys causing skew, use bucketing on those columns.
PYTHON
df = df.repartition(col('skewed_column'))

Optimize Shuffle Operations:

Use coalesce and repartition judiciously to reduce the number of shuffle stages.
PYTHON
df = df.coalesce(num_partitions)

Network Optimization:

Ensure that network settings are optimized for large data transfers.

Retry Mechanism:

Implement a retry mechanism in your application logic to handle transient failures.

#### Closing Summary
and Common Mistakes
Skewed data can significantly impact shuffle-heavy stages, leading to FetchFailedException.
The key steps involve identifying skew, optimizing partitioning, and fine-tuning shuffle operations.

#### Common mistakes
include not adequately understanding the data distribution or setting too many partitions without considering available resources.
Common Follow-Ups
How do you handle large datasets with significant skew?
What are the trade-offs between increasing the number of partitions vs. using bucketing?
Can you explain the impact of network optimization on shuffle-heavy stages?

**Code:**

```python
# Example code for setting shuffle partitions
spark.conf.set("spark.sql.shuffle.partitions", 100)
```

