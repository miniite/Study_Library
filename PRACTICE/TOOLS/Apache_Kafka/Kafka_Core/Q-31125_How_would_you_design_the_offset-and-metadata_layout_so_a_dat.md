---
type: "practice"
difficulty: "Deep"
---

# How would you design the offset-and-metadata layout so a data engineer can reprocess exactly the events between two wall-clock timestamps after a bad transform deploy?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Offset Commits](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Offset_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to design an efficient and robust offset-and-metadata layout that allows a data engineer to reprocess exactly the events within a specific time window after deploying a faulty transformation logic. The key idea is to leverage Kafka’s offset mechanism in combination with metadata that captures the relevant timestamps for each event. Here's how we can achieve this:

Understand the Question and Probing: The interviewer is probing your understanding of Kafka’s offsets, partitioning strategy, timestamp handling, and how to implement a rollback or reprocessing scenario. We need to ensure that we have a clear understanding of the timestamps (both log and wall-clock) and how they relate to event processing.

Key Assumptions:

We are dealing with a fault-tolerant system where we can reliably identify events based on timestamps.
The data pipeline processes messages in batches or streams them continuously.
There is a mechanism to capture the exact timestamps of events as they are produced.
Offsets are unique identifiers for each message within a partition.

#### First-Principles Solution

Step 1: Capture Timestamps: Ensure that each event in Kafka has an associated timestamp. This can be either the log time (when the event was written to Kafka) or wall-clock time (when the event occurred).
Step 2: Define Time Windows: For reprocessing, we need to define a window of interest based on the two given timestamps.
Step 3: Fetch Relevant Offsets: Use the timestamp range to fetch the relevant offsets from Kafka. We can use Kafka's ListOffset API with a time-based seek method.
Step 4: Replaying Events: Replay only the events within this offset range using a custom processor or by adjusting the consumer group’s starting point.

#### Implementation
Details

Metadata Layout:

Store offsets alongside their associated timestamps in a metadata topic.
Each entry in the metadata topic could look like:
JSON
{
  "offset": 12345,
  "timestamp_ms": 1672539000000, // Unix timestamp in milliseconds
  "event_id": "abc123"
}

Reprocessing Workflow:

Given the start and end timestamps (let's call them start_time and end_time), query the metadata topic to find all offsets within this range.
Use Kafka’s consumer API to seek to these offsets and replay only those messages.

#### Example
Code Snippet
PYTHON
from kafka import KafkaConsumer, TopicPartition

def get_metadata_for_timestamps(start_time, end_time):
    # Fetch relevant metadata entries between start_time and end_time
    # This is a simplified example; in practice, you'd query your metadata store
    return [
        {"offset": 1000, "timestamp_ms": 1672539000000},
        {"offset": 1200, "timestamp_ms": 1672540000000}
    ]

def reprocess_events(start_time, end_time):
    # Get the relevant offsets
    metadata_entries = get_metadata_for_timestamps(start_time, end_time)

#### cons
umer = KafkaConsumer(bootstrap_servers='localhost:9092', auto_offset_reset='earliest')

    for entry in metadata_entries:
        tp = TopicPartition('event_topic', 0)

#### cons
umer.seek(tp, entry['offset'])

        # Consume and process the message
        for msg in consumer:
            if entry['timestamp_ms'] >= start_time and entry['timestamp_ms'] <= end_time:
                print(f"Processing event at offset {msg.offset} with timestamp {entry['timestamp_ms']}")

reprocess_events(start_time=1672539000000, end_time=1672540000000)

#### Common Mistakes
and Follow-Ups
Mistake 1: Failing to properly store or retrieve the timestamps associated with each message. Ensure that timestamps are consistently captured and stored.
Mistake 2: Not handling edge cases such as messages outside the defined time window or errors during reprocessing.

#### Follow-Up
Question: How would you handle scenarios where there is a large gap in data between start_time and end_time?

**Comparison:**

| Metadata Entry | Offset | Timestamp (ms) | Event ID |
|---------------|--------|----------------|----------|
| 1             | 1000   | 1672539000000  | abc123   |
| 2             | 1200   | 1672540000000  | def456   |

**Code:**

```python
from kafka import KafkaConsumer, TopicPartition

def get_metadata_for_timestamps(start_time, end_time):
    # Fetch relevant metadata entries between start_time and end_time
    return [
        {"offset": 1000, "timestamp_ms": 1672539000000},
        {"offset": 1200, "timestamp_ms": 1672540000000}
    ]

def reprocess_events(start_time, end_time):
    # Get the relevant offsets
    metadata_entries = get_metadata_for_timestamps(start_time, end_time)

    consumer = KafkaConsumer(bootstrap_servers='localhost:9092', auto_offset_reset='earliest')

    for entry in metadata_entries:
        tp = TopicPartition('event_topic', 0)
        consumer.seek(tp, entry['offset'])

        # Consume and process the message
        for msg in consumer:
            if entry['timestamp_ms'] >= start_time and entry['timestamp_ms'] <= end_time:
                print(f"Processing event at offset {msg.offset} with timestamp {entry['timestamp_ms']}")
```

**Example input / output:**

```text
Input:
Start Time: 1672539000000, End Time: 1672540000000

Output:
Processing event at offset 1000 with timestamp 1672539000000
Processing event at offset 1200 with timestamp 1672540000000
```

**Data:**

```json
[
    {
      "offset": 1000,
      "timestamp_ms": 1672539000000,
      "event_id": "abc123"
    },
    {
      "offset": 1200,
      "timestamp_ms": 1672540000000,
      "event_id": "def456"
    }
]
```

**Flow:**

1. Yes
2. Yes
3. No
4. Start Timestamp
5. Is within window?
6. Seek to Offset
7. Process Message
8. End Timestamp
9. Log Outside Window

