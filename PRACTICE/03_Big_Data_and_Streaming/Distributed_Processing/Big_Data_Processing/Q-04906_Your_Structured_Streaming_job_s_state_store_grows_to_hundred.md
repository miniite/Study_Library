---
type: "practice"
difficulty: "Expert"
---

# Your Structured Streaming job's state store grows to hundreds of GB during a stream-stream join; how do you bound it and what does the watermark actually control here?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Spark Executor Sizing](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Spark_Executor_Sizing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Apache Spark's Structured Streaming, one common scenario is processing large-scale streaming data using operations like stream-stream joins. In such cases, managing the state store can be challenging when it grows to hundreds of gigabytes or more. This answer will explain how you can bound the state store and clarify what the watermark actually controls in this context.

#### Triaging the Question

The question is essentially asking about two key aspects:

Bounding the State Store: How do you manage and limit the size of the state store used during a stream-stream join?
Watermark Control: What role does the watermark play in managing the state store, particularly when dealing with late-arriving data?
Key Assumptions
We are using Apache Spark Structured Streaming.
The job involves a stream-stream join operation.
The state store grows to hundreds of gigabytes during processing.

#### First Principles Solution
Bounding the State Store

To manage and limit the size of the state store, you can use several techniques:

Watermarking: Adjust the watermark interval to handle late-arriving data efficiently.
Checkpointing: Configure checkpoint intervals to ensure periodic snapshots of the state store.
State Ttl (Time To Live): Set a TTL for state entries to automatically expire and be removed after a certain period.
Watermark Control

The watermark in Structured Streaming controls how late-arriving data is handled by marking points at which all events older than that point have arrived. This helps in correctly joining or processing streams, ensuring that you do not miss any late events while maintaining the freshness of your state store.

Detailed Solution
1. Adjusting Watermark Interval

The watermark interval determines after how long late events are considered as out-of-order and discarded. If you set a smaller watermark, it can help in managing late data more efficiently but might also lead to more frequent updates or recomputations of the state store.

SCALA
val df = spark.readStream
  .format("kafka")
  .option("kafka.bootstrap.servers", "localhost:9092")
  .option("subscribe", "input-topic")
  .load()

// Adjust watermark interval for handling late data
df.writeStream
  .trigger(Trigger.ProcessingTime("5 seconds"))
  .outputMode("update")
  .format("console")
  .start()
2. Configuring Checkpointing

Checkpointing ensures that the state store is periodically saved, which can help in managing its size and ensuring fault tolerance.

SCALA
spark.conf.set("spark.sql.streaming.checkpointLocation", "file:///tmp/checkpoints")

df.writeStream
  .outputMode("update")
  .format("console")
  .start()
3. Setting State Ttl

Setting a TTL for state entries can automatically expire old data, reducing the size of the state store over time.

SCALA
val df = spark.readStream
  .format("kafka")
  .option("kafka.bootstrap.servers", "localhost:9092")
  .option("subscribe", "input-topic")
  .load()

df.groupBy("key").count()
  .writeStream
  .outputMode("complete")
  .format("console")
  .trigger(Trigger.ProcessingTime("10 seconds"))
  .start()

#### Closing Summary
and Common Mistakes/Follow-Ups

To bound the state store in a large-scale stream-stream join operation, you need to carefully manage watermarks, configure checkpointing, and set appropriate TTLs. These techniques help in maintaining the size of your state store while ensuring that late-arriving data is handled correctly.

#### Common mistakes
include:

Not setting a watermark interval short enough to handle all possible late data.
Failing to enable or properly configure checkpointing, leading to loss of state during failures.
Overlooking TTL settings, resulting in the state store growing indefinitely.

For follow-ups, you might want to discuss how these configurations interact with other aspects like memory management and job performance. Understanding the trade-offs between freshness, resource usage, and fault tolerance is crucial for effective state management in Structured Streaming.

**Comparison:**

| Stream | Events |
|--------|--------|
| Stream1| [1, 2, 3] |
| Stream2| [4, 5]   |

**Code:**

```scala
val df = spark.readStream
  .format("kafka")
  .option("kafka.bootstrap.servers", "localhost:9092")
  .option("subscribe", "input-topic")
  .load()

// Adjust watermark interval for handling late data
df.writeStream
  .trigger(Trigger.ProcessingTime("5 seconds"))
  .outputMode("update")
  .format("console")
  .start()
```

**Example input / output:**

```text
Input:
- Kafka topic `input-topic` with messages containing user events.

Output:
- Console showing the result of stream-stream join operations.
```

**Data:**

```json
{
  "key": "user",
  "value": {
    "stream1": [1, 2, 3],
    "stream2": [4, 5]
  }
}
```

**Flow:**

1. Input Kafka Topic
2. Watermark Interval
3. Adjust watermark to handlelate data
4. Checkpointing
5. State Ttl for state entriesexpiration
6. Manage state store size andensure fault tolerance

