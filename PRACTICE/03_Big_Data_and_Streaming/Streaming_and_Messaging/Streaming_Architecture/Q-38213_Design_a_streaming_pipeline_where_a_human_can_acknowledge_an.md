---
type: "practice"
difficulty: "Intermediate"
---

# Design a streaming pipeline where a human can acknowledge an alert and that ack flows back to suppress duplicate pages for the same incident. How does the ack event propagate and how long does suppression last?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Duplicates During Rebalance](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Duplicates_During_Rebalance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: The key idea is designing a streaming pipeline where real-time alerts can be acknowledged, with an acknowledgment propagating to suppress duplicate pages for the same incident. The solution involves leveraging Apache Kafka for event ingestion and processing, along with ACK events being stored in a database like Cassandra or DynamoDB for reliable deduplication.

#### Triage
This question is probing your understanding of streaming architectures, event propagation, and handling duplicates in real-time systems. Assumptions include using open-source tools, typical AIOps use cases, and the need for reliability in acknowledging alerts to prevent redundant notifications. We'll design a system where an ACK from a human operator results in suppression of further duplicate pages.

Step-by-Step Solution:

Event Ingestion: Use Apache Kafka as the message broker due to its high throughput, fault tolerance, and ability to scale horizontally.
Real-time Processing: Implement a stream processor (e.g., Apache Flink) to handle real-time data processing and ACK handling.
ACK Propagation: Design an ACK mechanism that stores the incident ID and timestamp in a database like Cassandra or DynamoDB for deduplication logic.
Suppression Logic: Define the suppression period, e.g., 24 hours, after which new alerts can be generated.

#### First-Principles Solution

Architecture Overview:

Kafka Topic for Alerting
Flink for Real-time Processing
Cassandra/DynamoDB for ACK Storage

Kafka Setup:

Create a topic alerts to ingest real-time alerts.
Use Kafka Streams or KSQL for initial filtering and sending ACKs.

Flink Stream Processor:

JAVA
// Pseudo-code for Flink stream processor
env = StreamExecutionEnvironment.getExecutionEnvironment()
kafkaStream = env.addSource(new FlinkKafkaConsumer<>("alerts", new SimpleStringSchema(), properties))

processedStream = kafkaStream.map(lambda alert: (alert.id, alert.timestamp))
  .keyBy("id")
  .filter(lambda (id, timestamp): !ackDB.containsKey(id) || (System.currentTimeMillis() - timestamp < suppressionPeriod))
  .addSink(new KafkaProducerSink<>("ack", new SimpleStringSchema(), ackProperties))

ACK Handling:

JAVA
// Pseudo-code for ACK handling in Flink
kafkaAckStream = env.addSource(new FlinkKafkaConsumer<>("ack", new SimpleStringSchema(), ackProperties))

processedAck = kafkaAckStream.map(lambda ack: (ack.id, ack.timestamp))
  .keyBy("id")
  .reduce((a1, a2) -> {
    // Update database with latest ACK time
    return a2;
  })
  .addSink(new CassandraSink<>("acks", keySpaceName))

Suppression Logic:

Store ACKs in a database with a composite key (incident ID and timestamp).
Define the suppression period, e.g., 24 hours.
Check if an incident has been acknowledged within this period before re-sending alerts.

#### Example
Worked Input-Output:

PLAINTEXT
Initial Alerts:
Alert1 (ID=001) at 9:00 AM -> Kafka Topic

Processed Alerts by Flink:
Alert1 (ID=001) at 9:05 AM -> Filtered, no ACK yet -> Kafka Topic for Ack

ACK Received from Human:
ACK1 (ID=001) at 9:30 AM -> Kafka Ack Topic

Processed ACK by Flink:
ACK1 (ID=001) at 9:35 AM -> Stored in Cassandra, key = 001, timestamp = 9:30 AM

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Ensure high availability and fault tolerance by using multi-datacenter Kafka setups.

#### Cons
ider edge cases such as network latency or ACK delays.
Discuss trade-offs between performance and cost when choosing databases like Cassandra vs. DynamoDB.

**Comparison:**

| Incident ID | Last ACK Timestamp |
|-------------|--------------------|
| 001         | 2023-10-05T09:30Z  |

**Code:**

```java
// Pseudo-code for Flink stream processor
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaProducer;

env = StreamExecutionEnvironment.getExecutionEnvironment()
kafkaStream = env.addSource(new FlinkKafkaConsumer<>("alerts", new SimpleStringSchema(), properties))
processedStream = kafkaStream.map(lambda alert: (alert.id, alert.timestamp))
  .keyBy("id")
  .filter(lambda (id, timestamp): !ackDB.containsKey(id) || (System.currentTimeMillis() - timestamp < suppressionPeriod))
  .addSink(new KafkaProducerSink<>("ack", new SimpleStringSchema(), ackProperties))

kafkaAckStream = env.addSource(new FlinkKafkaConsumer<>("ack", new SimpleStringSchema(), ackProperties))
processedAck = kafkaAckStream.map(lambda ack: (ack.id, ack.timestamp))
  .keyBy("id")
  .reduce((a1, a2) -> {
    // Update database with latest ACK time
    return a2;
  })
  .addSink(new CassandraSink<>("acks", keySpaceName))
```

**Example input / output:**

```text
Initial Alerts:
- Alert1 (ID=001) at 9:00 AM -> Kafka Topic

Processed Alerts by Flink:
- Alert1 (ID=001) at 9:05 AM -> Filtered, no ACK yet -> Kafka Topic for Ack

ACK Received from Human:
- ACK1 (ID=001) at 9:30 AM -> Kafka Ack Topic

Processed ACK by Flink:
- ACK1 (ID=001) at 9:35 AM -> Stored in Cassandra, key = 001, timestamp = 9:30 AM
```

**Data:**

```json
{
  "alerts": {
    "topic": "alerts",
    "partitionCount": 3,
    "replicationFactor": 1
  },
  "ack": {
    "topic": "ack",
    "partitionCount": 2,
    "replicationFactor": 1
  }
}
```

**Flow:**

1. No
2. Yes
3. Yes
4. Alert
5. Is ACKed?
6. Check Suppression Period
7. Suppress Alert
8. Acknowledge Alert

