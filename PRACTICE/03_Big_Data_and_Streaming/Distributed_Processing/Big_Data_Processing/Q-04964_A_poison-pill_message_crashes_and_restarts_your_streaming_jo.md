---
type: "practice"
difficulty: "Deep"
---

# A poison-pill message crashes and restarts your streaming job in a loop; how do you make the job skip-and-record it without manual intervention?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Streaming Recovery and Deduplication](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Streaming_Recovery_and_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a "poison-pill" message crashing and restarting a Spark Streaming job, we need to ensure that our system can detect such messages and handle them gracefully. The key idea is to design the pipeline so it can bypass problematic messages while still maintaining the integrity of the data processing flow.

#### Triage

The question is essentially about identifying and handling exceptional cases within a streaming data processing application built with Apache Kafka and Spark. We need to ensure that our job doesn't get stuck in an endless loop due to an unhandled message, often referred to as a "poison-pill" or an erroneous message. The interviewer is probing your understanding of stream processing, error handling strategies, and the ability to design robust systems.

Step-by-Step Solution
Define the Poison-Pill: A poison-pill message can be any type of message that should not proceed through the normal processing pipeline but instead trigger a specific action.
Detection Mechanism: Implement a mechanism within your Spark Streaming job to detect such messages early in the processing flow.
Exception Handling Logic: Design the logic to handle these messages, ensuring they do not cause the entire job to fail or restart unnecessarily.
Step 1: Define the Poison-Pill

Let's define a poison-pill message as one that contains invalid data. For example, a message might be marked with a specific key or value indicating it should be treated differently.

PYTHON
def is_poison_pill(message):
    # Example logic to identify poison-pills based on content
    return "POISON_PILL" in str(message.value)
Step 2: Detection Mechanism

We will use Apache Kafka's StreamListener or KafkaConsumer with a custom deserialization method to detect such messages. In Spark Streaming, we can leverage the foreachBatch function to process batches of data.

PYTHON
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("StreamingJob").getOrCreate()

def process_stream(batch_df, batch_id):
    # Check for poison-pill messages in the current batch
    poison_pills = batch_df.filter(is_poison_pill)

    if not poison_pills.isEmpty():
        print(f"Detected {poison_pills.count()} poison-pill(s) at batch {batch_id}")

        # Optionally, you can log or handle these messages separately
        # poison_pills.show()

        # Drop the problematic rows from further processing
        non_poison_pill_df = batch_df.subtract(poison_pills)

    else:
        non_poison_pill_df = batch_df

    # Continue with normal processing of valid data
    results = non_poison_pill_df.groupBy("column_name").count()

    return results

# Start the streaming query
query = (
    spark.readStream.format("kafka")
      .option("kafka.bootstrap.servers", "localhost:9092")
      .option("subscribe", "topic_name")
      .load()
      .writeStream.foreachBatch(process_stream)
      .outputMode("update")
      .start()
)

# Wait for the stream to finish
query.awaitTermination()
Step 3: Exception Handling Logic

In the above code, we filter out poison-pill messages and drop them from further processing. This prevents these invalid messages from propagating through the pipeline and causing downstream errors.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By implementing this approach, we ensure that our streaming job can gracefully handle exceptional cases without getting stuck in an endless loop of restarts due to a single problematic message. The key is in early detection and swift action to bypass such messages.

#### Common mistakes
include:

Failing to define or detect the poison-pill condition accurately.
Not handling the dropped rows effectively, leading to data loss.
Overly complex logic that can introduce additional errors.

#### Follow-ups
could involve discussing how you would integrate this solution with other error-handling strategies like retries, dead-letter queues, or monitoring systems for better fault tolerance and observability in your streaming job.

**Code:**

```python
from pyspark.sql import SparkSession

def is_poison_pill(message):
    # Example logic to identify poison-pills based on content
    return "POISON_PILL" in str(message.value)

spark = SparkSession.builder.appName("StreamingJob").getOrCreate()

def process_stream(batch_df, batch_id):
    poison_pills = batch_df.filter(is_poison_pill)

    if not poison_pills.isEmpty():
        print(f"Detected {poison_pills.count()} poison-pill(s) at batch {batch_id}")

        # Optionally, you can log or handle these messages separately
        # poison_pills.show()

        non_poison_pill_df = batch_df.subtract(poison_pills)
    else:
        non_poison_pill_df = batch_df

    results = non_poison_pill_df.groupBy("column_name").count()

    return results

query = (
    spark.readStream.format("kafka")
      .option("kafka.bootstrap.servers", "localhost:9092")
      .option("subscribe", "topic_name")
      .load()
      .writeStream.foreachBatch(process_stream)
      .outputMode("update")
      .start()
)

query.awaitTermination()
```

**Flow:**

1. Yes
2. No
3. Read from Kafka
4. Is Poison-Pill?
5. Dropped and Log
6. Proceed to Normal Processing
7. End Batch Processing
8. Process Data
9. Output Results

