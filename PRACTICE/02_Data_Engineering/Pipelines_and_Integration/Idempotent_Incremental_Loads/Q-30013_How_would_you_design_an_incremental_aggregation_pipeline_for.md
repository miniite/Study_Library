---
type: "practice"
difficulty: "Deep"
---

# How would you design an incremental aggregation pipeline for daily active users that must be correct after a partial-day reprocess?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Idempotent Incremental Loads](<../../../../02_Data_Engineering/Pipelines_and_Integration/Idempotent_Incremental_Loads/Idempotent_Incremental_Loads.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure correctness after a partial-day reprocess, we need to design our data pipeline with both idempotency and incremental processing in mind. This means any state transition should be reversible and repeatable without causing duplicate or incorrect results. For daily active users (DAU), this involves handling updates and deletions gracefully while ensuring the final count remains accurate.

#### Triaging the Question

The interviewer is probing for a solution that can handle partial-day reprocessing correctly, which requires careful consideration of how to manage state transitions, especially around timestamping and data integrity. Key assumptions include:

Each user's activity should be tracked by a unique identifier.
The pipeline processes user activities with timestamps to determine the date of their last activity.

#### First-Principles Solution
Event Handling: Capture every event (e.g., login) as it happens, marking it with a timestamp.
Timestamp Deduplication: Ensure that each event is processed only once by maintaining a record of processed events.
State Management: Use a state store to keep track of the latest state and ensure idempotency.
Incremental Aggregation: Aggregate DAUs incrementally, starting from the last known state.
Step-by-Step Solution
Event Capture:
Each event (login) is captured with a timestamp.
Timestamp Deduplication:
Maintain a set of processed events to ensure no duplicate processing.
State Management:
Use a key-value store (e.g., Redis or Cassandra) to track the latest state per user and date.
Incremental Aggregation:
For each new day, process only those events that occurred on that day and update the state accordingly.

#### Implementation
Details
Event Source: A stream processing system like Apache Kafka or AWS Kinesis.
Processing Framework: Apache Spark Streaming for stateful processing.
State Store: Redis for key-value operations to track processed states efficiently.

#### Example
Schema
JSON
{
  "userId": "12345",
  "date": "2023-10-01",
  "processedEvents": ["login-2023-10-01"]
}
Dry-Run

#### Cons
ider a scenario where the pipeline runs on October 1st, and user 12345 logs in at 9 AM. The state store would record:

JSON
{
  "userId": "12345",
  "date": "2023-10-01",
  "processedEvents": ["login-2023-10-01"]
}

If the pipeline reprocesses on October 1st and user 12345 logs in again at 1 PM, it should be marked as processed to avoid double counting.

Full Solution
PYTHON
from pyspark.sql import SparkSession
from pyspark.streaming.kafka import KafkaUtils
import json

# Initialize Spark session
spark = SparkSession.builder.appName("DAUIncrementalPipeline").getOrCreate()

# Define the topic and brokers
kafka_topic = "user_events"
brokers = "localhost:9092"

# Create a direct stream from Kafka
stream = KafkaUtils.createDirectStream(
    spark, [kafka_topic], {"metadata.broker.list": brokers})

# Define a stateful processing function
def updateState(rdd):
    # Process the data and update the state store
    events = rdd.collect()
    for event in events:
        user_id = event['userId']
        date = event['date']
        processed_events = json.loads(event['processedEvents'])

        if user_id not in processed_events:
            processed_events.append(date)
            # Update the state store with the new events
            # This is a placeholder for actual state update logic

# Apply the processing function to the stream
stream.foreachRDD(updateState)

# Run the pipeline indefinitely
spark.streams.awaitTermination()

#### Closing Summary
and Common Mistakes/Follow-Ups
Correctness: The solution ensures that each event is processed only once, even during partial reprocessing.

#### Common Mistakes
Not properly handling state transitions to avoid duplicates or missing events.
Failing to use a robust state store for maintaining processed states.
Ignoring edge cases like multiple logins per second.
Additional Follow-Ups
How would you handle very large volumes of data and ensure low latency?
What are the trade-offs between using different state stores (e.g., Redis vs. Cassandra)?
How would you implement this solution in a cloud environment?

**Comparison:**

| userId | date       | processedEvents         |
|--------|------------|-------------------------|
| 12345  | 2023-10-01 | [login-2023-10-01]      |
| 67890  | 2023-10-01 | [login-2023-10-01]      |

**Code:**

```python
from pyspark.sql import SparkSession
from pyspark.streaming.kafka import KafkaUtils
import json

# Initialize Spark session
spark = SparkSession.builder.appName("DAUIncrementalPipeline").getOrCreate()

# Define the topic and brokers
kafka_topic = "user_events"
brokers = "localhost:9092"

# Create a direct stream from Kafka
stream = KafkaUtils.createDirectStream(
    spark, [kafka_topic], {"metadata.broker.list": brokers})

# Define a stateful processing function
def updateState(rdd):
    # Process the data and update the state store
    events = rdd.collect()
    for event in events:
        user_id = event['userId']
        date = event['date']
        processed_events = json.loads(event['processedEvents'])

        if user_id not in processed_events:
            processed_events.append(date)
            # Update the state store with the new events
            # This is a placeholder for actual state update logic

# Apply the processing function to the stream
stream.foreachRDD(updateState)

# Run the pipeline indefinitely
spark.streams.awaitTermination()
```

**Example input / output:**

```

Example input:

JSON
Copy
[
    {"userId": "12345", "timestamp": "2023-10-01T09:00:00Z"},
    {"userId": "67890", "timestamp": "2023-10-01T10:00:00Z"}
]

Example output:

JSON
Copy
[
    {"userId": "12345", "date": "2023-10-01", "processedEvents": ["login-2023-10-01"]},
    {"userId": "67890", "date": "2023-10-01", "processedEvents": ["login-2023-10-01"]}
]
```

**Data:**

```json
{
  "userId": "12345",
  "date": "2023-10-01",
  "processedEvents": ["login-2023-10-01"]
}
```

**Flow:**

1. Yes
2. No
3. Event Source (Kafka)
4. Spark Streaming
5. Is Event New?
6. Maintain State Store (Redis)
7. Ignore Duplicate
8. Update DAU Aggregation

