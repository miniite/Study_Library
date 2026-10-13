---
type: "practice"
difficulty: "Basic"
---

# What is a Kafka message (record) made of? Describe key, value, timestamp, and headers.

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ When Not to Use Kafka](<../../../../TOOLS/Apache_Kafka/Kafka_Core/When_Not_to_Use_Kafka/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Apache Kafka, a message (often referred to as a record or a log entry) consists of four main components: the key, the value, the timestamp, and the headers. Each part plays a crucial role in defining the structure and metadata associated with each message within a topic.

#### Triage

The question is asking for an understanding of the basic composition of a Kafka message. The interviewer might be probing your knowledge of Kafka's fundamental concepts and how messages are structured to support efficient data streaming.

Key, Value, Timestamp, Headers

Key: In Kafka, every record in a partition must have a key. While this is optional for records written to the end of a topic (for example, when using ProducerRecord without specifying a key), it's highly recommended as it allows Kafka to distribute messages across partitions based on the key value.

Value: This is the main content or payload of the message. It can be any binary data and is used by consumers for processing.

Timestamp: The timestamp field in Kafka records represents when a record was produced. Kafka internally uses this timestamp for its own purposes, such as managing the retention policy. When producing messages with ProducerRecord, you can set an explicit timestamp or use the current time.

Headers: Headers are optional metadata fields that you can attach to a message. They allow you to add custom information about the record that is not part of the key or value. Kafka supports up to 65,535 headers per message.

#### First-Principles Solution

Let’s break down each component in more detail:

Key

The key is used for partitioning messages within a topic. When a record is sent to a Kafka topic, it is hashed using the key's hash function and then mapped to one of the partitions based on this hash value. This ensures that records with similar keys are likely to be stored together in the same partition, which can improve performance during data processing.

Value

The value represents the actual message content. It could be anything from a simple string to complex JSON or binary objects. The size and format of the value are not constrained by Kafka itself; these constraints are typically imposed by application logic.

Timestamp

When producing messages, you can set an explicit timestamp that will be stored in the metadata. If no explicit timestamp is provided, Kafka assigns it based on the current time when the message is produced. This timestamp can be used for ordering purposes and is especially useful when dealing with real-time data where freshness is critical.

Headers

Headers provide a way to attach additional metadata to messages without changing their key or value. They are often used for logging, tracing, or any custom information that you might need during processing. Each header consists of a key (string) and a value (array of bytes). You can add as many headers as needed.

#### Closing Summary
+ Common Mistakes/Follow-Ups

To summarize, a Kafka record is composed of four main parts: the key, the value, the timestamp, and the headers. Understanding these components is crucial for effective usage of Kafka in production environments.

#### Common Mistakes

Forgetting to include keys when producing messages, which can lead to random distribution across partitions.
Not setting appropriate timestamps, leading to potential issues with ordering or retention policies.
Overlooking the use of headers for essential metadata that could simplify data processing and debugging.

During follow-ups, the interviewer might ask about scenarios where these components are critical (e.g., implementing ordered messaging) or how they interact in complex applications involving multiple Kafka brokers and consumers. Understanding these nuances will demonstrate your depth of knowledge and practical experience with Kafka.

**Code:**

```python
# Example of producing a record with all fields
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers='localhost:9092')
record = {
    'key': b'1',
    'value': b'some message content',
    'timestamp': 1634576897,
    'headers': [(b'header-key', b'header-value')]
}

producer.send('test-topic', key=record['key'], value=record['value'], timestamp_ms=record['timestamp'],
              headers=[(k.encode(), v) for k, v in record['headers']])
```

**Example input / output:**

```text
Input:
- Key: '1'
- Value: 'some message content'
- Timestamp: 1634576897
- Headers: [('header-key', 'header-value')]

Output (produced record):
- Produced to topic: test-topic
```

**Data:**

```json
{
    "key": "1",
    "value": "some message content",
    "timestamp": 1634576897,
    "headers": [
        {"key": "header-key", "value": "header-value"}
    ]
}
```

