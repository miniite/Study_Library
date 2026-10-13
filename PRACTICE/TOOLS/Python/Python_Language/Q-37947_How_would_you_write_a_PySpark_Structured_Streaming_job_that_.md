---
type: "practice"
difficulty: "Deep"
---

# How would you write a PySpark Structured Streaming job that reads from Kafka, and what does a checkpoint location give you for exactly-once-ish fault tolerance?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Kafka Consumers in Python](<../../../../TOOLS/Python/Python_Language/Kafka_Consumers_in_Python.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To write a PySpark Structured Streaming job that reads from Kafka with exactly-once-ish fault tolerance, we need to understand the core concepts of Structured Streaming and Kafka's processing guarantees. The checkpoint location is crucial as it provides state persistence, ensuring that the job can recover its state in case of failures.

#### Triaging the Question

The question is probing your understanding of:

PySpark Structured Streaming: Its architecture, how it processes data streams.
Kafka Integration: How to read from Kafka topics and handle messages.
Exactly-Once Processing Guarantee: The nuances of fault tolerance in streaming applications.
Step-by-Step Solution

Firstly, PySpark Structured Streaming processes data in batches but operates on a continuous stream of data. For exactly-once processing, we need to ensure that each message is processed only once, even if the system fails and needs to recover from a checkpoint.

Setup
Kafka Integration: Kafka producers write messages to topics, which are partitioned for parallel consumption.
PySpark Structured Streaming Job: We define a streaming query that reads from Kafka topics using the readStream API and writes results back to another sink (e.g., another topic or a database).
Code Example
PYTHON
from pyspark.sql import SparkSession

# Initialize Spark session with necessary configurations for Structured Streaming
spark = SparkSession.builder \
    .appName("KafkaToSparkStructuredStreaming") \
    .getOrCreate()

# Define the Kafka source and set properties including checkpoint location
kafka_source = "kafkaSource"
checkpoint_location = "/tmp/checkpoint"

df = (
    spark.readStream.format("kafka")
        .option("kafka.bootstrap.servers", "localhost:9092")
        .option("subscribe", kafka_source)
        .load()
)

# Apply transformations and output the stream
query = df.writeStream \
    .format("console")  # For simplicity, writing to console; could be other sinks like HDFS, etc.
    .trigger(processingTime="1 second")  # Processing trigger time
    .option("checkpointLocation", checkpoint_location)  # Mandatory for exactly-once processing
    .start()

query.awaitTermination()
Checkpoint Location

A checkpoint location is a directory where Spark stores the state of your streaming job. This is essential for exactly-once processing because:

State Persistence: It allows the system to recover the state from a known point in time if a failure occurs.
Exactly-Once Processing Guarantee: By tracking the offsets and state, it ensures that each message is processed only once.
Why Checkpoint Location Matters
Fault Tolerance: If the job fails, it can resume exactly where it left off using the checkpointed state.
Avoiding Duplicates: Ensures that messages are not reprocessed unnecessarily, maintaining data integrity and consistency.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, to achieve exactly-once processing in a PySpark Structured Streaming job reading from Kafka:

Use writeStream with appropriate transformations.
Set the checkpointLocation to enable state persistence.
Ensure that your streaming query is triggered at regular intervals.

#### Common mistakes
include forgetting to set the checkpoint location or misconfiguring it, leading to potential data duplication issues. Additionally, ensuring that your Kafka cluster and Spark setup are properly configured for fault tolerance can also be a critical aspect of this implementation.

#### Follow-ups
might involve discussing how to handle different types of failures (e.g., permanent vs. transient), or exploring alternative approaches like using a database sink with ACID transactions.

**Comparison:**

```
Parameter	Description
kafka.bootstrap.servers	Comma-separated list of host and port pairs that the consumer should contact to find initial brokers.
subscribe	The name(s) of the Kafka topic(s).
checkpointLocation	Path in HDFS or local file system where Spark will store the checkpoints for this stream processing application.
```

**Code:**

```python
from pyspark.sql import SparkSession

# Initialize Spark session with necessary configurations for Structured Streaming
spark = SparkSession.builder \
    .appName("KafkaToSparkStructuredStreaming") \
    .getOrCreate()

# Define the Kafka source and set properties including checkpoint location
kafka_source = "kafkaSource"
checkpoint_location = "/tmp/checkpoint"

df = (
    spark.readStream.format("kafka")
        .option("kafka.bootstrap.servers", "localhost:9092")
        .option("subscribe", kafka_source)
        .load()
)

# Apply transformations and output the stream
query = df.writeStream \
    .format("console")  # For simplicity, writing to console; could be other sinks like HDFS, etc.
    .trigger(processingTime="1 second")  # Processing trigger time
    .option("checkpointLocation", checkpoint_location)  # Mandatory for exactly-once processing
    .start()

query.awaitTermination()
```

**Example input / output:**

```text
Input:
Stream of messages from Kafka topic

Output (example):
Structured DataFrame with processed data, printed to console or written to another sink.
```

**Data:**

```json
{
  "kafka_source": "topicName",
  "checkpoint_location": "/tmp/checkpoint"
}
```

**Flow:**

1. Yes
2. No
3. Read from Kafka
4. Transform Data
5. Write to Console/Other Sink
6. Failed?
7. Resume from Checkpoint
8. Continue Processing

