---
type: "practice"
difficulty: "Deep"
---

# How would you monitor UnderReplicatedPartitions and OfflinePartitionsCount, and what does each alert tell you?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Kafka Streams and Event Time](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Kafka_Streams_and_Event_Time/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

In a Kafka cluster, monitoring UnderReplicatedPartitions and OfflinePartitionsCount is crucial to ensure data reliability and system availability. These metrics provide insights into partition replication issues that can affect data durability and consumer groups' ability to consume messages.

#### Triage

The question probes your understanding of Kafka's replication mechanism, the importance of partition management, and how to effectively monitor these critical metrics using a combination of operational tools and custom scripts. Key assumptions include familiarity with Kafka's internal mechanics, monitoring tools like Prometheus or Grafana, and understanding of the implications of these alerts.

#### First Principles Solution
UnderReplicatedPartitions: This metric indicates partitions that do not have enough replicas to meet the required replication factor. An alert for UnderReplicatedPartitions suggests potential issues with leader election or partition reassignment.
OfflinePartitionsCount: This metric tracks partitions that are completely offline, meaning no replicas in any of the brokers can serve requests for those partitions. It highlights severe data unavailability.
Step-by-Step Solution

UnderReplicatedPartitions:

Alert Criteria: Typically, this alert is triggered when a partition has fewer replicas than its replication factor.
Monitoring Tools: Use Kafka's built-in metrics or external monitoring tools like Prometheus to track the kafka_partition_under_replication metric.
Impact Analysis: A high number of under-replicated partitions can lead to data loss, increased risk of outages, and reduced fault tolerance. It may indicate issues such as broker failures, network partitioning, or misconfiguration.

OfflinePartitionsCount:

Alert Criteria: This alert is triggered when a partition has no active replicas across all brokers.
Monitoring Tools: Monitor the kafka_partition_offline metric using Prometheus or Grafana.
Impact Analysis: Offline partitions mean that consumers cannot access data from those partitions, leading to potential service disruptions. It could be due to complete broker failures, network issues, or incorrect topic configuration.

#### Implementation
Steps

Setup Monitoring:

Configure Kafka's built-in metrics collection and expose them via a monitoring solution like Prometheus.
Set up alert rules in Grafana or any other visualization tool to trigger notifications when these metrics exceed predefined thresholds (e.g., UnderReplicatedPartitions > 0, OfflinePartitionsCount > 0).

Custom Scripts:

Write custom scripts using Kafka’s admin client API to periodically check the state of partitions and generate alerts if necessary.
For example, a script could run every hour, gather partition states from all brokers, and send an alert via email or Slack if any under-replicated or offline partitions are found.

Root Cause Analysis:

When an alert is triggered, conduct a root cause analysis to determine the underlying issue (e.g., broker failure, network outage).
Implement corrective actions such as restarting brokers, rebalancing partitions, or adjusting replication factors based on the findings.

#### Closing Summary

Monitoring UnderReplicatedPartitions and OfflinePartitionsCount is essential for maintaining Kafka cluster reliability. These alerts help in identifying potential issues related to data durability and availability, allowing timely interventions to prevent service disruptions. The solution involves setting up robust monitoring tools, configuring alert rules, writing custom scripts if needed, and conducting thorough root cause analysis.

#### Common Mistakes/Follow-Ups
Mistake: Ignoring the UnderReplicatedPartitions alert, leading to data loss.

#### Follow-Up
Discuss how to balance replication factors to minimize under-replication without compromising on fault tolerance.
Mistake: Misinterpreting OfflinePartitionsCount as just a network issue when it could be due to broker failures.

#### Follow-Up
Explain the importance of checking both broker and network status for accurate diagnostics.

**Comparison:**

```
tableChart
cols: "Metric, Description"
rows:
    "UnderReplicatedPartitions, Partitions with fewer replicas than the replication factor."
    "OfflinePartitionsCount, Partitions with no active replicas across all brokers."
```

**Code:**

```python
import kafka.admin

def check_partitions_status(brokers):
    admin_client = kafka.admin.AdminClient({'bootstrap.servers': brokers})
    topic_names = admin_client.list_topics().topics
    under_replicated_partitions = 0
    offline_partitions_count = 0

    for topic_name in topic_names:
        partitions_metadata = admin_client.describe_topic(topic_name).partitions_metadata
        for partition_metadata in partitions_metadata:
            if not partition_metadata.isr_leaders and len(partition_metadata.replicas) < partition_metadata.replication_factor:
                under_replicated_partitions += 1
            elif not any(replica.server_id > 0 for replica in partition_metadata.replicas):
                offline_partitions_count += 1

    return under_replicated_partitions, offline_partitions_count

brokers = "localhost:9092"
under_replicated_partitions, offline_partitions_count = check_partitions_status(brokers)
print(f"UnderReplicatedPartitions: {under_replicated_partitions}")
print(f"OfflinePartitionsCount: {offline_partitions_count}")
```

**Example input / output:**

| Input Parameters | Output Metrics |
|------------------|----------------|
| Brokers: `localhost:9092` | UnderReplicatedPartitions: 1 <br> OfflinePartitionsCount: 0 |

**Data:**

```json
{
  "brokers": "localhost:9092",
  "replication_factor": 3,
  "partitions_metadata": [
    {
      "partition_id": 0,
      "replicas": [1, 2, 3],
      "isr_leaders": true
    },
    {
      "partition_id": 1,
      "replicas": [1, 4, 5],
      "isr_leaders": false
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Check Kafka Cluster
6. Are partitionsunder-replicated?
7. Alert forUnderReplicatedPartitions
8. Are partitions offline?
9. Alert for OfflinePartitionsCount
10. No alerts needed
11. Check root cause and takecorrective action
12. Check root cause and takecorrective action

