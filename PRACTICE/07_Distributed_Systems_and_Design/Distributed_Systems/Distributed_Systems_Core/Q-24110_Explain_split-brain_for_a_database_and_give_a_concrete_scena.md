---
type: "practice"
difficulty: "Deep"
---

# Explain split-brain for a database and give a concrete scenario where two primaries both accept writes and corrupt the data. What's the difference between preventing it with quorum/fencing and merely detecting it after the fact?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Ordering and Partition Keys](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Ordering_and_Partition_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Split-brain in distributed systems, especially databases, is a situation where multiple nodes believe they are primary and can accept writes independently. This leads to data corruption when these independent primaries accept conflicting writes simultaneously. To understand this concept deeply, we need to explore the mechanics of how split-brain occurs and why preventing it is crucial.

First, let's clarify what split-brain means in a distributed system context:

Split-brain happens when a network partition causes multiple nodes to think they are the only valid primary.
In a database setup with replication and failover mechanisms, this can lead to a scenario where writes are accepted by more than one node, resulting in data inconsistencies.

To illustrate, consider a simple three-node cluster configured for replication and automatic failover. If a network partition happens between nodes A and B, and C remains isolated, the system might automatically promote both A and B as primaries due to lack of communication with each other. Both A and B start accepting writes independently without knowing about the other.

Now, let's dive into a concrete scenario: Suppose we have a three-node database cluster: Node 1 (Primary), Node 2 (Secondary), and Node 3 (Secondary). The primary node is Node 1. In a sudden network partition, Node 2 and Node 3 are isolated from each other.

Node 2 and Node 3 both fail to hear from Node 1 for several minutes, so they promote themselves as primaries due to their local quorum conditions. Both start accepting writes independently:

Node 2 receives an update to customer records.
Node 3 also receives the same update independently.

When the network stabilizes and nodes can communicate again, both Nodes 2 and 3 will have conflicting versions of the data, leading to corruption or loss of consistency in the database.

Preventing split-brain versus detecting it:

Prevention with Quorum/Fencing:

The idea is to ensure that only a specific subset (quorum) of nodes can form a primary node.
For example, if we require a 2-out-of-3 quorum for any node to become primary, Node 2 and Node 3 would not be able to promote themselves as primaries without communication with each other.
Fencing techniques involve isolating unresponsive or misbehaving nodes from the network. This ensures that only one node can remain active at a time.

Detection After the Fact:

Detection involves monitoring and identifying split-brain conditions after they have occurred.
One approach is to use timestamping or version control on writes, where each write includes a timestamp. Upon reconciliation, nodes compare timestamps to detect conflicting updates.
Another method is to implement conflict resolution logic that automatically reconciles conflicts based on rules (e.g., latest write wins).

The difference lies in proactive vs. reactive measures:

Prevention with quorum/fencing ensures data consistency by preventing split-brain from happening at all.
Detection after the fact allows for some degree of inconsistency but provides mechanisms to recover and resolve conflicts post-facto.

In conclusion, while detection offers a way to handle split-brain when it occurs, prevention through robust quorum and fencing strategies is crucial for maintaining data integrity in distributed systems. Preventive measures are generally more reliable and can significantly reduce the risk of data corruption.

**Comparison:**

```
Node	State	Last Write Timestamp
1	Primary	500
2	Promoted	500
3	Secondary	498
```

**Code:**

```python
# Example Python code snippet for detecting conflicting writes using timestamps

def detect_conflict(write1, write2):
    return write1['timestamp'] != write2['timestamp']

write1 = {'data': 'update', 'timestamp': 1}
write2 = {'data': 'update', 'timestamp': 1}

if detect_conflict(write1, write2):
    print("No conflict detected.")
else:
    print("Conflict detected. Resolving...")
```

**Example input / output:**

```text
Input:
- Node1 receives write with timestamp 500
- Node2 and Node3 (isolated) receive same write with timestamp 500

Output:
Node2 promotes itself as primary due to quorum loss
Node3 does not promote but remains secondary
Node1 continues operation as usual
```

**Data:**

```json
{
  "node1": {
    "state": "primary",
    "last_write_timestamp": 500
  },
  "node2": {
    "state": "secondary",
    "last_write_timestamp": 499
  },
  "node3": {
    "state": "secondary",
    "last_write_timestamp": 498
  }
}
```

**Flow:**

1. Node 1
2. Network Partition
3. Node 2 and Node 3 isolated
4. Promote self as primary
5. Node 1 continues operation
6. Conflict detected after networkrecovery
7. Resolve conflicts usingtimestamps

