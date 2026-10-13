---
type: "practice"
difficulty: "Deep"
---

# Why is one consumer thread per partition a common concurrency model, and what breaks if you share a consumer across threads?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Consumer Groups and Rebalancing](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Consumer_Groups_and_Rebalancing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a Kafka consumer, using one consumer thread per partition is a common concurrency model due to several practical benefits. This approach ensures that each partition's messages are processed independently by a single thread, which simplifies error handling and ensures that consumers do not inadvertently lose messages or cause data skew.

To understand why this pattern works well, we need to break down the core concepts of Kafka partitions and consumer groups. Each topic in Kafka is divided into multiple partitions for load balancing and fault tolerance. A partition is a sequence of log entries; all messages within a single partition are ordered by their offset. A consumer group can have multiple consumers, but each partition must be assigned exactly one consumer from the group.

Let's dive into why this model works so well:

Simplicity in Error Handling: By having a single thread for each partition, error handling becomes more straightforward. If a single thread fails to process a message, it does not affect other threads processing different partitions. This isolation allows for easier debugging and recovery.

Balanced Load Distribution: Each consumer thread handles its own partition, ensuring that the load is evenly distributed across all available resources. This helps in avoiding situations where one part of the system becomes a bottleneck due to an overloaded consumer.

Guaranteed Ordering: Since each thread processes messages from only one partition, the order of messages within a partition is preserved. This is crucial for many applications that rely on message ordering.

Scalability and Fault Tolerance: If you have more consumers than partitions, some will be idle (which can be configured via max.poll.records), allowing your system to scale better with additional resources. Moreover, if a consumer fails, the group rebalance mechanism ensures that another thread takes over the partition.

Now, let's consider what breaks when you share a consumer across multiple threads:

Message Ordering: If a single thread handles messages from multiple partitions, message ordering is no longer guaranteed within the application. This can lead to incorrect state and behavior in applications that depend on the order of events.

Load Imbalance: Sharing a consumer across threads means that some threads might process many more messages than others, leading to load imbalances. Inefficient resource utilization can occur if one thread is overwhelmed while others are idle.

Error Handling Complexity: If an error occurs in one part of the application, it could affect multiple partitions handled by the same thread. This makes debugging and recovery more complex since the failure might not be isolated to a single partition.

Concurrency Issues: Synchronizing access to shared resources within a multi-threaded consumer can introduce complexity and potential deadlocks or race conditions. Ensuring thread safety for critical operations becomes a significant challenge.

In summary, using one consumer thread per partition is a robust design choice that simplifies the application of error handling, ensures balanced load distribution, maintains message ordering, and provides better fault tolerance and scalability. Conversely, sharing consumers across threads can lead to issues with message ordering, imbalance, increased complexity in error handling, and potential concurrency problems.

**Comparison:**

```
Partition	Messages
0	Message1, Message2, Message3
1	Message4, Message5
2	Message6
```

**Code:**

```python
# Example of a single-threaded Kafka consumer for each partition
from kafka import KafkaConsumer

def setup_consumer(topic):
    consumer = KafkaConsumer(bootstrap_servers='localhost:9092',
                             auto_offset_reset='earliest',
                             enable_auto_commit=True)
    consumer.subscribe([topic])

    # Process messages one by one from a single partition
    for message in consumer:
        print(f"Received message {message.value} on partition {message.partition}")
```

**Example input / output:**

```

Given a topic order_events with 3 partitions and the following input:

Partition 0: Message1, Message2, Message3
Partition 1: Message4, Message5
Partition 2: Message6

The output would be:

PLAINTEXT
Copy
Received message Message1 on partition 0
Received message Message4 on partition 1
Received message Message6 on partition 2
```

**Data:**

```json
{
  "topics": ["order_events", "transaction_logs"],
  "partitions_per_topic": {
    "order_events": 3,
    "transaction_logs": 2
  }
}
```

**Flow:**

1. Partition 0
2. Message1
3. Handle Message1
4. Message2
5. Handle Message2
6. Message3
7. Handle Message3
8. Partition 1
9. Message4
10. Handle Message4
11. Message5
12. Handle Message5
13. Partition 2
14. Message6
15. Handle Message6

