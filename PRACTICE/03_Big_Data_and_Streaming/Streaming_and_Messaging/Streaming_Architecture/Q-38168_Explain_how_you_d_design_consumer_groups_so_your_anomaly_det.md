---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you'd design consumer groups so your anomaly detector, your dashboard, and your archival sink all read the same telemetry topic independently without one slowing the others.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Consuming Untrusted Streams](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Consuming_Untrusted_Streams.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a real-time streaming architecture where an anomaly detector, a dashboard, and an archival sink each consume data from the same topic independently, we need to ensure that the consumer groups are configured correctly to avoid blocking one another. This involves understanding how Kafka consumer groups work and leveraging concepts such as partitioning, offset management, and consumer group isolation.

#### Triaging the Question

The key points being probed here include:

#### Cons
umer Group Management: How different consumer groups can be managed independently.
Partition Distribution: Ensuring that partitions are distributed among multiple consumers to achieve parallel processing.
Offset Management: Controlling the consumption offsets so that each consumer group operates correctly and doesn't interfere with others.
Architectural Trade-offs: Balancing between resource utilization, performance, and scalability.

#### First-Principles Solution

Partitioning Strategy:

We will use Kafka partitions to distribute the data among multiple consumer groups. Each partition represents a separate "stream" of data that can be consumed independently.

#### Cons
umer Group Configuration:

Configure each of the anomaly detector, dashboard, and archival sink as distinct consumer groups to ensure they operate independently.

Offset Management:

Use auto.offset.reset properties and manual offset management strategies to control how consumers start and continue consuming messages.

Parallelism and Load Balancing:

Ensure that the number of partitions matches or exceeds the number of consumer instances within each group to achieve parallel processing.
Detailed Steps

Setup Kafka Topic with Partitions:

BASH
# Create a topic with 3 partitions (you can scale this based on your needs)
kafka-topics --create --topic telemetry-topic --partitions 3 --replication-factor 1 --if-not-exists --zookeeper localhost:2181

Configure Consumer Groups:

Anomaly Detector Group:

PROPERTIES
# Configuration for anomaly detector consumer group
bootstrap.servers=kafka-broker-host:9092
group.id=anomaly-detector-group
auto.offset.reset=earliest
enable.auto.commit=false

Dashboard Group:

PROPERTIES
# Configuration for dashboard consumer group
bootstrap.servers=kafka-broker-host:9092
group.id=dashboard-group
auto.offset.reset=latest
enable.auto.commit=false

Archival Sink Group:

PROPERTIES
# Configuration for archival sink consumer group
bootstrap.servers=kafka-broker-host:9092
group.id=archival-sink-group
auto.offset.reset=earliest
enable.auto.commit=false

Implement Offset Management:

Use Kafka's ConsumerGroupOffsetManager to manage offsets manually if needed.

#### Example
of manual offset management using Python (Kafka-Python):
PYTHON
from kafka import TopicPartition, KafkaConsumer

#### cons
umer = KafkaConsumer(
    'telemetry-topic',
    bootstrap_servers=['kafka-broker-host:9092'],
    group_id='anomaly-detector-group'
)

for partition in range(consumer.partitions_for_topic('telemetry-topic')):
    tp = TopicPartition('telemetry-topic', partition)
    # Assuming we want to start from a specific offset

#### cons
umer.seek(tp, 100)  # Offset of 100

Load Balancing and Resource Utilization:

Ensure that the number of consumers per group matches the number of partitions to fully utilize the available resources.
Use Kafka’s load balancing policies if necessary.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By configuring distinct consumer groups with appropriate partitioning, we ensure that each service (anomaly detector, dashboard, archival sink) consumes data independently without blocking one another. Key takeaways include:

Proper use of partitions to distribute the workload.
Careful offset management to avoid conflicts.
Balancing resources and load for optimal performance.

#### Common mistakes
to watch out for include not understanding how auto.offset.reset works or misconfiguring partition distribution, which can lead to inefficient data consumption.

For follow-ups, one might ask about handling late arriving data or implementing strategies for fault tolerance in the consumer groups. Additionally, discussing how this setup scales with increasing telemetry could also be valuable.

**Comparison:**

| Consumer Group | Partitions | Auto Reset | Enabled |
|---------------|-----------|-----------|---------|
| Anomaly Detector | 3 | earliest | false |
| Dashboard | 3 | latest | false |
| Archival Sink | 3 | earliest | false |

**Code:**

```
PROPERTIES
Copy
# Example Kafka consumer group properties
bootstrap.servers=kafka-broker-host:9092
group.id=anomaly-detector-group
auto.offset.reset=earliest
enable.auto.commit=false

bootstrap.servers=kafka-broker-host:9092
group.id=dashboard-group
auto.offset.reset=latest
enable.auto.commit=false

bootstrap.servers=kafka-broker-host:9092
group.id=archival-sink-group
auto.offset.reset=earliest
enable.auto.commit=false
```

**Example input / output:**

```text
# Example Python code for manual offset management
from kafka import TopicPartition, KafkaConsumer

consumer = KafkaConsumer(
    'telemetry-topic',
    bootstrap_servers=['kafka-broker-host:9092'],
    group_id='anomaly-detector-group'
)

for partition in range(consumer.partitions_for_topic('telemetry-topic')):
    tp = TopicPartition('telemetry-topic', partition)
    consumer.seek(tp, 100)  # Offset of 100
```

**Data:**

```json
{
  "topic": "telemetry-topic",
  "partitions": [
    {"id": 0},
    {"id": 1},
    {"id": 2}
  ]
}
```

**Flow:**

1. Create Kafka Topic withPartitions
2. Configure Consumer Groups
3. Set up Offset ManagementStrategies
4. Implement Load Balancing andResource Utilization
5. Monitor and OptimizePerformance

