---
type: "practice"
difficulty: "Deep"
---

# How do you handle small-file problems produced by a streaming-to-batch feature pipeline that hurt downstream Spark read performance?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Batch vs Streaming Features](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Batch_vs_Streaming_Features/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Handling small-file problems in a data pipeline, especially when transitioning from a streaming system to batch processing with Spark, is critical to maintain performance and avoid bottlenecks. The key challenge here involves managing the overhead of reading numerous small files, which can significantly degrade the performance of Spark tasks. This problem arises because each file read incurs startup costs and context switching overheads.

To solve this issue, we need to understand the root causes and explore multiple strategies that range from modifying the streaming source to optimizing how data is processed in the batch stage. We will cover the following steps: identifying the small-file problem, stream aggregation techniques, file merging strategies, and leveraging external storage systems like HDFS or cloud object stores.

**Comparison:**

| File Count | Read Time (s) |
|-----------|--------------|
| 100       | 30           |
| 500       | 240          |
| 1,000     | 1,800        |

**Code:**

```python
# Example of a simple Spark job reading from many small files
spark = SparkSession.builder.appName("SmallFileProblem").getOrCreate()
df = spark.read.format("parquet").load("/mnt/data/streaming-output/*.parquet")
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Streaming output with many small files (e.g., /mnt/data/streaming-output/*.parquet) | Merged data ready for batch processing in Spark, with reduced file count and improved read performance. |

**Data:**

```yaml
# Sample data structure for streaming output
- timestamp: 2023-10-01T08:00:00Z
  user_id: 12345
  event: login

- timestamp: 2023-10-01T09:00:00Z
  user_id: 67890
  event: logout

# This generates a large number of small files, each containing a few records.
```

**Flow:**

1. Streaming Source
2. Aggregation Before Write
3. Merge Small Files
4. Batch Processing with Spark
5. Optimized Data Reading
6. Improved Performance

