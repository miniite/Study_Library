---
type: "practice"
difficulty: "Deep"
---

# You need a compacted topic to serve as an always-current snapshot of a source table that new lake consumers bootstrap from; how do you tune compaction so bootstrap is fast and complete?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Log Compaction](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Log_Compaction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of creating a compacted Kafka topic that serves as an always-current snapshot of a source table, we need to ensure that the bootstrapping process from new lake consumers is both fast and complete. The key idea here is to leverage the compaction feature in Kafka to retain only the latest message for each unique key in the log. This ensures that when a consumer starts consuming messages, it can quickly get up-to-date with minimal lag.

Kafka compaction works by periodically scanning the log from the beginning (earliest) offset and keeping only the most recent message per key. The frequency of these scans and the retention policies play crucial roles in ensuring both fast and complete bootstrapping. Let's dive into how we can achieve this through a step-by-step first-principles approach.

#### Triage
Understanding the Question

The question is essentially asking for strategies to configure Kafka compaction so that new consumers can quickly bootstrap from an always-up-to-date snapshot of the source table data. This involves understanding and tuning various parameters such as compaction interval, retention policies, and potential edge cases like late-arriving messages.

#### Assumptions
The source table is being streamed into Kafka in batches.
The key space is well-defined and can be used for identifying unique records.
The data stream has a reasonable volume that can be handled by the compaction process.
The target audience for this topic includes both new and existing consumers.

#### First-Principles Solution

Compaction Interval:

Set the compaction interval to a value that balances between frequent enough scans to ensure data freshness and resource efficiency. For example, setting it too frequently (e.g., every hour) could consume more resources unnecessarily.
A common practice is to set it based on the expected rate of change in the source table. If changes are infrequent, a longer interval can be used; if changes are frequent, shorter intervals may be necessary.

Compaction Window:

The compaction window defines how far back in time Kafka will look for duplicate keys when performing compactions.
A larger window ensures more data is compacted but increases the load on the system during compaction runs. Conversely, a smaller window reduces this load but may not capture all necessary updates.

RetentionPolicy:

Define an appropriate retention policy that balances between keeping enough historical data for late-arriving messages and consuming less storage.
Use compact retention to retain only the latest message per key, which is suitable for bootstrapping new consumers.

Late Arrivals Handling:

Ensure that late-arriving messages are handled appropriately by configuring the topic with an appropriate retention policy and compaction settings.

#### Cons
ider using a combination of exactly-once processing guarantees (if supported) to ensure that no data is lost due to late arrivals during bootstrapping.

Monitoring and Tuning:

Regularly monitor the performance of the compaction process to identify any bottlenecks or anomalies.
Use metrics like compaction time, number of duplicates found, and overall cluster utilization to fine-tune the settings as needed.

#### Closing Summary

By carefully configuring Kafka's compaction features, we can ensure that new consumers bootstrap from an always-up-to-date snapshot while keeping resource consumption under control. The key is to balance between the frequency of compactions, the retention window, and the specific requirements of the source table and target consumers.

#### Common Mistakes/Follow-Ups
Ignoring Late Arrivals: Forgetting to account for late-arriving messages can lead to data inconsistencies during bootstrapping.
Overcompacting: Setting a compaction interval that is too frequent can unnecessarily stress the cluster, leading to performance degradation.
Inadequate RetentionPolicy: Not setting an appropriate retention policy might result in either excessive storage usage or missing critical data.

**Comparison:**

| Topic Configurations | Description |
|----------------------|-------------|
| `cleanup.policy`     | Compact      |
| `delete.retention.ms`| 7 days       |
| `min.compaction.lag.ms`| 5 minutes    |
| `max.compaction.lag.ms`| 15 minutes   |
| `segment.bytes`      | Unlimited    |
| `flush.messages.max.bytes`| 2 MB         |
| `retention.ms`       | 7 days       |

**Code:**

```python
# Example configuration snippet for Kafka producer and consumer properties
props = {
    'bootstrap.servers': 'localhost:9092',
    'group.id': 'bootstrapping-group',
    'enable.auto.commit': True,
    'auto.offset.reset': 'earliest',  # Important for bootstrapping
    'key.deserializer': 'org.apache.kafka.common.serialization.StringDeserializer',
    'value.deserializer': 'org.apache.kafka.common.serialization.StringDeserializer'
}

# Producer configuration example (simplified)
producer_config = {
    'bootstrap.servers': 'localhost:9092',
    'acks': 'all',  # Ensure all replicas receive the message
    'linger.ms': 10,  # Delay before sending unflushed data to broker
    'compression.type': 'lz4'  # Use LZ4 compression for efficient data transfer
}
```

**Example input / output:**

```text
Input:
- Producer sends a batch of records to the compacted topic every hour.
- The key is a unique identifier derived from source table data.

Output:
- Consumers can bootstrap quickly and completely by consuming the latest snapshot.
- Compaction ensures that only one message per key is retained, minimizing storage usage and improving performance.
```

**Data:**

```json
{
  "topicConfig": {
    "cleanup.policy": ["compact"],
    "delete.retention.ms": 604800000,  // 7 days retention policy
    "min.compaction.lag.ms": 300000,   // Compaction interval of 5 minutes
    "max.compaction.lag.ms": 900000,   // Maximum compaction lag for late-arrivals
    "segment.bytes": -1,               // Unlimited segment size to prevent compaction window issues
    "flush.messages.max.bytes": 2097152,
    "retention.ms": 604800000          // 7 days retention policy for bootstrapping
  }
}
```

**Flow:**

1. Record Produced
2. Producer Sends Record toTopic
3. Compaction Runs Periodically
4. Only Latest Message Per KeyRetained
5. Consumer Consumes LatestSnapshot
6. Bootstrap Complete & DataFresh

