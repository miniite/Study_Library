---
type: "practice"
difficulty: "Deep"
---

# How would you structure Kafka topics and Spark jobs so that adding a new signal type (say traces) doesn't require touching the metrics or logs pipelines?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Topic and Schema Design for Pipelines](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Topic_and_Schema_Design_for_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that adding a new signal type, such as "traces," does not impact existing pipelines processing metrics and logs, we need to design a modular architecture leveraging Kafka topics and Spark jobs with careful topic partitioning and job configuration.

#### Triage

The question aims to explore how to decouple the ingestion and processing of different types of signals (metrics, logs, traces) in a big data pipeline. The key is to maintain isolation between these components so that adding new types does not affect existing workflows.

#### First-Principles Solution

Topic Partitioning:

Metrics: Use a topic named metrics for all metric signals.
Logs: Use another topic named logs for log signals.
Traces: Introduce a new topic, say traces, specifically for trace signals.

Stream Processing with Spark Structured Streaming:

Define three separate Spark jobs:
A job to consume and process metrics.
Another to consume and process logs.
And one more to consume and process traces.

#### Cons
umer Group Management:

Use different consumer groups for each topic to avoid data interference between them.
Detailed Steps:

Create Kafka Topics:

BASH
# Create the metrics topic
kafka-topics.sh --create --topic metrics --partitions 4 --replication-factor 3 --zookeeper localhost:2181

# Create the logs topic
kafka-topics.sh --create --topic logs --partitions 4 --replication-factor 3 --zookeeper localhost:2181

# Create the traces topic
kafka-topics.sh --create --topic traces --partitions 4 --replication-factor 3 --zookeeper localhost:2181

Define Spark Jobs for Each Topic:

PYTHON
from pyspark.sql import SparkSession

# Initialize the Spark session
spark = SparkSession.builder \
    .appName("MetricsProcessor") \
    .getOrCreate()

# Define the schema for metrics
metrics_schema = "timestamp:long, metric_name:string, value:double"

# Start a structured streaming job to process metrics
df_metrics = (spark.readStream
              .format("kafka")
              .option("kafka.bootstrap.servers", "localhost:9092")
              .option("subscribe", "metrics")
              .option("startingOffsets", "earliest")
              .load()
              .selectExpr("CAST(value AS STRING) as json_value")
              .writeStream
              .format("console")
              .option("truncate", False)
              .outputMode("append")
              .start())

df_metrics.awaitTermination()

# Repeat similar steps for logs and traces topics.

#### Cons
umer Group Management:

Each consumer group should be configured to process only its specific topic, ensuring no overlap.

#### Closing Summary

By separating the Kafka topics and corresponding Spark jobs for different signal types (metrics, logs, traces), we ensure that adding a new type of data does not disrupt existing pipelines. This modular design allows each pipeline to evolve independently without requiring changes to other components.

#### Common Mistakes/Follow-Ups
Mistake: Mixing up the schemas and data models between topics.

#### Follow-Up
How would you handle schema evolution in Kafka if the trace format needs to change over time? Consider using Avro or Protobuf for schema compatibility.
Further Discussion: Discussing resilience strategies like checkpointing, fault tolerance, and state management.

**Comparison:**

```
Topic	Partitions	Replication Factor
metrics	4	3
logs	4	3
traces	4	3
```

**Code:**

```python
from pyspark.sql import SparkSession

# Initialize the Spark session
spark = SparkSession.builder \
    .appName("MetricsProcessor") \
    .getOrCreate()

# Define the schema for metrics
metrics_schema = "timestamp:long, metric_name:string, value:double"

# Start a structured streaming job to process metrics
df_metrics = (spark.readStream
              .format("kafka")
              .option("kafka.bootstrap.servers", "localhost:9092")
              .option("subscribe", "metrics")
              .option("startingOffsets", "earliest")
              .load()
              .selectExpr("CAST(value AS STRING) as json_value")
              .writeStream
              .format("console")
              .option("truncate", False)
              .outputMode("append")
              .start())

df_metrics.awaitTermination()

# Repeat similar steps for logs and traces topics.
```

**Example input / output:**

```
Input: Data streams into metrics, logs, and traces Kafka topics.
Output: Each Spark job processes its respective topic's data independently.
PLAINTEXT
Copy
# Example input to metrics topic
{
  "timestamp": 1632750943,
  "metric_name": "cpu_usage",
  "value": 85.5
}

# Example output from the metrics processing job
timestamp: 1632750943, metric_name: cpu_usage, value: 85.5 processed successfully.
```

**Data:**

```json
{
  "metrics": {
    "partitions": 4,
    "replication_factor": 3
  },
  "logs": {
    "partitions": 4,
    "replication_factor": 3
  },
  "traces": {
    "partitions": 4,
    "replication_factor": 3
  }
}
```

**Flow:**

1. metrics topic
2. metrics processing job
3. logs topic
4. logs processing job
5. traces topic
6. traces processing job
7. processed metrics data
8. processed logs data
9. processed traces data

