---
type: "practice"
difficulty: "Intermediate"
---

# How would you monitor UnderReplicatedPartitions and consumer lag on the cluster carrying your observability pipeline, and which is more urgent for anomaly detection?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Consumer Lag Monitoring](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Consumer_Lag_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively monitor under-replicated partitions (URPs) and consumer lag in an AIOps setup using Kafka, we need to understand that both metrics are critical for maintaining the health of our observability pipeline. URPs indicate issues with partition replication across brokers, while consumer lag can signal backlogs or potential bottlenecks in data processing. Given your role as a mid-level engineer, let's break down the problem from first principles and ensure we cover all necessary aspects.

URPs are more urgent for anomaly detection because they directly impact data availability and consistency. If partitions are under-replicated, it means that not enough replicas exist to handle failures, leading to potential data loss or increased latency. Consumer lag, on the other hand, is more about data processing delays but can also indicate issues if left unchecked over a long period.

#### Triage

The question asks us to monitor URPs and consumer lag on the cluster carrying our observability pipeline and determine which metric is more urgent for anomaly detection. Here’s what we need to consider:

URPs: These metrics are essential for ensuring data durability and availability.

#### Cons
umer Lag: This provides insight into processing efficiency but less so in immediate criticality.

#### Assumptions
The Kafka cluster is being used as part of an observability pipeline, which processes real-time or near-real-time data.
We have access to a monitoring system that can collect metrics from the Kafka brokers and consumer groups.
Step-by-Step Solution

Identify URPs: Under-replicated partitions occur when fewer replicas exist for a partition than required by the replication factor. This can be detected using Kafka's metadata API or via tools like Confluent Cloud Monitoring, which provide real-time insights into cluster health.

PYTHON
# Example Python code snippet to check URP status (simplified)
from confluent_kafka.admin import AdminClient

def get_under_replicated_partitions(broker_list):
    admin_client = AdminClient({'bootstrap.servers': ','.join(broker_list)})
    partitions_info = admin_client.list_groups()
    urps = []
    for group in partitions_info.groups:
        group_partitions = admin_client.describe_group(group.id)
        for topic_partition in group_partitions.partitions_assignment:
            if not group_partitions.replica_fetches[topic_partition].is_leader():
                urps.append(topic_partition)
    return urps

Monitor Consumer Lag: Consumer lag can be monitored by tracking the offset lags of consumer groups relative to their assigned partitions. Tools like Kafka's built-in metrics and Prometheus with Kafka Exporter can provide this information.

PYTHON
# Example Python code snippet to monitor consumer lag (simplified)
from kafka import KafkaConsumer

def get_consumer_lag(consumer):
    current_offset = dict()
    topic_partitions = set()

    for tp in consumer.assignment():
        current_offset[tp] = 0
        topic_partitions.add(tp.topic)

    # Consume records to update offsets
    for msg in consumer:
        topic_partition = (msg.topic, msg.partition)
        if topic_partition in current_offset:
            current_offset[topic_partition] += 1

    max_lag = {tp: abs(partition.leader.offset - offset) for tp, partition in current_offset.items() if partition}
    return max_lag

Anomaly Detection: For anomaly detection, we can set thresholds and trigger alerts based on these metrics.

PYTHON
# Example threshold checks (simplified)
def check_anomalies(urps, lag):
    urp_threshold = 10  # Example threshold for URP count
    lag_threshold = 5000  # Example threshold in milliseconds

    if len(urps) > urp_threshold or max(lag.values()) > lag_threshold:
        return True
    else:
        return False

#### Closing Summary
and Common Mistakes/Follow-Ups

URPs are more urgent for anomaly detection because they directly impact data durability and availability. Monitoring tools like Confluent Cloud, Kafka's built-in metrics, or third-party solutions can help in real-time monitoring of both URPs and consumer lag.

#### Common mistakes
include:

Failing to set appropriate thresholds for alerts.
Overlooking the importance of URP issues compared to consumer lag.
Not having a robust alerting system that integrates with your existing monitoring stack.

#### Follow-ups
might involve discussing how these metrics are integrated into an AIOps dashboard, which tools and services are used, or what actions are taken when anomalies are detected.

**Comparison:**

```
Partition	Under-Replicated Status
topic1	Yes
topic2	No
Topic	Partition	Lag in ms
topic1	partition1	5000
topic2	partition2	3000
```

**Code:**

```python
from confluent_kafka.admin import AdminClient

def get_under_replicated_partitions(broker_list):
    admin_client = AdminClient({'bootstrap.servers': ','.join(broker_list)})
    partitions_info = admin_client.list_groups()
    urps = []
    for group in partitions_info.groups:
        group_partitions = admin_client.describe_group(group.id)
        for topic_partition in group_partitions.partitions_assignment:
            if not group_partitions.replica_fetches[topic_partition].is_leader():
                urps.append(topic_partition)
    return urps

from kafka import KafkaConsumer

def get_consumer_lag(consumer):
    current_offset = dict()
    topic_partitions = set()

    for tp in consumer.assignment():
        current_offset[tp] = 0
        topic_partitions.add(tp.topic)

    # Consume records to update offsets
    for msg in consumer:
        topic_partition = (msg.topic, msg.partition)
        if topic_partition in current_offset:
            current_offset[topic_partition] += 1

    max_lag = {tp: abs(partition.leader.offset - offset) for tp, partition in current_offset.items() if partition}
    return max_lag
```

**Example input / output:**

```

Input:

PYTHON
Copy
brokers = ["broker1.example.com", "broker2.example.com"]
consumer_groups = ["group1", "group2"]

Output:

URPs: [(topic1, partition1), (topic2, partition2)]
Consumer Lag: {(topic1, partition1): 5000, (topic2, partition2): 3000}
```

**Data:**

```json
{
  "brokers": ["broker1.example.com", "broker2.example.com"],
  "consumer_groups": ["group1", "group2"]
}
```

**Flow:**

1. Yes
2. No
3. Lag > Threshold
4. Check URP Status
5. URP Count > Threshold?
6. Alert and Take Action
7. Monitor Lag
8. Check Lag Threshold
9. Alert and Take Action

