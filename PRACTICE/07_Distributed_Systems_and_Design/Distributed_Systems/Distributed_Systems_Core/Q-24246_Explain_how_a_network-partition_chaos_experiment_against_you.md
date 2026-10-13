---
type: "practice"
difficulty: "Deep"
---

# Explain how a network-partition chaos experiment against your quorum-replicated database would confirm it's genuinely CP. What behavior on the minority side (write rejection, read fencing) proves it, and how do you scope the experiment so a real bug can't cause customer data loss?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Quorums and Consensus](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Quorums_and_Consensus.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with distributed systems, ensuring consistency is a critical challenge, especially in a scenario where network partitions occur. A quorum-replicated database ensures strong consistency by requiring more than half of nodes to agree on any read or write operation within the same partition. To validate that your system truly adheres to the C (Consistency) property under these conditions, you can conduct a network-partition chaos experiment. This involves isolating one side of a quorum-replicated database and observing how it handles writes and reads.

The key idea is that if a write operation fails on the minority partition but read operations return correct data, this confirms that your system is genuinely CP (Consistent, Partition-Tolerant). In contrast, if the minority partition returns stale or incorrect data during reads after a network partition, then your system might have issues that could lead to data loss.

#### Triaging the Question

The question aims to assess how you would design and execute an experiment to verify consistency in a distributed database under network partitions. The core of the problem involves understanding quorum replication mechanisms, identifying failure modes, and ensuring robustness against bugs that could cause customer data loss.

#### Assumptions
Quorum Replication: The system uses a majority quorum for write operations.
Network Partitions: We can simulate network partitions to isolate one side of the database.

#### Cons
istency Requirements: The database must maintain consistent reads and writes under partitioned conditions.
Step-by-Step First Principles Solution

Setup Environment:

Install and configure a distributed database system with quorum replication (e.g., Apache Cassandra, Raft).
Ensure the system is properly configured to handle network partitions.

Simulate Network Partition:

Use tools like tc or iptables to simulate network partitions.
Isolate one side of the cluster while ensuring the other side remains operational.

Trigger Write Operations on Minority Side:

Perform write operations from applications or external clients targeting nodes in the minority partition.
Observe if these writes are rejected due to insufficient quorum, indicating genuine CP behavior.

Perform Read Operations on Majority Side:

Trigger read requests from the majority side.
Ensure reads return correct data without stale values.

Analyze Behavior and Scope Experiment:

Log detailed information about both successful and failed operations.
Check for any unexpected bugs or anomalies that could lead to data loss.

#### Common Mistakes
and Follow-Ups
Incorrect Configuration: Failing to properly configure quorum settings can result in false positives where the system does not reject writes as expected during a partition.
Data Loss: Simulating real-world scenarios might accidentally cause customer data loss if not carefully controlled. Ensure all experiments are isolated from production environments.
False Positives/Negatives: Misinterpreting write rejection or read fencing can lead to incorrect conclusions about the system's consistency properties.

#### Closing Summary

By conducting a well-designed network-partition chaos experiment, you can validate that your quorum-replicated database behaves consistently under partitioned conditions. The minority side rejecting writes while returning correct data during reads is a strong indicator of genuine CP behavior. Careful setup and monitoring are essential to avoid real bugs causing customer data loss.

#### Common Mistakes/Follow-Ups
Ensure all experiments are isolated.
Log detailed operation traces for analysis.
Test various network partition scenarios, including partial and full partitions.

**Comparison:**

| Scenario | Expected Write Result | Expected Read Result |
|----------|-----------------------|----------------------|
| Normal   | Accept                | Correct              |
| Partition | Reject (Quorum Failure) | Stale/Failed         |

**Code:**

```python
# Example Python script to simulate a network partition using iptables
import subprocess

def disable_network_interface(interface):
    command = f"sudo ip link set {interface} down"
    subprocess.run(command, shell=True)

def enable_network_interface(interface):
    command = f"sudo ip link set {interface} up"
    subprocess.run(command, shell=True)
```

**Example input / output:**

```text
# Example Input/Output for the Chaos Experiment

Input:
- Network partition simulated with `disable_network_interface` on node 1 (192.168.1.1)
- Write operation attempted to node 2 (192.168.1.2) and node 3 (192.168.1.3)

Output:
- Node 2 and 3 successfully accept the write requests
- Node 1 rejects the writes due to lack of quorum

Read operation performed on all nodes after partition:
- Nodes 2 and 3 return correct data
- Node 1 returns stale data or fails (depending on configuration)
```

**Data:**

```json
{
  "nodes": [
    {"id": 1, "ip": "192.168.1.1"},
    {"id": 2, "ip": "192.168.1.2"},
    {"id": 3, "ip": "192.168.1.3"}
  ],
  "quorum_size": 2
}
```

**Flow:**

1. Simulate Network Partition
2. Write Operations
3. Node Rejected
4. Nodes Accepted
5. Read Operations
6. Correct Data Returned
7. Stale/Failed Data Returned

