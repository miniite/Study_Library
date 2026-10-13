---
type: "practice"
difficulty: "Deep"
---

# How would you implement alert suppression/dedup windows in Spark so a flapping service doesn't generate 500 pages, without hiding a genuine second incident?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Streaming Recovery and Deduplication](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Streaming_Recovery_and_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement alert suppression or deduplication windows in Apache Spark for managing flapping services effectively, we need to ensure that repeated alerts from the same source within a certain time window are suppressed, while allowing new unique incidents to trigger notifications. This is crucial to avoid overwhelming operators with noise and ensuring they receive relevant alerts.

#### Triage

The question requires us to design an alert suppression mechanism in Spark that can handle flapping services (services generating frequent alerts) without missing genuine second incidents. The core aspects include:

Defining a window of time within which repeated alerts are considered duplicates.
Differentiating between repeat alerts and new, unique incidents.

#### First Principles Solution

Define the Deduplication Window:

We need to set up a sliding window that tracks alert events from the same source.

Identify Repeat Alerts:

Use Kafka for real-time ingestion of alert data and a Spark Streaming context to process these events in real time.
Apply a window operation to group alerts by their source within the defined window.

Suppression Logic:

Suppress repeated alerts from the same source if they occur within the deduplication window, but allow new incidents to trigger notifications.

Handling Outliers:

Ensure that true second incidents are not suppressed and still generate alerts.

State Management:

Use Spark's stateful processing capabilities (e.g., using updateStateByKey in DStream) to maintain the state of each alert source across batches.

#### Implementation
Steps

Setup Kafka for Ingestion:

BASH
kafka-topics.sh --create --topic alerts-topic --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1

Spark Streaming Setup with Deduplication Logic:

Define the Spark streaming context and set up a DStream to consume from Kafka.
Use map transformations to parse incoming JSON alerts.
SCALA
import org.apache.spark.SparkConf
import org.apache.spark.streaming.{Seconds, StreamingContext}
import org.apache.kafka.common.serialization.StringDeserializer
import org.apache.spark.streaming.kafka010.ConsumerStrategies.Subscribe
import org.apache.spark.streaming.kafka010.KafkaUtils
import org.apache.spark.streaming.kafka010.LocationStrategies.PreferConsistent

val conf = new SparkConf().setAppName("AlertSuppression")
val ssc = new StreamingContext(conf, Seconds(5))
ssc.checkpoint("path/to/checkpoint")

val kafkaParams = Map[String, Object] (
  "bootstrap.servers" -> "localhost:9092",
  "group.id" -> "alert-suppression-group",
  "key.deserializer" -> classOf[StringDeserializer],
  "value.deserializer" -> classOf[StringDeserializer],
  "auto.offset.reset" -> "latest",
  "enable.auto.commit" -> (false: java.lang.Boolean)
)

val topics = List("alerts-topic")
val stream = KafkaUtils.createDirectStream(
  ssc,
  PreferConsistent,
  Subscribe[String, String] (topics, kafkaParams)
).map(record => (record.key(), record.value()))
Apply Deduplication Logic with Stateful Processing:
Use updateStateByKey to maintain state for each alert source.
Define a window of time within which alerts are considered duplicates.
SCALA
val dedupWindow = Seconds(60)
val dedupKeyFunc = (k: String) => k // Map key by service name

stream.map(_._2)
  .map(alert => (alert.serviceName, alert.timestamp))
  .keyBy(dedupKeyFunc)
  .updateStateByKey[(Option[Long], Int)] ((newAlert: (String, Long), oldStateOpt: Option[(Option[Long), Int)]) => {
    val (oldTimestampOpt, count) = oldStateOpt.getOrElse((None, 0))

    // Check if the new alert is within the deduplication window
    if (oldTimestampOpt.exists(t => t + dedupWindow >= newAlert._2)) {
      Some((Some(newAlert._2), count + 1))
    } else {
      Some((Some(newAlert._2), 0))
    }
  })
Output New Alerts:
Filter out repeated alerts and output unique incidents.
SCALA
stream.map(_._2)
  .map(alert => (alert.serviceName, alert.timestamp))
  .keyBy(dedupKeyFunc)
  .updateStateByKey[(Option[Long], Int)] ((newAlert: (String, Long), oldStateOpt: Option[(Option[Long), Int)]) => {
    val (oldTimestampOpt, count) = oldStateOpt.getOrElse((None, 0))

    // Check if the new alert is within the deduplication window
    if (oldTimestampOpt.exists(t => t + dedupWindow >= newAlert._2)) {
      Some((Some(newAlert._2), count + 1))
    } else {
      Some((Some(newAlert._2), 0))
    }
  })
  .filter(_._2._2 == 0) // Only emit alerts with a count of 0
  .foreachRDD { rdd =>
    val results = rdd.collect()
    println(s"New Alerts: $results")
  }

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing this solution, we ensure that repeated alerts are suppressed while genuine new incidents still generate notifications. This is achieved through a combination of Kafka for real-time data ingestion and Spark's stateful processing capabilities.

#### Common Mistakes

Not setting up the deduplication window correctly.
Failing to maintain state properly across batches.
Ignoring edge cases such as alert sources that are not unique.

#### Follow-Ups
How would you handle different severity levels of alerts?
What if the service name is not a fixed field in the alert data?
How do you ensure the solution scales with increasing data volume?

**Comparison:**

```
Time	Alert Service Name	Timestamp	Level
12:00	example-service	2023-10-01T12:00:00Z	CRITICAL
12:05	example-service	2023-10-01T12:05:00Z	CRITICAL
```

**Code:**

```scala
import org.apache.spark.SparkConf
import org.apache.spark.streaming.{Seconds, StreamingContext}
import org.apache.kafka.common.serialization.StringDeserializer
import org.apache.spark.streaming.kafka010.ConsumerStrategies.Subscribe
import org.apache.spark.streaming.kafka010.KafkaUtils
import org.apache.spark.streaming.kafka010.LocationStrategies.PreferConsistent

val conf = new SparkConf().setAppName("AlertSuppression")
val ssc = new StreamingContext(conf, Seconds(5))
ssc.checkpoint("path/to/checkpoint")

val kafkaParams = Map[String, Object](
  "bootstrap.servers" -> "localhost:9092",
  "group.id" -> "alert-suppression-group",
  "key.deserializer" -> classOf[StringDeserializer],
  "value.deserializer" -> classOf[StringDeserializer],
  "auto.offset.reset" -> "latest",
  "enable.auto.commit" -> (false: java.lang.Boolean)
)

val topics = List("alerts-topic")
val stream = KafkaUtils.createDirectStream(
  ssc,
  PreferConsistent,
  Subscribe[String, String](topics, kafkaParams)
).map(record => (record.key(), record.value()))

val dedupWindow = Seconds(60)
val dedupKeyFunc = (k: String) => k

stream.map(_._2)
  .map(alert => (alert.serviceName, alert.timestamp))
  .keyBy(dedupKeyFunc)
  .updateStateByKey[(Option[Long], Int)]((newAlert: (String, Long), oldStateOpt: Option[(Option[Long), Int)]) => {
    val (oldTimestampOpt, count) = oldStateOpt.getOrElse((None, 0))

    if (oldTimestampOpt.exists(t => t + dedupWindow >= newAlert._2)) {
      Some((Some(newAlert._2), count + 1))
    } else {
      Some((Some(newAlert._2), 0))
    }
  })
  .filter(_._2._2 == 0)
  .foreachRDD { rdd =>
    val results = rdd.collect()
    println(s"New Alerts: $results")
  }

ssc.start()
ssc.awaitTermination()
```

**Example input / output:**

```text
New Alerts: [
  {"service_name": "example-service", "timestamp": "2023-10-01T12:05:00Z", "level": "CRITICAL"}
]
```

**Data:**

```json
{
  "service_name": "example-service",
  "timestamp": "2023-10-01T12:00:00Z",
  "level": "CRITICAL"
}
```

**Flow:**

1. Yes
2. No
3. Real-time Alert Stream
4. Kafka
5. Is Service Name Unique?
6. Process Alert
7. Deduplicate Alerts in Window
8. Filter Out Repeats
9. Output New Incidents toOperator

