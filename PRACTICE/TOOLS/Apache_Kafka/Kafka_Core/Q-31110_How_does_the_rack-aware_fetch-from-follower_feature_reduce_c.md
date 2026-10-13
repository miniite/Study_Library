---
type: "practice"
difficulty: "Deep"
---

# How does the rack-aware fetch-from-follower feature reduce cross-AZ cost for a consumer-heavy analytics workload, and what are its correctness caveats?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Kafka Capacity and Cost](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Kafka_Streams_and_Event_Time/Kafka_Capacity_and_Cost.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how the rack-aware fetch-from-follower feature in Apache Kafka reduces cross-AZ (availability zone) costs for a consumer-heavy analytics workload, we need to first delve into Kafka's replication mechanism and then explore how this feature optimizes data fetching.

Kafka ensures high availability and fault tolerance through its topic-partition-replica model. Each partition of a topic has multiple replicas distributed across brokers (nodes), which are typically grouped by availability zones (AZs) for redundancy and performance. In a consumer-heavy analytics workload, consumers often lag behind the producers, leading to frequent fetch requests from the leader nodes, which can be expensive if those nodes are in different AZs.

Rack-aware fetch-from-follower is designed to address this issue by allowing followers within the same rack (AZ) to handle fetch requests for partitions they replicate. This feature reduces cross-AZ network latency and cost by leveraging local storage resources.

To reason through this step-by-step, let's start from first principles:

Partition Distribution: Each partition in a Kafka topic is replicated across multiple brokers.
Replica Assignment: Replicas are assigned to different brokers within the same rack or across racks based on the replication factor and available broker nodes.
Fetch Requests: Consumers issue fetch requests to the leader node of a partition, which then sends data from its own storage or local replicas.

#### Triaging the Question

The interviewer is probing:

How Kafka's distributed architecture affects cost and performance for consumer-heavy workloads.
The trade-offs between local and cross-AZ network operations.
The correctness caveats associated with this feature.
Step-by-Step Solution

Replica Assignment: In a typical setup, replicas of partitions are distributed across brokers within the same rack (AZ) or across different racks to ensure high availability and fault tolerance.

Leader Selection: When a broker fails, another broker in the same rack takes over as the leader for that partition.

Fetch Requests: Consumers connect to the leader node to fetch data. If the leader is in a different AZ, cross-AZ network latency and cost can be significant.

Rack Awareness: The racks parameter in Kafka's replication strategy allows defining rack information for brokers. By setting up rack awareness, Kafka can ensure that followers within the same rack can handle fetch requests from their leaders more efficiently.

Fetch-from-Follower Mechanism: When a consumer sends a fetch request to the leader, if there is an available follower in the same rack (AZ) with the required data, the leader will route the request to that follower instead of going through the network. This reduces latency and cost by leveraging local storage resources.

Correctness Caveats:

Data Consistency: Ensuring that all replicas are up-to-date can be challenging. If a replica lags behind due to high write traffic, fetching from an older replica might lead to data inconsistency.
Failure Scenarios: In case of network issues or failures within the same rack, fetch-from-follower might not always work as expected.
Resource Management: Properly managing resources and ensuring that followers are not overloaded with too many fetch requests can be tricky.

#### Closing Summary

Rack-aware fetch-from-follower in Kafka significantly reduces cross-AZ costs for consumer-heavy analytics workloads by enabling local data fetching. However, it comes with trade-offs such as potential data inconsistency and resource management challenges. It is crucial to carefully configure rack awareness based on the specific workload characteristics to achieve optimal performance and correctness.

#### Common Mistakes/Follow-Ups
Misconfiguring rack information can lead to incorrect replica assignments.
Not accounting for network latency between racks might result in suboptimal performance even with local fetching.
Ignoring data consistency issues when using older replicas can cause data skew or stale reads.

**Comparison:**

```
Partition	Leader	Followers
logs-0	1	2
metrics-1	3	2
```

**Code:**

```python
# Example configuration snippet for Kafka broker settings (not actual code)
replica_fetch_rate = 500 # bytes per second
racks = {
    "broker1": "rack1",
    "broker2": "rack1",
    "broker3": "rack2"
}
```

**Example input / output:**

```text
Consumer sends fetch request to Leader (Broker1) for Partition 0 of Topic "logs".
Leader Broker1 checks if any follower in rack1 has the required data.
If yes, it routes the fetch request to Follower Broker2 within the same rack.
```

**Data:**

```json
{
  "brokers": [
    {"id": 1, "host": "broker1", "rack": "rack1"},
    {"id": 2, "host": "broker2", "rack": "rack1"},
    {"id": 3, "host": "broker3", "rack": "rack2"}
  ],
  "partitions": [
    {"topic": "logs", "partition_id": 0, "replicas": [1, 2]},
    {"topic": "metrics", "partition_id": 1, "replicas": [2, 3]}
  ]
}
```

**Flow:**

1. Consumer
2. Leader
3. Follower in same rack

