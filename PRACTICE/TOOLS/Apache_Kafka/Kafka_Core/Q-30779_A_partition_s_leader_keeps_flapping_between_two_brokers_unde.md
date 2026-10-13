---
type: "practice"
difficulty: "Expert"
---

# A partition's leader keeps flapping between two brokers under load. How would you diagnose whether it's ISR churn, GC pauses, or network, and what's your fix?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Leader Election and Unclean Failover](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Leader_Election_and_Unclean_Failover.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a Kafka cluster, when a partition’s leader keeps flapping between two brokers, we need to diagnose the root cause to ensure optimal performance and stability. This scenario could be due to ISR (In-Sync Replicas) churn, garbage collection (GC) pauses on one of the brokers, or network issues. Here's how you can methodically approach this problem:

Understanding the Problem: Kafka maintains a group of in-sync replicas for each partition. If the leader changes frequently between two brokers, it could indicate instability with ISR membership.
Gather Information: Before jumping to solutions, gather detailed logs and metrics from all involved brokers to understand the current state.

**Code:**

```python
# Example log snippet
2023-11-15 14:47:36.234 [KafkaServer-1] INFO  - Leader change for topic-partition, new leader is broker-2
2023-11-15 14:48:06.452 [KafkaServer-1] INFO  - Leader change for topic-partition, new leader is broker-5
2023-11-15 14:49:23.121 [KafkaServer-1] INFO  - Leader change for topic-partition, new leader is broker-2
```

**Data:**

```json
{
  "leader_changes": [
    {"timestamp": 1631780856234, "broker_id": 2},
    {"timestamp": 1631780886452, "broker_id": 5},
    {"timestamp": 1631780963121, "broker_id": 2}
  ]
}
```

**Flow:**

1. Identify ISR Churn
2. Adjust min.insync.replicas
3. Check GC Logs
4. Optimize JVM Settings
5. Monitor Network Latency
6. Improve NetworkConfiguration
7. Fix Identified Issues
8. Stable Operation

