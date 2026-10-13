---
type: "practice"
difficulty: "Intermediate"
---

# How would you use a message key to make sure all events for the same order end up in order in your warehouse staging table?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Partitions and Ordering](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Partitions_and_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that all events for the same order end up in order in the warehouse staging table, we can leverage Apache Kafka's message keys effectively. At a high level, we will assign each event a unique key based on the order ID and use this key to maintain the order of delivery to our processing pipeline.

#### Triaging the Question

The question is asking how to ensure that all events related to the same order are processed in the correct sequence by leveraging Kafka's message keys. The interviewer might be probing your understanding of Kafka's internal mechanisms, particularly its partitioning and ordering features. We need to assume that we have a stream of order-related events and that these events can be grouped by an order_id field.

#### First-Principles Solution
Event Structure: Each event in the stream should contain fields such as order_id, event_type, timestamp, etc.
Message Key Assignment: Assign each event a key based on its order_id. This ensures that all events for the same order will be routed to the same partition.
Kafka Topic Configuration: Configure the Kafka topic with an appropriate number of partitions to handle the throughput and ensure even distribution of events across partitions.
Step-by-Step Solution

Event Data Structure:

JSON
{
    "order_id": "12345",
    "event_type": "ORDER_PLACED",
    "timestamp": "2023-10-01T10:00:00Z"
}

Message Key Assignment:

For each event, the order_id field will be used as the message key.
PYTHON
def assign_key(event):
    return event['order_id']

Kafka Topic Setup:

Create a Kafka topic with an appropriate number of partitions. The number of partitions should match or exceed the expected concurrency level in your processing pipeline.
BASH
kafka-topics.sh --create --topic order-events --partitions 10 --replication-factor 3 --if-not-exists --zookeeper localhost:2181

Kafka Consumer Configuration:

Configure the Kafka consumer to consume events from the topic, ensuring that all messages with the same key are processed in order.
PYTHON

#### cons
umer = KafkaConsumer(
    'order-events',
    bootstrap_servers=['localhost:9092'],
    group_id='order-processing-group',
    auto_offset_reset='earliest'
)
for message in consumer:
    if assign_key(message.value) == "12345":
        # Process the event
        process_event(message.value)

#### Closing Summary
and Common Mistakes/Follow-Ups

By using order_id as the key, we ensure that all events related to a specific order are routed to the same partition and processed in the correct order. This approach leverages Kafka's inherent properties of ordering within partitions.

#### Common mistakes
include:

Not correctly assigning keys, leading to messages not being grouped by order.
Failing to configure the number of partitions appropriately, which can lead to load imbalances or decreased performance.

#### Follow-ups
might explore scaling considerations (e.g., what happens if more orders are processed) and how to handle late events gracefully.

**Code:**

```python
def assign_key(event):
    return event['order_id']
```

**Example input / output:**

```

No sample input-output is provided for this problem, but a typical scenario might be:

PYTHON
Copy
message = {'order_id': '12345', 'event_type': 'ORDER_PLACED', 'timestamp': '2023-10-01T10:00:00Z'}
key = assign_key(message)
print(f"Key for event {message}: {key}")

Output:

PYTHON
Copy
Key for event {'order_id': '12345', 'event_type': 'ORDER_PLACED', 'timestamp': '2023-10-01T10:00:00Z'}: 12345
```

**Data:**

```json
{
    "order_id": "12345",
    "event_type": "ORDER_PLACED",
    "timestamp": "2023-10-01T10:00:00Z"
}
```

**Flow:**

1. Event Producer
2. Assign Order ID as Key
3. Kafka Broker
4. Order Events Topic (10Partitions)
5. Consumer Group Consumes
6. Process Order Events in Order

