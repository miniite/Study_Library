---
type: "practice"
difficulty: "Deep"
---

# How would you use the Spark event log / history server to compare two runs and prove a partitioning change reduced shuffle bytes?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Spark Skew and Spill](<../../../../TOOLS/Apache_Spark/Spark_Optimization/Spark_Skew_and_Spill.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compare two runs of Apache Spark jobs using the event logs or history server, specifically to prove that a partitioning change has reduced shuffle bytes, we need to follow a systematic approach. This involves understanding how shuffle works in Spark, identifying key metrics from the event logs, and then conducting a detailed analysis.

#### Triaging the Question

The core of this question is about comparing two Spark job runs with different partitioning strategies to measure the impact on shuffle bytes. Shuffle bytes are crucial for performance as they represent data movement between tasks within the same stage. The history server stores detailed execution metrics, including shuffle read and write bytes, which can be used to compare the two runs.

#### Assumptions
We have access to Apache Spark's history server.
Both jobs were run with the same dataset but different partitioning strategies.
The dataset size is sufficiently large to ensure meaningful statistical analysis.
Step-by-Step Solution
1. Understanding Shuffle in Spark

Shuffle occurs during operations like groupByKey, reduceByKey, and join where data needs to be redistributed across tasks. It involves creating a shuffle map output from the source RDDs, which are then combined into reduce tasks.

2. Metrics to Monitor
Shuffle Read Bytes: The amount of data read during shuffles.
Shuffle Write Bytes: The amount of data written as part of shuffles.
Number of Shuffle Parts: The number of shuffle partitions created.

These metrics are crucial for understanding the performance impact of different partitioning strategies.

3. Accessing Event Logs

Event logs from the history server provide detailed information about job execution, including:

PLAINTEXT
job_id: 123456
stage_id: 7890
shuffle_read_bytes: 123456789
shuffle_write_bytes: 987654321
4. Extracting Metrics for Each Job

To compare the two runs, we need to extract these metrics from each job's event logs.

PYTHON
from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()

# Example query to fetch shuffle metrics
def get_shuffle_metrics(spark_session, job_id):
    events_df = spark_session.read.format("parquet").option("path", f"historyserver/events/{job_id}").load()
    shuffle_metrics = (
        events_df.filter(col("event") == "Stage Completed")
        .select(
            col("shuffle_read_bytes"),
            col("shuffle_write_bytes"),
            col("stage_id")
        )
        .collect()
    )
    return shuffle_metrics

# Example usage
job1_shuffle_metrics = get_shuffle_metrics(spark, 123456)
job2_shuffle_metrics = get_shuffle_metrics(spark, 789012)

print(job1_shuffle_metrics)
print(job2_shuffle_metrics)
5. Analyzing and Comparing Metrics

Once we have the shuffle metrics for both jobs, we can compare them to see if the partitioning change has reduced the shuffle bytes.

PYTHON
shuffle_read_bytes_job1 = sum(metric['shuffle_read_bytes'] for metric in job1_shuffle_metrics)
shuffle_write_bytes_job1 = sum(metric['shuffle_write_bytes'] for metric in job1_shuffle_metrics)

shuffle_read_bytes_job2 = sum(metric['shuffle_read_bytes'] for metric in job2_shuffle_metrics)
shuffle_write_bytes_job2 = sum(metric['shuffle_write_bytes'] for metric in job2_shuffle_metrics)

print(f"Shuffle Read Bytes: Job 1 - {shuffle_read_bytes_job1}, Job 2 - {shuffle_read_bytes_job2}")
print(f"Shuffle Write Bytes: Job 1 - {shuffle_write_bytes_job1}, Job 2 - {shuffle_write_bytes_job2}")

If the values for shuffle_read_bytes and shuffle_write_bytes are lower in job 2 compared to job 1, it indicates that the partitioning change has been effective in reducing shuffle bytes.

6. Additional Considerations

#### Cons
istency Check: Ensure that both jobs had similar conditions (e.g., same Spark configuration, data size).
Statistical Significance: Use statistical methods if comparing multiple runs to ensure the observed difference is significant.
Other Metrics: Also consider other performance metrics like task duration and number of tasks for a comprehensive analysis.

#### Closing Summary

By leveraging Apache Spark's event logs and history server, we can systematically compare two runs with different partitioning strategies. This approach helps in quantitatively assessing whether the partitioning change has reduced shuffle bytes, which is crucial for improving job performance.

#### Common Mistakes/Follow-Ups
Not Considering Other Metrics: Focusing only on shuffle bytes without considering other performance indicators.
Inadequate Data Size: Running comparisons on small datasets may not yield meaningful results.
Ignoring Configuration Factors: Not accounting for variations in Spark configuration or data characteristics between the two runs.

**Comparison:**

| Shuffle Metrics | Job 1          | Job 2          |
|-----------------|----------------|----------------|
| Shuffle Read Bytes (GB) | 123.4       | 87.6          |
| Shuffle Write Bytes (GB) | 98.8       | 65.4          |

**Code:**

```python
from pyspark.sql import SparkSession

def get_shuffle_metrics(spark_session, job_id):
    events_df = spark_session.read.format("parquet").option("path", f"historyserver/events/{job_id}").load()
    shuffle_metrics = (
        events_df.filter(col("event") == "Stage Completed")
        .select(
            col("shuffle_read_bytes"),
            col("shuffle_write_bytes"),
            col("stage_id")
        )
        .collect()
    )
    return shuffle_metrics

# Example usage
spark = SparkSession.builder.getOrCreate()
job1_shuffle_metrics = get_shuffle_metrics(spark, 123456)
job2_shuffle_metrics = get_shuffle_metrics(spark, 789012)

shuffle_read_bytes_job1 = sum(metric['shuffle_read_bytes'] for metric in job1_shuffle_metrics)
shuffle_write_bytes_job1 = sum(metric['shuffle_write_bytes'] for metric in job1_shuffle_metrics)

shuffle_read_bytes_job2 = sum(metric['shuffle_read_bytes'] for metric in job2_shuffle_metrics)
shuffle_write_bytes_job2 = sum(metric['shuffle_write_bytes'] for metric in job2_shuffle_metrics)

print(f"Shuffle Read Bytes: Job 1 - {shuffle_read_bytes_job1}, Job 2 - {shuffle_read_bytes_job2}")
print(f"Shuffle Write Bytes: Job 1 - {shuffle_write_bytes_job1}, Job 2 - {shuffle_write_bytes_job2}")
```

**Example input / output:**

```text
Input:
- job1_shuffle_metrics = [{'shuffle_read_bytes': 123456789, 'shuffle_write_bytes': 987654321, 'stage_id': 7890}]
- job2_shuffle_metrics = [{'shuffle_read_bytes': 87654321, 'shuffle_write_bytes': 654321876, 'stage_id': 7890}]

Output:
Shuffle Read Bytes: Job 1 - 123456789, Job 2 - 87654321
Shuffle Write Bytes: Job 1 - 987654321, Job 2 - 654321876
```

**Data:**

```plaintext
job_id: 123456
stage_id: 7890
event: Stage Completed
shuffle_read_bytes: 123456789
shuffle_write_bytes: 987654321
```

**Flow:**

1. Yes
2. No
3. Access Event Logs
4. Extract Shuffle Metrics
5. Are Both Jobs Comparable?
6. Compare Metrics
7. Adjust Comparisons
8. Determine Improvement

