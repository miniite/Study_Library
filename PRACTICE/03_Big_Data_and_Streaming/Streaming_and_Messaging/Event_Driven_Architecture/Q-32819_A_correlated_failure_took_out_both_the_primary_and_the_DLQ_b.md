---
type: "practice"
difficulty: "Deep"
---

# A correlated failure took out both the primary and the DLQ because they shared a broker cluster, so failed messages had nowhere to go and were dropped. Explain why the DLQ must live in a different failure domain and how you'd architect that separation at scale.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Poison Messages and DLQs](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Poison_Messages_and_DLQs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an event-driven architecture with message queues, ensuring the resilience of your system is crucial. A common pitfall is when primary and DLQ (Dead Letter Queue) shares the same broker cluster, leading to correlated failures where messages get lost or are dropped. To address this issue, we need to ensure that the DLQ operates in a different failure domain, meaning it should be isolated from the primary queue's potential outages.

To architect such separation at scale, we will leverage multiple brokers and a well-thought-out deployment strategy. Here’s how we can do it:

Understanding the Problem: The shared broker cluster means that any failure affecting one part of the system affects both queues. We need to ensure that even if the primary queue fails, messages in the DLQ still have somewhere safe to land.

Key Concepts: Failure domains refer to groups of components that can fail together due to shared resources like network partitions or hardware failures. By spreading out our systems across different failure domains, we mitigate this risk.

#### Solution
Approach:

Use Multiple Broker Clusters: Deploy multiple broker clusters in separate availability zones (AZs) or regions.
Isolate DLQ: Place the DLQ in a dedicated cluster in a different AZ or region than the primary queue.
Synchronous Replication: Design the system to replicate messages from the primary queue to the DLQ synchronously, ensuring no message loss.

Detailed Architecture:

Broker Clusters: Deploy two separate broker clusters in different availability zones (e.g., AZ1 and AZ2). This ensures that even if one AZ goes down, the other can continue processing.
Primary Queue: Host your primary queue in AZ1.
DLQ Cluster: Host the DLQ cluster in AZ2.
Replication Mechanism: Implement a synchronous message replication mechanism between the two clusters. This could be achieved using a custom implementation or leveraging existing tools like Kafka’s MirrorMaker.

Operational Considerations:

Monitoring and Logging: Set up comprehensive monitoring and logging to detect any failures early.
Disaster Recovery Plan: Develop a robust disaster recovery plan, including regular backups and failover mechanisms.
Scalability and Load Balancing: Ensure both clusters are scalable and load-balanced to handle varying traffic patterns.

Failure Modes:

If AZ1 fails, the primary queue will stop processing messages, but since the DLQ is in a different AZ (AZ2), failed messages can still be processed.
Conversely, if AZ2 fails, only the DLQ’s processing would be affected, not the primary queue.

#### Closing Summary
By separating the primary and DLQ queues into different failure domains using multiple broker clusters, we ensure that message loss is minimized even in the event of a failure. This architecture leverages redundancy to provide high availability and resilience.

**Code:**

```
MERMAID
Copy
graph LR
    A[Primary Queue] --> B[Broker Cluster 1]
    B --> C{Failure}
    C -- yes --> D[DLQ]
    C -- no --> E[Processing Continues]

    F[DLQ] --> G[Broker Cluster 2]
    G --> H{Failure}
    H -- no --> I[Processing Continues]
```

**Example input / output:**

```text
Input:
- Primary Queue Message: {"id": "123", "payload": "message content"}

Output (Primary Processing):
- Message processed successfully.

Failure Scenario:
- AZ1 Fails

Output (DLQ Handling):
- DLQ receives the message and processes it.
```

**Data:**

```json
{
  "brokerClusters": {
    "primaryCluster": {
      "az": "AZ1",
      "hosts": ["host1", "host2", "host3"]
    },
    "dlqCluster": {
      "az": "AZ2",
      "hosts": ["host4", "host5", "host6"]
    }
  },
  "replication": {
    "mode": "synchronous",
    "interval": 10
  }
}
```

