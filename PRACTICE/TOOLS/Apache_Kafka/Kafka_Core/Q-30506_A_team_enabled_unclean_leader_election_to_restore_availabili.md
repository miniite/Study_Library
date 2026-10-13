---
type: "practice"
difficulty: "Deep"
---

# A team enabled unclean.leader.election to restore availability during an outage and later found corrupted downstream data. Explain what happened and the safer path.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Leader Election and Unclean Failover](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Leader_Election_and_Unclean_Failover.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, a backend team restored the availability of their system by enabling unclean.leader.election in Kafka during an outage but later discovered that downstream systems were processing corrupted data. Let's break down what happened and explore safer alternatives.

TL;DR

Enabling unclean.leader.election can help restore Kafka cluster availability quickly, but it also relaxes consistency guarantees, leading to potential data corruption. A safer approach involves using best practices like idempotent consumers and ensuring strict consistency controls when necessary.

#### Triaging the Question

The core of this question revolves around understanding the trade-offs between availability and consistency in distributed systems, specifically Kafka. The team enabled unclean.leader.election to quickly bring the cluster back online during an outage but did not account for the implications on data integrity.

#### Assumptions
Outage Scenario: A critical node or nodes went down.
Unclean Leader Election: Enabled to elect a follower as leader without waiting for all replicas to confirm.
Downstream Systems: Consumed data from Kafka topics, which turned out to be corrupted due to the lack of strict consistency controls.

#### First Principles Solution

Understanding Unclean Leader Election:

unclean.leader.election is a mechanism in Kafka that allows a new leader to take over quickly if all followers are down.
This can help maintain cluster availability but risks data corruption since the new leader might not have the latest offsets, leading to potential duplicates or missing records.

Data Corruption Path:

When unclean.leader.election is enabled, a follower that was lagging behind could become the new leader.
The new leader may start serving requests before catching up with the latest offsets, causing inconsistencies in the downstream systems.
For example, if a transaction was partially committed, and then the leader changed, it might result in partial data being available to consumers.

Safe Practices:

Strict Consistency: Use strict consistency controls like session.timeout.ms and ensure that all transactions are fully committed before they are visible.
Idempotent Consumers: Design consumer applications to handle idempotent processing, ensuring messages can be safely reprocessed without causing duplicates or conflicts.
Replication Factor and Acknowledgments: Configure the replication factor appropriately and set appropriate ack levels to ensure that only fully replicated and committed data is served.

#### Closing Summary

Enabling unclean.leader.election can help maintain high availability, but it compromises data consistency. To safely handle such scenarios, adopt best practices like strict consistency controls, idempotent consumers, and proper replication settings. This approach ensures that even in the event of an outage, the system remains consistent and reliable.

#### Common Mistakes/Follow-Ups
Overlooking Consistency: Failing to enable strict consistency mechanisms can lead to data corruption.
Underestimating Idempotency: Not designing for idempotent behavior in consumer applications can result in duplicates or failures during recovery.
Insufficient Replication: Low replication factors increase the risk of data loss and inconsistency.

**Comparison:**

```
Configuration	Description	Impact
acks=all	Ensure all in-sync replicas acknowledge the write	Prevents partial data from being committed
retries=3	Retry if a message is not acknowledged by the required number of replicas	Ensures messages are fully replicated before serving
```

**Code:**

```python
# Example of configuring Kafka producer with strict consistency
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers=['localhost:9092'],
                         acks=all,  # Ensure all in-sync replicas acknowledge the write
                         retries=3)  # Retry if a message is not acknowledged by the required number of replicas
```

**Example input / output:**

| Topic | Partition | Offset | Message |
|-------|-----------|--------|---------|
| example_topic | 0 | 42 | {"key": "value"} |
| Action | Result |
|--------|--------|
| Enable unclean.leader.election | Cluster becomes available but data is corrupted. |
| Disable unclean.leader.election and use strict consistency | Data remains consistent but availability might be affected during outages. |

**Data:**

```json
{
  "topic": "example_topic",
  "replication_factor": 3,
  "partition_count": 10,
  "consumer_group": "group_id"
}
```

**Flow:**

1. Yes
2. No
3. Inconsistent
4. Consistent
5. Producer
6. Is Message Acked?
7. Commit Successfully
8. Retry
9. Consumer
10. Check Offset Consistency?
11. Rerun Processing
12. Process Data

