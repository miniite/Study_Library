---
type: "practice"
difficulty: "Deep"
---

# You must replicate a telemetry topic from an edge region to a central Kafka for warehouse loading; how does MirrorMaker 2 translate offsets so the central loader resumes correctly on failover?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Durability with acks and ISR](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Durability_with_acks_and_ISR.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When setting up replication between an edge region and a central Kafka cluster using Apache Kafka's MirrorMaker 2, one of the critical aspects is ensuring that data processing in the central Kafka can resume correctly from where it left off after a failover or restart. This involves understanding how MirrorMaker 2 manages offsets to maintain state across clusters.

MirrorMaker 2 uses a mechanism based on consumer groups and broker-based offset storage to replicate messages between Kafka clusters. The key challenge is ensuring that when a consumer in the central Kafka resumes processing, it starts from the correct offset, which might be different if there was a failover or data loss. This requires careful handling of offsets to guarantee consistency.

Let's break this down step by step:

Understanding Consumer Groups and Offsets: In Kafka, each message within a topic is assigned an offset (a unique number representing its position). Consumers are organized into groups, and each group has a leader that manages the offset state for all members of the group. When data is replicated using MirrorMaker 2, it essentially moves messages from one cluster to another, maintaining offsets to ensure correct processing.

Offset Storage Mechanism: By default, Kafka stores offsets in the __consumer_offsets topic within the same cluster. This means that when you replicate a topic using MirrorMaker 2, the offsets are stored locally in each cluster's __consumer_offsets topic. When replicating from an edge region to a central Kafka, this setup ensures that both clusters have their own offset states.

Failover and Resumption: If there is a failover or a need for resumption of processing in the central Kafka, MirrorMaker 2 needs to ensure that the consumer group resumes processing correctly. This involves checking the latest offsets stored in the __consumer_offsets topic of both clusters. When a new consumer joins the group, it will fetch and use the latest offset from the __consumer_offsets topic, ensuring continuity.

Handling Failures: In case of a failure or data loss, MirrorMaker 2 can also use snapshotting to restore offsets. Snapshots are periodic copies of topics that can be used to recover state in case of failures. This feature allows for more robust recovery and ensures that the central Kafka can resume processing from the last known good state.

#### Cons
umer Group Design: To ensure smooth failover, it's crucial to design consumer groups correctly. Each group should have a unique identifier and maintain its own state independently across clusters. This way, when a failover occurs, consumers in the new cluster can pick up where they left off without disrupting ongoing processing.

Monitoring and Validation: Regular monitoring is essential to ensure that offset replication is functioning as expected. Tools like Kafka's built-in metrics or external monitoring solutions can help track the state of offsets and detect any issues early on.

In summary, MirrorMaker 2 effectively manages offsets through a combination of local storage in __consumer_offsets topics and snapshotting mechanisms. This ensures that when processing resumes after a failover, it does so correctly from the latest known offset, minimizing data loss or duplication.

**Comparison:**

```
Consumer Group	Edge Region Offset State	Central Kafka Offset State
Group 1	Offset 500	Offset 600
Group 2	Offset 300	Offset 400
```

**Code:**

```python
# Example Python code snippet for understanding consumer group setup
from kafka import KafkaConsumer

def setup_consumer(topic, bootstrap_servers):
    consumer = KafkaConsumer(
        topic,
        bootstrap_servers=bootstrap_servers,
        auto_offset_reset='latest',
        enable_auto_commit=True,
        group_id="my-consumer-group"
    )
    return consumer

# Example of fetching and committing offsets
consumer = setup_consumer("telemetry", ["edge-region-broker1:9092"])
for message in consumer:
    print(f"Processing message with offset {message.offset}")
    # Process the message...
    # Committing offsets after processing
    consumer.commit()
```

**Example input / output:**

```text
Input:
- Kafka topic `telemetry` on edge region with data being produced.
- MirrorMaker 2 setup to replicate `telemetry` topic to central Kafka.

Output:
- Central Kafka cluster receives replicated messages from the edge region.
- Consumers in the central Kafka resume processing from correct offsets based on state stored in their respective `__consumer_offsets` topics.
```

**Data:**

```json
{
  "bootstrap_servers": ["edge-region-broker1:9092", "edge-region-broker2:9092"],
  "topic": "telemetry",
  "group_id": "my-consumer-group"
}
```

**Flow:**

1. No Failover
2. Yes Failover
3. Message Produced
4. MirrorMaker 2 ReplicatesMessage
5. Failover Occurs
6. Data Continues Processing
7. Central Kafka Resumes fromLatest Offset
8. Consumer Group Syncs withCentral Offset State

