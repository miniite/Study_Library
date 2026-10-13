---
type: "practice"
difficulty: "Deep"
---

# A CDC pipeline via Debezium and Kafka is duplicating updates in the warehouse after a connector restart; where do the dupes come from and how do you make the sink tolerant?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Streaming Recovery and Deduplication](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Streaming_Recovery_and_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with CDC (Change Data Capture) pipelines using Debezium and Kafka, duplication issues can arise due to several reasons, primarily stemming from the way the system handles state during connector restarts or failures. To effectively address this issue, we need to understand where these duplicates come from and implement strategies that make the sink tolerant to such scenarios.

#### Triage
Understanding the Problem

The core of the question is about ensuring data integrity in a CDC pipeline after a connector restart. Debezium is a powerful tool for extracting changes from databases, but it can introduce duplicates if not properly managed during stateful operations like checkpointing and recovery. The key aspects to consider are:

Checkpointing: How Debezium manages its position in the database stream.
Retry Mechanisms: How Kafka handles retries and message delivery guarantees.
Sink Tolerance: Ensuring that downstream systems (like a data warehouse) can handle duplicate records.

#### First-Principles Solution
1. Understanding Checkpointing

Debezium uses Kafka to deliver change events, which are checkpointed periodically to ensure recovery in case of failures. However, if the connector restarts between two checkpoints, it might process some changes twice. This is because the last known state (checkpoint) might be out of date when the connector resumes.

2. Message Deduplication at Sink

To avoid duplicates at the sink level, we need to implement deduplication mechanisms that are robust and efficient. This can be achieved through various strategies:

Event Timestamps: Using a unique timestamp or sequence number for each event.
Message Acknowledgment: Ensuring messages are only processed once by using Kafka's exactly-once semantics.
3. Implementing Tolerant Sinks

A tolerant sink should handle duplicate records gracefully without causing data integrity issues:

Duplicate Detection: Use a unique identifier or hash of the message content to detect duplicates.
Conflict Resolution: Define clear policies for handling conflicts, such as updating only if the latest record is newer.
Detailed Steps and Solution
Step 1: Configure Debezium Checkpointing

Ensure that your Kafka topic configured with Debezium has proper checkpointing enabled. This involves setting up a reliable state store like ZooKeeper or a database to keep track of offsets:

PROPERTIES
# Example configuration snippet for Kafka Connector
connector.class=io.debezium.connector.mysql.MySqlConnector
tasks.max=1
offset.storage=kafka OFFSET_STORAGE
offset.flush.interval.ms=5000
Step 2: Implement Deduplication at the Sink

Use a combination of timestamps and unique identifiers to detect duplicates. For example, in Spark:

PYTHON
from pyspark.sql.functions import monotonically_increasing_id

# Read from Kafka topic
df = spark.readStream.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "changes_topic") \
    .load()

# Add a unique identifier for each message
df = df.withColumn("message_id", monotonically_increasing_id())

# Deduplicate using message ID and timestamp
deduped_df = df.groupBy("message_id").agg({"timestamp": "max"}).join(df, ["message_id", "timestamp"], "inner")

# Process deduplicated data
result = deduped_df.selectExpr("value", "timestamp")
Step 3: Define Conflict Resolution Policies

Ensure that your sink writes to the database with proper conflict resolution policies. For example, in a transactional context:

SQL
-- Example SQL for upsert operation
INSERT INTO target_table (id, value)
VALUES (?, ?)
ON DUPLICATE KEY UPDATE
    id = VALUES(id),
    value = VALUES(value);

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, the key steps involve setting up robust checkpointing with Debezium, implementing message deduplication at the sink level, and defining clear conflict resolution policies. Common mistakes include insufficient state management, not handling duplicates properly, and ignoring transactional integrity during writes.

#### Follow-up
questions might revolve around:

What are some alternative methods for implementing exactly-once semantics?
How would you handle large-scale data volumes with this setup?

**Comparison:**

```
Topic	Partitions	Offsets
changes_topic	2	[12345, 67890]
```

**Code:**

```python
from pyspark.sql.functions import monotonically_increasing_id

# Example code snippet for deduplication in Spark
df = spark.readStream.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "changes_topic") \
    .load()

df = df.withColumn("message_id", monotonically_increasing_id())
deduped_df = df.groupBy("message_id").agg({"timestamp": "max"}).join(df, ["message_id", "timestamp"], "inner")
result = deduped_df.selectExpr("value", "timestamp")
```

**Example input / output:**

```

Input: Kafka topic with change events Output: Deduplicated data stream processed by Spark Request: spark.readStream.format("kafka").option("subscribe", "changes_topic").load() Response:

PYTHON
Copy
result = deduped_df.selectExpr("value", "timestamp")
```

**Data:**

```json
{
  "topic": "changes_topic",
  "offsets": [
    {"partition": 0, "offset": 12345},
    {"partition": 1, "offset": 67890}
  ]
}
```

**Flow:**

1. Debezium
2. Kafka
3. Sink (Spark)
4. Deduplicate & Process

