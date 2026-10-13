---
type: "practice"
difficulty: "Expert"
---

# Your Structured Streaming checkpoint got tied to a specific query plan; how do you evolve the job's logic without invalidating the checkpoint and losing offsets?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Streaming Recovery and Deduplication](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Streaming_Recovery_and_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Structured Streaming in Spark, checkpoints are essential for ensuring fault tolerance by storing offset information in a reliable storage system like Kafka or HDFS. In this scenario, if your checkpoint is tied to a specific query plan and you need to evolve the job's logic without losing offsets, it's crucial to understand how Spark handles checkpoints and how you can manage the evolution of the job.

To tackle this problem, let's break down the core solution step by step:

Understanding Checkpoints in Structured Streaming: Checkpoints are used to periodically save metadata about the state of the processing, which allows Spark to recover from failures. If a checkpoint is tied to a specific query plan, any changes to that plan will invalidate the current checkpoint.

Evolving Job Logic Without Invalidating Checkpoints: To evolve your job logic without invalidating checkpoints and losing offsets, you need to ensure that the new query plan is compatible with the existing one. This involves maintaining backward compatibility in your schema and logic.

Backward Compatibility and Schema Changes: When evolving your Spark Structured Streaming job, always consider making changes that are backwards-compatible. For instance, if adding a column or changing the data type of an existing column, ensure that this change does not affect the offsets stored in the checkpoint.

Handling Non-Backwards Compatible Changes: If you need to make non-backwards-compatible changes, such as adding a new processing step or significantly altering the schema, you must first clean up and start with fresh checkpoints. This involves stopping the current job, deleting the old checkpoint data, and then starting the job from scratch.

Incremental Checkpointing: Consider using incremental checkpointing techniques to minimize the impact of changes. Incremental checkpoints can help in scenarios where small updates are made frequently without needing to invalidate the entire checkpoint.

Schema Evolution Strategies:

Versioned Schemas: Implement versioned schemas where you maintain historical versions and ensure that new logic can handle these versions.
Dynamic Schema Handling: Use dynamic schema handling techniques to allow the job to adapt to schema changes without invalidating checkpoints.

Testing and Validation: Thoroughly test the new query plan in a non-production environment before deploying it to production. This helps catch any issues that might arise due to compatibility problems or unexpected behavior.

By following these steps, you can evolve your Spark Structured Streaming job's logic while maintaining the integrity of your checkpoints and avoiding the loss of offsets.

**Comparison:**

| Step Name | Description |
| --- | --- |
| FilterRecords | Filters records based on certain criteria. |
| TransformData | Applies data transformation logic. |

**Code:**

```python
# Example of setting up checkpoint directory in Spark session
spark = SparkSession.builder \
    .appName("StructuredStreamingExample") \
    .config("spark.sql.streaming.checkpointLocation", "hdfs://path/to/checkpoints") \
    .getOrCreate()

df = spark.readStream.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "topic_name") \
    .load()
```

**Example input / output:**

```text
# Example input and output for checkpointing
Input:
{
  "timestamp": "2023-10-01T12:00:00Z",
  "value": {"field1": "data1", "field2": 123}
}

Output:
{
  "timestamp": "2023-10-01T12:00:00Z",
  .checkpointData: {
    "offsets": [
      { "topicPartition": {"topic": "topic_name", "partition": 0}, "offset": 100 },
      { "topicPartition": {"topic": "topic_name", "partition": 1}, "offset": 200 }
    ]
  }
}
```

**Data:**

```json
{
  "checkpointLocation": "hdfs://path/to/checkpoints",
  "streamingQueryPlan": {
    "inputSource": "Kafka topic",
    "outputSink": "Parquet file system",
    "processingSteps": [
      {
        "stepName": "FilterRecords",
        "description": "Filter records based on certain criteria"
      },
      {
        "stepName": "TransformData",
        "description": "Apply data transformation logic"
      }
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is checkpoint tied to specificplan?
5. Stop current job and deleteold checkpoints
6. Create new checkpoint forupdated query plan
7. Update logic withbackwards-compatiblechanges
8. Ensure schema evolutionstrategies are in place
9. Test new logic innon-production environment
10. Deploy to production if testspass

