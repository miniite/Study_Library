---
type: "practice"
difficulty: "Intermediate"
---

# How would you throttle a kafka-reassign-partitions run on a busy ingestion cluster so replication traffic doesn't starve live producers and stall the warehouse feed?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Kafka Capacity and Cost](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Kafka_Streams_and_Event_Time/Kafka_Capacity_and_Cost.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of throttling a Kafka reassign-partitions operation during peak activity, we need to ensure that the ongoing replication tasks do not significantly impact active data ingestion and processing pipelines. This involves understanding the mechanics of partition reassignment in Kafka, identifying potential bottlenecks, and implementing strategies to mitigate their effects.

The key idea is to carefully manage the reassignment process by leveraging configuration parameters such as num.reassign.throttle.ms (which controls how frequently the reassignment task retries), and by adjusting other relevant settings. We will also use monitoring tools and possibly dynamic scaling mechanisms to ensure that the cluster remains stable during the reassignment.

**Comparison:**

```
Partition	Current Leader	New Leader
0	Node 1	Node 3
1	Node 2	Node 4
2	Node 3	Node 5
```

**Code:**

```python
# Example configuration snippet for Kafka producer properties
producer_config = {
    "bootstrap.servers": "kafka1:9092,kafka2:9092",
    "enable.idempotence": True,
    "acks": "all",
    "retries": 5,
    "batch.size": 32768,
    "linger.ms": 1,
    "compression.type": "gzip"
}
```

**Example input / output:**

```text
Input: Kafka cluster is under heavy load with many active topics and partitions. The `reassign-partitions` task needs to be executed.

Output: The reassignment process completes without causing significant disruption, ensuring live producers and the warehouse feed remain unaffected.
```

**Data:**

```json
{
    "kafka_cluster_stats": {
        "total_partitions": 1000,
        "in_use_partitions": 750,
        "free_partitions": 250,
        "current_ops": [
            {"topic": "user_events", "partition": 3, "op_type": "produce"},
            {"topic": "session_data", "partition": 15, "op_type": "consume"}
        ],
        "replication_factor": 3
    }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Initial State
6. Is Cluster Under Heavy Load?
7. Monitor Live Data Processing
8. Identify Critical Topics andPartitions
9. Configure Throttle Parameters
10. Execute Reassignment TaskNormally
11. Adjust Replication TasksDuring Throttle Period
12. Reassign Completed?
13. Monitor Post-ReassignmentStability

