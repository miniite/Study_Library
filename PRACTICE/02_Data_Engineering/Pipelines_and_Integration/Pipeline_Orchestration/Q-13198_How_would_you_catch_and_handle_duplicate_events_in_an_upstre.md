---
type: "practice"
difficulty: "Deep"
---

# How would you catch and handle duplicate events in an upstream Kafka topic before they inflate your Spark-computed count features?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Exactly Once Spark Writes](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Exactly_Once_Spark_Writes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To catch and handle duplicate events in a Kafka topic before they cause issues with Spark computations, we need to implement a robust deduplication mechanism that leverages both Kafka’s capabilities and Spark's processing power. The key idea is to ensure that each unique event is processed only once, thereby avoiding double counting or other inconsistencies.

#### Triaging the Question

The question is essentially about designing an efficient pipeline to handle duplicate events in a Kafka topic before they reach a downstream Spark application for further processing. This involves understanding how Kafka handles messages and leveraging its KafkaConsumer configurations along with Spark’s capabilities to ensure data integrity.

#### Assumptions
The Kafka topic has a large volume of events, possibly containing duplicates.
The goal is to prevent these duplicates from affecting the accuracy of any downstream processing using Spark.

#### First-Principles Solution
Step 1: Configure Kafka for Deduplication

Kafka can be configured to treat messages as unique based on their key or value. This is achieved by setting appropriate properties in the KafkaConsumer configuration.

PROPERTIES
# Example Kafka consumer configurations
bootstrap.servers=kafka-broker-0.kafka-headless.default.svc.cluster.local:9092,kafka-broker-1.kafka-headless.default.svc.cluster.local:9092,kafka-broker-2.kafka-headless.default.svc.cluster.local:9092
group.id=dedup-group
auto.offset.reset=earliest
enable.auto.commit=false
key.deserializer=org.apache.kafka.common.serialization.StringDeserializer
value.deserializer=org.apache.kafka.common.serialization.StringDeserializer

By setting enable.auto.commit=false, we can manually control the offset commits, which is useful for processing messages in a deduplication stream.

Step 2: Implement Deduplication Logic Using Spark Structured Streaming

We need to implement logic within our Spark application to detect and handle duplicates. This involves using the Window function to identify unique keys over a sliding window of time or events, ensuring that each key is processed only once during this period.

Here’s how we can achieve this in PySpark:

PYTHON
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, row_number, window

# Initialize Spark session
spark = SparkSession.builder.appName("DeduplicationPipeline").getOrCreate()

# Read from Kafka
df = spark.readStream.format("kafka") \
    .option("kafka.bootstrap.servers", "kafka-broker-0,kafka-broker-1,kafka-broker-2:9092") \
    .option("subscribe", "your-topic-name") \
    .option("startingOffsets", "earliest") \
    .load()

# Assuming the Kafka message has a key and value
df = df.selectExpr("CAST(value AS STRING)")

# Define window specification (e.g., 10 seconds)
window_spec = window("event_timestamp", "10 seconds")

# Deduplicate based on key within a window
df_dedup = df.withColumn("row_num", row_number().over(window_spec)) \
             .filter(col("row_num") == 1) \
             .drop("row_num")

# Perform further processing (e.g., aggregation)
output_df = df_dedup.groupBy("key").agg({"value": "count"})

# Write to console for demonstration
query = output_df.writeStream.outputMode("complete").format("console").start()

query.awaitTermination()
Step 3: Monitoring and Logging

Ensure proper monitoring of the deduplication process. Use tools like Prometheus or Grafana to monitor Kafka lag, Spark job status, and potential failures.

#### Closing Summary

By configuring Kafka with appropriate deserializers and manually controlling offsets, and by implementing a deduplication logic within our Spark application using window functions, we can effectively prevent duplicates from inflating the count features. This solution ensures data integrity while leveraging the strengths of both technologies.

#### Common Mistakes/Follow-Ups
Forgetting to Manually Control Offsets: Kafka’s auto-commit feature might commit offsets before all messages are processed.
Improper Window Specification: Incorrect window settings can lead to incorrect deduplication or unnecessary processing.
Insufficient Monitoring: Not setting up adequate monitoring and logging can make troubleshooting difficult.

By addressing these points, we ensure a robust solution that handles duplicates effectively in a complex data pipeline.

**Comparison:**

| Event ID | User ID | Action          | Timestamp             |
|----------|---------|-----------------|----------------------|
| 1        | 12345   | login           | 2023-10-10T10:00:00Z  |
| 2        | 12345   | logout          | 2023-10-10T10:05:00Z  |  # Duplicate

**Code:**

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, row_number, window

# Initialize Spark session
spark = SparkSession.builder.appName("DeduplicationPipeline").getOrCreate()

# Read from Kafka
df = spark.readStream.format("kafka") \
    .option("kafka.bootstrap.servers", "kafka-broker-0,kafka-broker-1,kafka-broker-2:9092") \
    .option("subscribe", "your-topic-name") \
    .option("startingOffsets", "earliest") \
    .load()

# Assuming the Kafka message has a key and value
df = df.selectExpr("CAST(value AS STRING)")

# Define window specification (e.g., 10 seconds)
window_spec = window("event_timestamp", "10 seconds")

# Deduplicate based on key within a window
df_dedup = df.withColumn("row_num", row_number().over(window_spec)) \
             .filter(col("row_num") == 1) \
             .drop("row_num")

# Perform further processing (e.g., aggregation)
output_df = df_dedup.groupBy("key").agg({"value": "count"})

# Write to console for demonstration
query = output_df.writeStream.outputMode("complete").format("console").start()

query.awaitTermination()
```

**Example input / output:**

```

Input: Kafka message stream with key-value pairs. Output: Deduplicated and processed data, ensuring no duplicates are counted twice.

Request:

CODE
Copy
+----------+-------------------+
| user_id  | event_data        |
+----------+-------------------+
| 12345    | {"action": "login"}|
| 12345    | {"action": "logout"|  # Duplicate
+----------+-------------------+

Output:

+----------+------------------+ | user_id | event_count | +----------+------------------+ | 12345 | 1 | +----------+------------------+
```

**Data:**

```json
{
  "kafka_message_key": "user_id",
  "kafka_message_value": "event_data"
}
```

**Flow:**

1. Message
2. Deduplicated Events
3. Processed Events
4. Output
5. Consume from Kafka Topic
6. Spark Structured StreamingApp
7. Deduplicate Events
8. Process Aggregated Data
9. Monitor Pipeline

