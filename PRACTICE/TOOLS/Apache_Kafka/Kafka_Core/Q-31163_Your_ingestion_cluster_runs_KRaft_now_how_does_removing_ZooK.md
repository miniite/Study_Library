---
type: "practice"
difficulty: "Deep"
---

# Your ingestion cluster runs KRaft now; how does removing ZooKeeper change your operational runbook for controller failover and metadata during a broker outage?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ KRaft Mode](<../../../../TOOLS/Apache_Kafka/Kafka_Core/When_Not_to_Use_Kafka/KRaft_Mode.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the shift from ZooKeeper to KRaft in managing Kafka's controller failover and metadata, we need to understand that KRaft introduces a more distributed and fault-tolerant approach compared to the traditional centralized ZooKeeper setup. This change fundamentally alters how operational procedures should be managed.

Firstly, let’s break down what KRaft brings to the table:

Distributed Controller Election: KRaft operates without an external control plane like ZooKeeper. Instead, it uses a distributed consensus mechanism for electing the controller.
Self-Managing Cluster: The cluster itself is responsible for managing its state and metadata, reducing reliance on external components.

Now, let’s dive into how this change impacts your operational runbook:

#### Triage
Key Aspects Probed

The interviewer is probing:

Your understanding of KRaft's architecture.
Your ability to manage a Kafka cluster without ZooKeeper.
Operational strategies for handling broker outages in the new setup.

#### Assumptions
Kafka Version: We are discussing Kafka 3.x or later, which supports KRaft mode.
Cluster Configuration: The Kafka brokers are running with KRaft enabled.
Monitoring and Alerts: You have a robust monitoring system to alert on issues like broker failures.
Step-by-Step Solution

Controller Failover:

In the traditional ZooKeeper setup, the controller is elected by a ZooKeeper ensemble.
With KRaft, each broker runs as a raft peer and elects a leader. This leader acts as the controller.
If a broker fails, the remaining brokers will vote on a new leader (controller) according to the Raft consensus algorithm.

Metadata Management:

In ZooKeeper, metadata is stored in the ensemble's state directory.
KRaft stores this information directly within the Kafka cluster itself, using a dedicated namespace.
Metadata updates are coordinated through raft consensus among brokers.
Operational Runbook Adjustments

Monitoring and Alerts:

Ensure you have monitoring for broker health, controller leader election, and metadata changes.
Set up alerts to notify on failures or anomalies in the raft log or broker logs.

Failover Management:

Regularly check the status of raft logs and leader elections. Tools like kafka-topics.sh with KRaft options can help monitor these.
Perform regular health checks on brokers, focusing on network connectivity and storage health to prevent outages.

Metadata Handling During Broker Outage:

When a broker fails, the cluster will automatically elect a new controller leader. This process is managed internally without external intervention.
Ensure that metadata updates are consistent across all brokers. Tools like kafka-reassign-partitions.sh can help in managing partitions during failures.

Drain and Restart Procedures:

Develop procedures to safely drain a broker before restarting it, ensuring minimal disruption to the cluster.
Use Kafka’s built-in tools to manage topic replication and partition leadership.

Cluster Expansion and Reduction:

Plan for controlled expansion or reduction of the cluster by leveraging KRaft's ability to handle leader elections during these operations.

#### Closing Summary

Removing ZooKeeper in favor of KRaft simplifies some aspects but introduces new operational considerations. Key takeaways include a focus on internal cluster management, robust monitoring, and proactive failure handling. The shift towards distributed consensus means that the cluster must be carefully monitored to ensure smooth failover processes and metadata consistency.

#### Common Mistakes/Follow-Ups
Over-reliance on External Tools: Continuing to use ZooKeeper tools or configurations without understanding KRaft’s internal mechanisms can lead to errors.
Lack of Monitoring: Insufficient monitoring for raft log health, leader elections, or broker state can result in undetected issues.
Improper Failover Handling: Not having clear procedures for drain and restart operations can cause data loss or service disruptions.

Would you like me to elaborate on any specific aspect or provide a sample input-output scenario?

**Comparison:**

| Broker ID | State    | Raft Leader |
|-----------|----------|-------------|
| 1         | online   | true        |
| 2         | offline  | -           |
| 3         | online   | false       |

**Code:**

```python
# Example Python script to check Kafka cluster health using KRaft mode
import subprocess

def check_kafka_health():
    result = subprocess.run(['kafka-topics.sh', '--describe', '--bootstrap-server', 'localhost:9092'], capture_output=True, text=True)
    if "raft" in result.stdout:
        print("KRaft is active and the cluster health seems good.")
    else:
        print("Possible issues with KRaft or ZooKeeper setup.")

check_kafka_health()
```

**Example input / output:**

```text
Input:
- Check Kafka cluster health using KRaft mode.

Output:
- Kafka cluster health seems good. KRaft is active and the raft leader election appears to be functioning correctly.
- Possible issues with KRaft or ZooKeeper setup if "raft" keyword not found in output.
```

**Data:**

```json
{
  "brokers": [
    {"id": 1, "state": "online", "raft_leader": true},
    {"id": 2, "state": "offline"},
    {"id": 3, "state": "online", "raft_leader": false}
  ],
  "metadata": {
    "version": 5,
    "topics": [
      {"name": "topic1", "partitions": [{"leader_id": 1, "replicas": [2, 3]}]}
    ]
  }
}
```

**Flow:**

1. Broker Outage
2. Kafka detects failure
3. Leader election initiated byremaining brokers
4. New controller elected
5. Metadata updates propagatedto all brokers
6. Cluster continues operationwith new leader

