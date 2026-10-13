---
type: "practice"
difficulty: "Expert"
---

# What is cooperative sticky (incremental) rebalancing and how is it better than eager rebalancing?

**Difficulty:** Expert · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Kafka Capacity and Cost](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Kafka_Streams_and_Event_Time/Kafka_Capacity_and_Cost.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Cooperative sticky incremental rebalancing in Apache Kafka is a strategy designed to improve partition reassignment by reducing disruption during cluster changes, especially when nodes are added or removed. It's an enhancement over eager rebalancing, which can cause significant load imbalance and data movement when reassigning partitions immediately upon topology changes.

#### Triage

The question asks for the explanation of cooperative sticky incremental rebalancing (CSI) in Kafka and why it is superior to eager rebalancing. We need to understand the current state of Kafka's partition reassignment process, explain how CSI addresses limitations, and discuss its benefits in terms of performance and stability.

#### First-Principles Solution
Understanding Eager Rebalancing

Eager rebalancing involves immediately redistributing partitions across the new topology as soon as a node is added or removed. This can lead to sudden changes in load distribution and increased network traffic, causing potential disruptions for consumers.

Introducing Cooperative Sticky Incremental Rebalancing (CSI)

Cooperative sticky incremental rebalancing, introduced in Kafka 2.8, aims to mitigate these issues by delaying partition reassignment until it’s more manageable. Here are the key aspects:

Sticky Assignment: Partitions are assigned to brokers based on a "sticky" algorithm that tries to keep partitions with similar properties together.
Incremental Rebalancing: Instead of immediately redistributing all partitions, Kafka waits for new data to arrive or updates in metadata before performing incremental reassignments.
Cooperation Across Brokers: Brokers communicate and coordinate their actions to ensure that the overall load remains balanced.
Detailed Explanation

Sticky Assignment:

In eager rebalancing, each partition can be reassigned independently based on current node capacity. This might lead to partitions being spread across brokers unevenly.
CSI ensures that partitions with similar characteristics (e.g., same topic, same replication factor) are assigned together, reducing the risk of imbalanced load.

Incremental Rebalancing:

Kafka delays the actual reassignment until there is a new batch of data to be processed or metadata updates occur.
This incremental approach allows brokers to adjust their loads gradually without causing significant disruption.

Cooperation Across Brokers:

Kafka brokers coordinate with each other to ensure that no broker becomes overloaded during the rebalancing process.
They use a raft-like consensus protocol to agree on reassignment strategies, ensuring that the cluster remains stable and performant.
Advantages of CSI Over Eager Rebalancing
Reduced Disruption:
By delaying full rebalances, Kafka can handle changes more gracefully, minimizing downtime for consumers.
Improved Load Balancing:
Sticky assignment helps maintain a balanced load across brokers, reducing the likelihood of hotspots or cold spots in the cluster.
Better Throughput and Latency:
Incremental reassignment allows brokers to adjust their loads more smoothly, leading to higher throughput and lower latency.

#### Common Mistakes
and Follow-Ups
Misunderstanding Sticky Assignment: Failing to recognize that partitions are not reassigned independently but based on sticky algorithms can lead to suboptimal load balancing.
Ignoring Incremental Nature: Assuming that all rebalances will be immediate can result in underestimating the complexity of managing dynamic clusters.

#### Closing Summary

Cooperative sticky incremental rebalancing is a sophisticated strategy designed to improve Kafka's partition reassignment process by reducing disruptions, maintaining better load balance, and enhancing overall cluster performance. It addresses the limitations of eager rebalancing by delaying full reassignments until it’s more manageable, ensuring that brokers can coordinate effectively.

**Comparison:**

```
Broker ID	Partitions
1	topic1-part-0, topic1-part-1
2	topic1-part-2
3	topic1-part-3
4	None
New State (After Rebalance)
Broker ID
-----------
1
2
3
4

The table shows how partitions are reassigned to maintain load balance without immediate disruption.
```

**Code:**

```python
# Example Python snippet for sticky assignment (simplified)
class StickyAssignment:
    def __init__(self):
        self.broker_partitions = {}

    def assign_partition(self, broker_id, partition):
        # Logic to ensure partitions are assigned based on sticky criteria
        pass

def rebalance_cluster(new_topology):
    # Incremental reassignment logic using cooperation across brokers
    pass
```

**Example input / output:**

```text
Input: Kafka cluster with brokers [1, 2, 3] and a new broker added (4).
Output: Incremental reassignment of partitions to maintain load balance without immediate disruption.
```

**Data:**

```yaml
# Example YAML for metadata updates
brokers:
  - id: 1
    partitions:
      topic1-part-0: active
      topic1-part-1: active

  - id: 2
    partitions:
      topic1-part-2: active

  - id: 3
    partitions:
      topic1-part-3: inactive
```

**Flow:**

1. Yes
2. No
3. Broker Adds New Node
4. Metadata Update?
5. Incremental Rebalance
6. Sticky Assignment
7. Cooperate with Other Brokersfor Rebalance
8. No Immediate Changes, Waitfor Data

