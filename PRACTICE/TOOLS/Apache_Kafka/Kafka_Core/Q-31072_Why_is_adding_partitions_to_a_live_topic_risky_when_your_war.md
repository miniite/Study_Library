---
type: "practice"
difficulty: "Deep"
---

# Why is adding partitions to a live topic risky when your warehouse loads depend on per-key ordering, and what would you do instead?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Partitions and Ordering](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Partitions_and_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why adding partitions to a live Kafka topic can be risky when your warehouse relies on per-key ordering, let's first break down the key concepts involved.

TL;DR Summary

Adding partitions to a live Kafka topic disrupts the order of records with the same key across partitions. This impacts applications that depend on per-key ordering for their data processing logic, such as real-time warehousing systems. Instead, you can implement strategies like log compaction or use a different partitioning strategy.

#### Triaging the Question

The question is probing your understanding of Kafka's partition mechanics and the implications of changing partitions in a live system. It also tests your ability to think through alternative solutions that maintain ordering without disrupting operations.

#### First Principles Solution
Understanding Per-Key Ordering

Per-key ordering means that all records with the same key are guaranteed to be processed in the order they were produced, even when distributed across multiple partitions. This is crucial for maintaining consistency and order in systems like real-time warehouses where transactions need to be processed as a single stream of events.

Risks of Adding Partitions
Ordering Disruption: When you add more partitions to an existing topic, Kafka will rebalance the partition leaders among brokers. This process can lead to records with the same key being distributed across different partitions. As a result, per-key ordering is broken.
Data Distribution Changes: Existing data may be redistributed, which could impact how your consumer applications or real-time processing pipelines handle the data.
Latency and Throughput Impact: Rebalancing can cause temporary increases in latency as producers wait for new leaders to be elected. This might also affect throughput if rebalancing happens frequently.
Alternative Solutions
1. Log Compaction

One approach is to use log compaction, which retains the lastest message for each key while retaining all previous messages. This ensures that even if you add partitions, messages with the same key will still be ordered correctly.

#### Implementation
Kafka brokers automatically compact logs based on a configurable retention policy.

#### Example
Configuration:
PROPERTIES
log.retention.hours=168 # Retain data for 7 days
delete.retention.ms=86400000 # Compact every day at midnight
2. Using a Single Partition

Another approach is to ensure that all records with the same key are always written to the same partition, thus maintaining per-key ordering.

#### Implementation
You can use a custom partitioner in Kafka to route messages based on their keys.

#### Example
Code:
JAVA
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.common.serialization.StringSerializer;

public class CustomPartitioner implements Partitioner {
    @Override
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        // Use the hash of the key to determine the partition number.
        return Math.abs(key.hashCode()) % cluster.partitionsFor(topic).size();
    }

    @Override
    public void close() {}

    @Override
    public void configure(Map<String, ?> configs) {}
}
3. Data Modeling Changes

You can also change your data modeling approach to ensure that per-key ordering is maintained without needing additional partitions.

Strategy: Design your topics and partitions such that the key space is balanced and evenly distributed.

#### Closing Summary

Adding partitions to a live Kafka topic disrupts per-key ordering, which can impact real-time warehouse systems. To mitigate this risk, consider using log compaction or ensuring consistent partitioning strategies. These approaches maintain data integrity and order while avoiding the risks of rebalancing.

#### Common Mistakes
and Follow-Ups
Ignoring Rebalancing Risks: Many engineers focus solely on adding partitions without considering the impact of rebalancing.
Overlooking Log Compaction: Not understanding how log compaction works can lead to suboptimal solutions.
Inadequate Data Modeling: Poor initial design can make it difficult to maintain order even with additional measures.

**Code:**

```java
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.common.serialization.StringSerializer;

public class CustomPartitioner implements Partitioner {
    @Override
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        // Use the hash of the key to determine the partition number.
        return Math.abs(key.hashCode()) % cluster.partitionsFor(topic).size();
    }

    @Override
    public void close() {}

    @Override
    public void configure(Map<String, ?> configs) {}
}
```

**Data:**

```json
{
  "log.retention.hours": 168,
  "delete.retention.ms": 86400000
}
```

**Flow:**

1. Yes
2. No
3. Produce Event
4. Is Key Present?
5. Send to Correct Partition
6. Hash Key and Send to Partition
7. Log Compaction Ensures Order
8. Rebalance for New Partitions

