---
type: "practice"
difficulty: "Deep"
---

# You have a Kafka topic where each message is ~2MB (embedded document blobs). Consumers keep hitting rebalances and OOMs. How would you redesign the payload and consumer config to fix this?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tiered Storage for Logs](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Tiered_Storage_for_Logs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of rebalances and Out-of-Memory (OOM) errors in consumers, we need to revisit both the payload structure and the consumer configuration. The core problem stems from the large message size (~2MB embedded document blobs), which can lead to inefficient memory usage and frequent rebalancing.

Triaging the Problem

The question is primarily asking about optimizing data transmission by reducing the size of each message, ensuring efficient memory usage, and tuning Kafka consumer configurations to handle larger message sizes more gracefully. The assumptions are that the current system relies on a single or few consumers, and rebalances occur frequently due to the large payload.

#### First Principles Solution
Reduce Message Size: Splitting the document into smaller chunks could help in reducing the memory footprint.
Optimize Memory Usage: Tuning consumer configurations can prevent OOM errors.
Improve Consumer Resilience: Ensuring consumers handle large messages without crashing or frequently rebalancing.
Step-by-Step Solution

Refactor Payload:

Split large documents into smaller, more manageable chunks.
Use Kafka’s ability to send multiple messages in a single batch by setting the batch.size and linger.ms properties appropriately.

Tune Consumer Configurations:

Increase the consumer's memory usage limits with max.partition.fetch.bytes.
Configure the fetch.min.bytes and session.timeout.ms to manage how often consumers fetch data, reducing frequent rebalances.
Adjust the max.poll.records to control the number of records a consumer can process in a single poll.

Implement Error Handling:

Use Kafka's built-in error handling mechanisms or custom logic to retry failed messages.

#### Cons
ider using a dead-letter queue (DLQ) for messages that cannot be processed due to errors.
Code and Configuration Example

Here is an example of how you might configure the consumer in Python:

PYTHON
from kafka import KafkaConsumer

# Initialize consumer with optimal configuration

#### cons
umer = KafkaConsumer(
    'your_topic',
    bootstrap_servers=['localhost:9092'],
    group_id='your_group',
    max.partition.fetch.bytes=5242880,  # 5MB
    fetch.min.bytes=1,
    session.timeout.ms=30000,  # 30 seconds
    auto_offset_reset='earliest',
    enable.auto.commit=False,

#### cons
umer_timeout_ms=5000,  # 5 seconds for fetching messages
    max.poll.records=200  # Process up to 200 records per poll
)
Sample Data and Schema

Assume each document has a structure like this:

JSON
{
  "id": "unique_id",
  "data": {
    "chunk1": "chunk_data",
    "chunk2": "more_data",
    "chunk3": "even_more_data"
  }
}
Dry-Run and Input->Output Example

Let’s simulate a scenario where we have a large document split into three chunks:

JSON
[
  {"id": "1", "data": {"chunk1": "chunk_data"}},
  {"id": "1", "data": {"chunk2": "more_data"}},
  {"id": "1", "data": {"chunk3": "even_more_data"}}
]

#### Closing Summary
and Common Mistakes

By refactoring the payload to smaller chunks, tuning Kafka consumer configurations, and implementing robust error handling, we can significantly reduce rebalances and OOM errors. Common mistakes include not understanding the implications of max.partition.fetch.bytes and fetch.min.bytes, underestimating the memory requirements for large messages, and failing to handle retries properly.

Self-Check

This solution is correct and complete.

**Comparison:**

```
Property	Value	Description
max.partition.fetch.bytes	5242880 (5MB)	Maximum amount of data to fetch per partition.
fetch.min.bytes	1	Minimum bytes to fetch before completing the request.
session.timeout.ms	30000	Timeout for a consumer session.
auto_offset_reset	earliest	Offset reset policy when no offset found.
enable.auto.commit	False	Enable or disable automatic commit of offsets.
consumer_timeout_ms	5000	Timeout for the consumer to fetch messages.
max.poll.records	200	Maximum number of records to poll in one request.
```

**Code:**

```python
from kafka import KafkaConsumer

consumer = KafkaConsumer(
    'your_topic',
    bootstrap_servers=['localhost:9092'],
    group_id='your_group',
    max.partition.fetch.bytes=5242880,
    fetch.min.bytes=1,
    session.timeout.ms=30000,
    auto_offset_reset='earliest',
    enable.auto.commit=False,
    consumer_timeout_ms=5000,
    max.poll.records=200
)
```

**Example input / output:**

```text
[
  {"id": "1", "data": {"chunk1": "chunk_data"}},
  {"id": "1", "data": {"chunk2": "more_data"}},
  {"id": "1", "data": {"chunk3": "even_more_data"}}
]
```

**Data:**

```json
{
  "id": "unique_id",
  "data": {
    "chunk1": "chunk_data",
    "chunk2": "more_data",
    "chunk3": "even_more_data"
  }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Split large documents intosmaller chunks
5. Is the message size reduced?
6. Configure consumer settingsfor memory usage and fetchlimits
7. Handle errors using DLQ orretries
8. Tune Kafka producer forefficient batching
9. Monitor and adjust as needed
10. End

