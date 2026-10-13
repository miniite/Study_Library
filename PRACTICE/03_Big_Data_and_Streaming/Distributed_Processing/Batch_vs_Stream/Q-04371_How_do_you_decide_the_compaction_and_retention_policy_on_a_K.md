---
type: "practice"
difficulty: "Deep"
---

# How do you decide the compaction and retention policy on a Kafka topic that feeds both a streaming consumer and periodic batch reloads?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Watermarks and State Management](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Watermarks_and_State_Management.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding the compaction and retention policies for a Kafka topic that serves dual purposes—streaming consumers and periodic batch reloads—the key is to balance the trade-offs between data integrity, processing efficiency, and storage costs. In this scenario, we must ensure that historical data can be reliably accessed by batch processes while not overwhelming our storage with redundant or stale information.

#### Triaging the Question

The question probes your understanding of Kafka’s compaction and retention policies in a hybrid use case. It requires you to consider how these settings affect both real-time streaming consumers and offline batch processing, ensuring that the data is both fresh for stream processing and stored appropriately for batch operations. Key assumptions include:

The Kafka topic is used by multiple types of consumers.
Batch reloads are periodic and rely on historical data.

#### First Principles Solution

Compaction Policy:

Type: Choose between Log Compaction or Time-based compaction.
Log Compaction: This policy retains the lastest value for each unique key, which is useful if you need to maintain the latest state of entities. However, it’s not suitable for all batch processing use cases because old values might be needed.
Time-based compaction: Retains messages based on a defined time window before discarding them. This can help with periodic batch reloads that don’t require real-time updates.

Retention Policy:

Topic Retention Time (in seconds or days): Define the period for which data should be retained, balancing between storage costs and necessary historical data.
For batch processing, you might want to keep data longer than what’s strictly needed by stream consumers to support backfills or deep analysis.

Partitioning:

Ensure proper partitioning strategy that aligns with both streaming and batch use cases, possibly using hash-based partitioning for efficient distribution across nodes.
Detailed Solution

Compaction Policy:

For a Kafka topic used by both streaming consumers and periodic batch reloads, time-based compaction is often the best choice.
PLAINTEXT
"compaction.window.ms": 604800000, // ~7 days
This ensures that old messages are cleaned up after a certain period, reducing storage overhead and ensuring that batch processes can rely on recent data.

Retention Policy:

Set the topic retention to a value that meets both needs:
PLAINTEXT
"retention.ms": 604800000 // ~7 days
This ensures that while stream processing gets fresh and relevant data, batch processes can access reliable historical data.

Partitioning Strategy:

Use a partitioner that aligns with the use case. For example, if you are processing events from multiple sources (like different geographies or types of entities), hash-based partitioning might be appropriate.
PLAINTEXT
"partitioner.class": "org.apache.kafka.common.internals.DefaultPartitioner"

Monitoring and Adjustment:

Continuously monitor the topic’s size and performance impact on both streaming and batch consumers. Adjust retention policies based on observed needs and cost considerations.

#### Closing Summary

By carefully choosing between log compaction, time-based compaction, and setting appropriate retention times, you can ensure that your Kafka topic supports both real-time streaming and periodic batch processing efficiently. Regular monitoring and fine-tuning of these settings will help maintain a balanced storage strategy, optimizing for both performance and cost.

#### Common Mistakes/Follow-Ups

Mistake: Ignoring the trade-offs between compaction type and retention policies.

#### Follow-Up
Explain why it’s important to consider both stream and batch processing needs simultaneously.

Mistake: Setting overly long retention times, leading to increased storage costs without necessary benefits for batch processes.

#### Follow-Up
Discuss how you would balance between retaining enough data for batch processes while minimizing unnecessary storage usage.

**Comparison:**

```
Compaction Type	Description	Use Case
Log	Retains the latest value for each unique key.	Suitable when state changes are infrequent.
Time-Based	Retains messages based on a defined time window.	Ideal for periodic batch reloads, where old data is eventually discarded.
```

**Code:**

```yaml
# Example Kafka topic configuration with compaction and retention settings

kafka_topic_config:
  "compaction.window.ms": 604800000 # Time-based compaction window of 7 days
  "retention.ms": 604800000        # Retention time set to 7 days
```

**Example input / output:**

```text
# Example dry run input -> output

Input:
- Periodic batch reload starts, needs data from the last 7 days.

Output:
- Kafka topic returns all messages within the last 7 days, ensuring both stream and batch consumers get relevant data.
```

**Data:**

```json
{
  "compactionWindowMs": 604800000, // Time-based compaction window of 7 days
  "retentionMs": 604800000        // Retention time set to 7 days
}
```

**Flow:**

1. Stream Consumer
2. Check Topic Compaction andRetention
3. Fetch Latest Data
4. Batch Process
5. Check Topic Compaction andRetention
6. Access Historical Data for 7Days

