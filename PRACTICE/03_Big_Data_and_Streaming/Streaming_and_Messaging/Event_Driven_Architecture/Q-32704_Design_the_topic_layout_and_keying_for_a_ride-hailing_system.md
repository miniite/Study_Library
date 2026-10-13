---
type: "practice"
difficulty: "Deep"
---

# Design the topic layout and keying for a ride-hailing system where per-driver location updates must stay ordered but you have 500k active drivers. Justify partition count and key choice.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Idempotent Consumers](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Fat_vs_Thin_Events/Idempotent_Consumers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an efficient message queue system for handling per-driver location updates in a ride-hailing application, we need to ensure that messages are processed in order by each driver while minimizing the number of partitions and optimizing performance. Given 500k active drivers, our key choices will impact scalability and ordering guarantees.

The core challenge is to design a topic layout with keys that maintain message ordering for each driver. We'll start from first principles: understanding the constraints, choosing an appropriate partitioning strategy, and justifying the number of partitions based on the required throughput and latency.

Firstly, we need to acknowledge that in Kafka (a popular choice for such systems), messages within a partition are guaranteed to be processed in order. Therefore, our primary goal is to distribute drivers across partitions such that each driver’s location updates stay ordered as they are consumed by their respective consumers.

#### Triaging the Question
What is being asked? The question asks us to design the topic layout and keying strategy for a Kafka system where per-driver location updates must be ordered.
What's being probed? The interviewer is probing our understanding of Kafka partitions, keying strategies, and how to achieve ordered processing in an event-driven architecture.

#### Assumptions
We are using Apache Kafka as the message queue system.
Each driver sends a steady stream of location updates at a rate that needs to be processed in order by their respective consumers (e.g., trip matching service).
The goal is to minimize the number of partitions while ensuring ordered processing for each driver.

#### Solution

To achieve this, we will use a custom keying strategy that groups messages from the same driver into the same partition. This ensures that all location updates from a given driver are processed in order within their respective partition.

Partition Count:

Given 500k active drivers, we can start by calculating the number of partitions needed.
A common practice is to have at least one partition per driver to ensure each driver’s messages stay ordered. However, having more partitions can improve parallelism and throughput but might increase complexity.

Key Choice:

The key should uniquely identify each driver. A simple approach could be using the driver ID as the key.

#### Example
key = <driver_id>

Partitioning Strategy:

Use a custom partitioner that maps keys (driver IDs) to partitions in a way that minimizes load imbalance.
For simplicity, we can use the hash of the driver ID to determine the partition number.

#### Implementation
Example:

Suppose driver_id is a string or integer. The keying function could be as follows:
JAVA
public int getPartitionKey(String driverId) {
    return Math.abs(driverId.hashCode()) % numberOfPartitions;
}

Ordering Guarantee:

By using the same key (driver ID), all messages from a given driver will go to the same partition, ensuring ordered processing.
Each consumer will process its assigned partition’s messages in order.

Partition Count Justification:

For 500k drivers, we can use the default Kafka partition count of 1 per driver (i.e., 500k partitions). However, this might lead to high overhead.
A more practical approach is to balance between ordering and performance by using a smaller number of partitions. For instance, if each partition handles messages for approximately 100 drivers, we would have 5000 partitions.

#### Closing Summary

To summarize, the key design choices involve:

Using the driver ID as the message key.
Partitioning based on the hash of the driver ID to balance load.
Maintaining a partition count that balances between ordering requirements and performance constraints (e.g., 5000 partitions for 500k drivers).

This approach ensures ordered processing while keeping the number of partitions manageable.

#### Common Mistakes/Follow-Ups
Mistake: Using random keys or no keying strategy, leading to mixed-up messages from different drivers.

#### Follow-Up
Discuss potential failure modes such as high partition counts causing performance issues and how to monitor and manage load balance across partitions.
Further Exploration: Consider using additional strategies like co-location with stateful services if necessary.

**Code:**

```java
public class DriverPartitioner implements Partitioner {
    private final int numberOfPartitions;

    public DriverPartitioner(int numberOfPartitions) {
        this.numberOfPartitions = numberOfPartitions;
    }

    @Override
    public int partition(String topic, Object key, byte[] bytes, Integer partitions, Object object) {
        if (key instanceof String) {
            return Math.abs(((String) key).hashCode()) % numberOfPartitions;
        }
        throw new IllegalArgumentException("Key must be a string");
    }

    // Additional methods for Kafka Partitioner interface
}
```

**Flow:**

1. Message Produced with DriverID
2. Partitioner (Hash of Driver ID)
3. Assigned Partition
4. Consumer ConsumesMessages in Order fromAssigned Partition

