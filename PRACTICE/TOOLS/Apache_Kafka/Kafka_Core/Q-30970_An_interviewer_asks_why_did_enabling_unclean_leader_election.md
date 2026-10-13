---
type: "practice"
difficulty: "Deep"
---

# An interviewer asks: why did enabling unclean leader election on your telemetry cluster during an outage later corrupt your anomaly baselines, and what's the safer trade-off?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Leader Election and Unclean Failover](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Leader_Election_and_Unclean_Failover.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why enabling unclean leader election during an outage can corrupt your anomaly baselines in a Kafka cluster, we need to first grasp the core concepts of Kafka’s partition leadership and how unclean leader election works. This issue primarily revolves around data consistency and the potential for outdated or stale data to affect anomaly detection.

#### Triaging the Question

The interviewer is probing into the understanding of Kafka's partition leadership mechanism, specifically focusing on why enabling unclean leader election can lead to corrupted anomaly baselines during an outage. The key assumptions here are:

You have a Kafka cluster set up for telemetry data collection.
Anomaly detection relies on historical baseline data.
Enabling unclean leader election during an outage allows partitions to be elected as leaders even when the majority of replicas cannot agree.

#### First-Principles Solution
1. Partition Leadership and Replication

Kafka partitions are replicated across multiple brokers for fault tolerance. Each partition has a single leader that handles read/write operations, while replicas handle write acknowledgments. Normally, Kafka ensures that only leaders with the most recent state can be elected by enforcing majority agreement among replicas.

2. Unclean Leader Election Mechanism

During an outage where the majority of replicas cannot agree on a leader (quorum loss), unclean leader election allows any replica to become the new leader. This can happen even if that replica is behind in terms of data updates or has inconsistent data due to the outage.

3. Impact on Anomaly Baselines

Anomaly detection models rely heavily on historical baseline data, which typically represents normal behavior under various conditions. When unclean leader election occurs during an outage:

Partitions may be led by replicas with stale or outdated data.
This can introduce inconsistencies in the telemetry data collected post-outage, leading to skewed baselines.
4. Example Scenario

Imagine a Kafka cluster collecting telemetry data from IoT devices. An outage occurs where some replicas cannot communicate due to network issues. If unclean leader election is enabled and one of these replicas becomes a leader, it might start serving stale or inconsistent data. Post-outage, anomaly detection using the corrupted baselines will likely flag false anomalies.

#### Closing Summary

Enabling unclean leader election during an outage can corrupt your anomaly baselines because it allows partitions to be led by replicas with potentially outdated or inconsistent state. This disrupts the consistency of telemetry data and affects the accuracy of anomaly detection models, leading to false positives or negatives.

#### Common Mistakes/Follow-Ups
Misunderstanding Quorum Requirements: Not fully understanding that a quorum is required for normal leader election can lead to misconfiguration.
Overlooking Data Consistency: Ignoring the potential impact of unclean leader election on data consistency and baselines.
Neglecting Failure Scenarios: Failing to plan for and test failure scenarios, especially during outages.
Triaging Trade-offs
1. Risk-Averse Approach

Enabling only clean leader election ensures data consistency but may increase the risk of partition unavailability during an outage, potentially impacting service availability.

2. Balanced Approach: Clean Leader Election with Graceful Degradation

A safer trade-off is to enable clean leader election and implement mechanisms for graceful degradation. This approach minimizes data corruption risks while ensuring some level of availability even when outages occur.

Final Summary

To avoid corrupting anomaly baselines during an outage, it's crucial to carefully consider the trade-offs between data consistency and service availability. Enabling only clean leader elections can help maintain baseline accuracy but may compromise service in rare cases. Implementing robust failover strategies that prioritize data consistency can provide a balanced solution.

**Comparison:**

```
Scenario	Clean Leader Election	Unclean Leader Election
Availability	High (during outages)	Low (less likely, but possible)
Data Consistency	High	Low
```

**Code:**

```python
# Example of setting unclean leader election (not recommended during outages)
kafka_config = {
    "unclean.leader.election.enable": "true"
}
```

**Example input / output:**

```text
Input: Enable unclean leader election during an outage.
Output: Anomaly baselines may become corrupted due to stale or inconsistent data.
```

**Data:**

```json
{
  "topic_name": "telemetry",
  "partition_count": 10,
  "replication_factor": 3,
  "outage_duration": 24h,
  "corrupted_baseline_prob": 0.8
}
```

**Flow:**

1. No
2. Yes
3. Outage Occurs
4. Is Clean Leader ElectionEnabled?
5. Unclean Leader ElectionActivated
6. Partition Led by Stale Replica
7. Risk of Corrupted Baselines
8. Only Clean Leaders Can BeChosen
9. Baseline Data Consistent

