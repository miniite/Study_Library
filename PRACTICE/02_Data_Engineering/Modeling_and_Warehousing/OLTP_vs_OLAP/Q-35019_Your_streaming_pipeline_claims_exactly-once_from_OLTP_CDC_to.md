---
type: "practice"
difficulty: "Expert"
---

# Your streaming pipeline claims exactly-once from OLTP CDC to the OLAP sink, but you see duplicate facts after a broker failover — how do you locate where the guarantee actually broke?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Feeding OLAP from OLTP](<../../../../02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Separating_OLTP_and_Analytics/Feeding_OLAP_from_OLTP.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of duplicates in an OLAP sink despite claiming exactly-once processing from an OLTP CDC source, we need to methodically investigate the system architecture and understand the nuances between OLTP and OLAP systems. The key is to pinpoint where the exactly-once guarantee might have been breached during a broker failover.

Firstly, let's clarify the concepts:

OLTP (Online Transactional Processing): Systems designed for frequent transactional operations like insertions, updates, and deletions.
CDC (Change Data Capture): Captures changes in real-time as they occur in OLTP systems.
OLAP (Online Analytical Processing): Systems optimized for complex queries over large datasets.

The streaming pipeline is expected to ensure exactly-once processing from the CDC source. However, duplicate facts are appearing after a broker failover. This suggests that somewhere between the CDC and the OLAP sink, the guarantee was broken. We need to methodically check each component:

#### Triage
Understand the Streaming Pipeline: The pipeline likely involves CDC capturing changes in a streaming format (e.g., Kafka), processing transformations, and writing into an OLAP store.
Broker Failover Mechanism: During failover, the streaming broker might have temporarily paused or resumed data processing, leading to duplicates if the exactly-once guarantee was not maintained properly.
Step-by-Step Solution

Check CDC Source and Broker Configuration:

Ensure that the CDC source is correctly configured for exactly-once processing.
Verify the Kafka brokers' configuration settings such as acks, transactional.id, and replica.fetch.max.bytes to ensure they support the required guarantees.

Analyze the Streaming Processor:

Examine the state management mechanism of the streaming processor (e.g., Flink, KStreams).
Ensure that the state store is correctly versioned and transactions are properly managed across failovers.

Examine the OLAP Sink:

Verify the write semantics in the OLAP sink (e.g., PostgreSQL, Redshift) to ensure they support exactly-once writes or have mechanisms to handle duplicates.

Review Log Files and Metrics:

Analyze the logs from each component during the failover event.
Use metrics like Kafka offsets, Flink task manager state, and OLAP sink write operations to pinpoint where the processing went awry.

Investigate Data Consistency:

Perform a dry-run of data flows through the system to simulate the failover scenario.
Use tools like Kafka’s kafka-console-consumer or Flink’s own monitoring tools to trace data paths and identify inconsistencies.

#### Closing Summary

By systematically checking each component of the streaming pipeline, we can pinpoint where exactly-once processing was not maintained. Likely culprits include misconfigurations in the CDC source, issues with state management during broker failover, or problems in the OLAP sink's write mechanisms.

#### Common Mistakes
and Follow-Ups
Mistake: Assuming that just because a system claims exactly-once guarantees, it will always work.

#### Follow-Up
Ensure all components are independently verified for exactly-once behavior. Consider using distributed ledger technologies or consensus protocols for highly critical scenarios where such guarantees must be absolutely maintained.

**Comparison:**

```
Kafka Topic	Partitions	Replication Factor
topic1	3	3
topic2	4	3
Flink Task Manager	State Backend	Transaction ID
manager-0	RocksDB	my-transaction-id
```

**Code:**

```java
// Example Kafka Consumer Configuration with Exactly-Once Processing
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("group.id", "my-group-id");
props.put("enable.auto.commit", "false"); // Disable auto-commit to manage offsets manually
props.put("transactional.id", "my-transaction-id"); // Enable transaction support

KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(Arrays.asList("topic-name"));
```

**Example input / output:**

```text
Input (Kafka): {"event_id": 123, "data": {"product_id": 456, "quantity": 1}}
Output (OLAP): Duplicate event detected for product_id 456 with event_id 123.
```

**Data:**

```json
{
  "kafkaOffsets": {
    "topic1-offsets": [0, 10],
    "topic2-offsets": [5, 15]
  },
  "flinkStateManager": {
    "checkpointIntervalMs": 60000,
    "numSavepointsToKeep": 3
  }
}
```

**Flow:**

1. Event Source
2. CDC to Kafka
3. Kafka Broker
4. Flink Stream Processor
5. OLAP Sink
6. Broker Failover
7. Duplicate Facts in OLAP

