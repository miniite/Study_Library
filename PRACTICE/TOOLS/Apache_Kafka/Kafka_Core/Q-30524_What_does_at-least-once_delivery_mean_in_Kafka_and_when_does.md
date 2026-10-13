---
type: "practice"
difficulty: "Intermediate"
---

# What does 'at-least-once' delivery mean in Kafka and when does it occur?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Exactly Once Beyond Kafka](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Exactly_Once_Beyond_Kafka/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

At-least-once delivery in Apache Kafka means that a message will be delivered at least once to its intended consumer, even if there are transient errors or failures during processing. This guarantee is crucial for ensuring data integrity and reliability in distributed systems. Let's break down the concept, understand when it occurs, and explore some potential pitfalls.

#### Triage
Understanding the Question

The question probes your understanding of Kafka’s delivery semantics and how they affect message processing guarantees. Specifically, you need to explain what 'at-least-once' means, provide scenarios where this guarantee is necessary, and discuss any trade-offs involved.

#### Assumptions
and Context
Kafka Clusters: We are discussing a standard Kafka cluster with producers, brokers, and consumers.
Transient Errors: These refer to temporary failures like network glitches or broker restarts.

#### Cons
umer Processing: Consumers fetch messages from the Kafka topic and process them based on their business logic.

#### First Principles Solution

Message Flow in Kafka:

Producers send messages to a Kafka cluster, which are stored in topics partitioned across multiple brokers.

#### Cons
umers subscribe to these topics and pull or consume messages as needed.

At-Least-Once Delivery Mechanism:

When a message is produced, it is first written into the log by the broker.
For each consumer group, Kafka maintains an offset that tracks which messages have been consumed.
If a transient error occurs between when a consumer fetches a message and processes it (e.g., network failure, broker crash), the message remains in the offset log but is not acknowledged as fully processed.

Guarantee of Delivery:

Once a producer sends a message to a Kafka topic, it will be stored persistently on at least one broker.
Even if subsequent errors occur during processing, the message can be retried by consumers until it is successfully consumed or marked as dead-lettered.

Scenarios Where At-Least-Once Occurs:

Network partitioning: If a consumer fetches a message before a network failure but fails to process it due to an error.
Broker restarts: A broker may crash after storing the message and before acknowledging receipt, causing the message to be redelivered.

#### Closing Summary

At-least-once delivery ensures that messages are not lost due to temporary errors. However, this guarantee comes with potential side effects such as message reprocessing and dead-lettering scenarios. Understanding these dynamics is crucial for designing robust Kafka-based systems.

#### Common Mistakes/Follow-Ups
Misunderstanding the Trade-offs: Failing to recognize that while at-least-once delivery ensures messages are not lost, it can lead to duplicate processing if a message is consumed multiple times.
Not Considering Dead-Lettering Strategies: Not having a plan for handling and managing dead-lettered messages effectively can clutter your system with redundant data.

#### Example Scenario

Imagine an e-commerce application where order confirmation emails need to be sent after a purchase. If there’s a network failure between the Kafka broker storing the message and the consumer processing it, the email might not be sent immediately but will eventually be redelivered, ensuring the transaction is confirmed.

Message
Stays in Log

#### Cons
umer Fetches
Processes
Before Processing
After Storing, BeforeAcknowledgment
Producer
Kafka Broker
Offset

#### Cons
umer
Order Confirmation Email
Network Failure
Broker Restart

**Comparison:**

| Scenario           | Outcome         |
|--------------------|----------------|
| Successful Process | Message is consumed once.  |
| Network Partition  | Message reprocessed if undelivered. |
| Broker Restart     | Message redelivered on restart.    |

**Code:**

```python
# Example consumer code snippet in Python (simplified)
from kafka import KafkaConsumer

consumer = KafkaConsumer('my-topic', bootstrap_servers=['localhost:9092'])
for message in consumer:
    try:
        process_message(message.value)
    except Exception as e:
        print(f"Error processing message: {e}")
```

**Example input / output:**

```text
Input:
Producer sends message to 'my-topic'

Output:
Message is stored persistently on at least one broker.
If a network failure occurs after storage but before processing,
the consumer will eventually redeliver the message.

Dry-run scenario:
1. Producer -> Broker (Message stored)
2. Consumer fetches and processes
3. Network Failure -> Redelivery of Message
4. Consumer retries, ensuring message is processed.
```

**Data:**

```json
{
  "topic": "my-topic",
  "partitions": [
    {"id": 0, "replicas": [103, 204]},
    {"id": 1, "replicas": [105, 107]}
  ]
}
```

**Flow:**

1. Before Processing
2. After Storing, BeforeAcknowledgment
3. Producer
4. Kafka Brokers
5. Storing Messages
6. Consumer Fetches
7. Processing Message
8. Transient Error
9. Broker Restart

