---
type: "practice"
difficulty: "Deep"
---

# What is a checkpoint in Structured Streaming and why is it required for fault tolerance?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Streaming Ingest Tuning](<../../../../TOOLS/Databricks/Databricks_Platform/Streaming_Ingest_Tuning.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A checkpoint in Databricks' Structured Streaming is a mechanism to periodically save metadata about the state of your streaming query. This ensures that even if there's an interruption, you can recover the processing state and continue from where it left off. Checkpoints are crucial for fault tolerance, as they help mitigate data loss and ensure consistency.

To understand why checkpoints are required, we first need to look at how Structured Streaming works and what challenges arise when dealing with unbounded streams of data.

**Code:**

```python
# Example of a simple Databricks Structured Streaming job using PySpark
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("StructuredStreamingExample").getOrCreate()

lines = spark.readStream.format("socket").option("host", "localhost").option("port", 9999).load()
words = lines.writeStream.outputMode("append").format("console").start().awaitTermination()
```

**Flow:**

1. Yes
2. No
3. Input Stream
4. Is Checkpoint Enabled?
5. Save Metadata
6. Run Processing WithoutCheckpoints
7. Process Data with Metadata
8. Data Loss Risk
9. Output Results
10. Resume from Last Known Stateon Failure

