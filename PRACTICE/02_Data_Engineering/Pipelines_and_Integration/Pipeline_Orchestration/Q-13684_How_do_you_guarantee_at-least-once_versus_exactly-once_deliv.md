---
type: "practice"
difficulty: "Deep"
---

# How do you guarantee at-least-once versus exactly-once delivery from a Spark streaming job into Kafka and where does idempotence come from?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Exactly Once Spark Writes](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Exactly_Once_Spark_Writes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure at-least-once and exactly-once semantics in data pipelines using Apache Spark with integration to Apache Kafka, we need to understand the core concepts of idempotence and how they interact with streaming jobs and messaging systems. This is crucial for maintaining data integrity, especially in mission-critical applications.

First, let's clarify what we're dealing with:

At-Least-Once Delivery: Ensures that each message is delivered at least once.
Exactly-Once Delivery: Guarantees that each message is processed exactly once.

#### Triage

The question probes your understanding of idempotence and how it can be leveraged in Spark streaming jobs to achieve reliable data processing. We'll explore the implications for Kafka, which typically offers at-least-once delivery by default but requires additional configurations to support exactly-once semantics.

#### First-Principles Solution
1. Understanding Idempotence

Idempotence is a property where multiple identical requests have the same effect as a single request. In the context of data processing, this means that reprocessing the same input should not result in different outcomes. This is crucial for ensuring consistency when implementing exactly-once delivery.

2. Kafka's At-Least-Once Delivery

Apache Kafka inherently supports at-least-once delivery because messages are processed by consumers as long as they are present on the topic, even if a consumer crashes before processing all messages.

3. Exactly-Once Semantics in Spark with Kafka

To achieve exactly-once semantics from a Spark streaming job to Kafka, we need to ensure that:

Each message is only processed once.
No data loss occurs during failures.
4. Implementing Exactly-Once Delivery

The key steps involve:

Using Checkpointing for Spark jobs to maintain state and recover from failures.
Configuring Kafka producers to enable idempotent writes, which ensures that a producer can retry sending the same message without duplication.
Ensuring that both Kafka and Spark are configured correctly.
Detailed Steps

Enable Idempotent Writes in Kafka Producers

Configure the enable.idempotence property in your Kafka producer settings to true. This ensures that messages will be retried if they fail, but only one copy of the message will be delivered.

#### Example
configuration:
PROPERTIES
enable.idempotence=true

Set Up Spark Streaming with Checkpointing

Enable checkpointing in your Spark streaming job to maintain state and recover from failures. This is essential for ensuring that messages are processed exactly once.
Use the recovery option when setting up your Spark session:
SCALA
spark.conf.set("spark.streaming.recoveryMode", "checkpoint")

Ensure Exactly-Once Semantics in Kafka

Enable transactional IDs in Kafka to support exactly-once semantics by enabling transactional.id and setting enable.idempotence=true.

#### Example
configuration:
PROPERTIES
transactional.id=<some-unique-id>
enable.idempotence=true

Handling Failures Gracefully

Implement proper error handling in your Spark streaming job to ensure that partial failures do not lead to inconsistent states.
Use try-catch blocks and logging to capture and handle errors.

#### Closing Summary

By enabling idempotent writes in Kafka producers, setting up checkpointing in Spark, and ensuring correct configurations for both systems, you can achieve exactly-once delivery from a Spark streaming job into Kafka. This approach ensures that data is processed reliably and consistently, even in the face of failures.

#### Common Mistakes
and Follow-Ups
Mistake: Failing to enable idempotent writes in Kafka producers.

#### Follow-Up
Always ensure that enable.idempotence=true for reliable message delivery.
Mistake: Not configuring checkpointing in Spark streaming jobs.

#### Follow-Up
Use the recoveryMode setting to enable stateful recovery and avoid partial processing during restarts.
Mistake: Overlooking transactional IDs in Kafka.

#### Follow-Up
Ensure that you configure transactional IDs if you are working with distributed transactions.

**Comparison:**

```
Step	Configuration
1	Kafka Producer enable.idempotence = true
2	Kafka Topic Transactional ID configured
3	Spark Streaming Checkpointing enabled
4	Proper Error Handling in Code
```

**Code:**

```scala
// Example Spark Streaming Job Configuration for Exactly-Once Delivery
import org.apache.spark.sql.SparkSession

object ExactlyOnceSparkJob {
  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder()
      .appName("ExactlyOnceSparkJob")
      .config("spark.streaming.recoveryMode", "checkpoint")
      .getOrCreate()

    // Your data processing logic here
    spark.readStream
      .format("kafka")
      .option("kafka.bootstrap.servers", "<your-kafka-broker>")
      .option("subscribe", "your-topic")
      .load()
      .writeStream
      .toKafka("<your-kafka-broker>", "output-topic")
      .start()
      .awaitTermination()
  }
}
```

**Example input / output:**

| Kafka Producer Config | Spark Streaming Config |
|-----------------------|------------------------|
| enable.idempotence=true, transactional.id=<some-unique-id> | checkpointDir=/path/to/checkpoint, recoveryMode=checkpoint |

**Data:**

```json
{
  "kafkaProducerConfig": {
    "enable.idempotence": true,
    "transactional.id": "<some-unique-id>"
  },
  "sparkStreamingConfig": {
    "checkpointDir": "/path/to/checkpoint",
    "recoveryMode": "checkpoint"
  }
}
```

**Flow:**

1. Yes
2. No
3. Enable Idempotent Writes
4. Is Exactly-Once Required?
5. Configure Kafka and Spark
6. Simple At-Least-Once Delivery
7. Ensure Stateful Recovery withCheckpointing in Spark
8. Handle Errors Gracefully inCode

