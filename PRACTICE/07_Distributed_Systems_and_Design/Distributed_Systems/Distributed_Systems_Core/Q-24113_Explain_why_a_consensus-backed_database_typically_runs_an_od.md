---
type: "practice"
difficulty: "Intermediate"
---

# Explain why a consensus-backed database typically runs an odd number of replicas. Walk me through the quorum sizes to tolerate 1 and 2 failures, and why a 3-node cluster survives only one node loss.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Quorums and Consensus](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Quorums_and_Consensus.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In distributed systems, particularly in consensus algorithms like Raft or Paxos, using an odd number of replicas is crucial to ensure fault tolerance without splitting the quorum. This approach ensures that a majority can always be achieved even if one node fails.

To understand this better, let's break down how quorum sizes work with 3-node and 4-node clusters.

**Comparison:**

```
Nodes	Quorum Size (1 Failure)	Quorum Size (2 Failures)
3	2	1
4	2	1
```

**Code:**

```python
# Example of calculating quorums for different failure scenarios in a 3-node cluster (A, B, C)
nodes = ['A', 'B', 'C']
quorum_size_1_fail = len(nodes) // 2 + 1 - 1 # For 1 failure: (3//2) + 1 - 1 = 2 nodes
quorum_size_2_fails = len(nodes) // 2 + 1 - 2 # For 2 failures: (3//2) + 1 - 2 = 1 node

# Example for a 4-node cluster (A, B, C, D)
nodes_4 = ['A', 'B', 'C', 'D']
quorum_size_1_fail_4 = len(nodes_4) // 2 + 1 - 1 # For 1 failure: (4//2) + 1 - 1 = 2 nodes
quorum_size_2_fails_4 = len(nodes_4) // 2 + 1 - 2 # For 2 failures: (4//2) + 1 - 2 = 1 node
```

**Example input / output:**

```text
For a 3-node cluster (A, B, C):
- To tolerate 1 failure: The quorum size is 2 nodes. For example, if node A fails, the remaining nodes (B and C) can still form a majority.
- To tolerate 2 failures: The quorum size reduces to 1 node. This means that any single node will be considered sufficient for making decisions.

For a 4-node cluster (A, B, C, D):
- To tolerate 1 failure: The quorum size is 2 nodes. For example, if node A fails, the remaining nodes (B, C, and D) can still form a majority.
- To tolerate 2 failures: The quorum size again reduces to 1 node. This means that any single node will be considered sufficient for making decisions.

This behavior ensures that the system remains highly available and resilient even with node failures.
```

**Data:**

```json
{
  "3-node-cluster": {
    "quorum_size_for_1_failure": 2,
    "quorum_size_for_2_failures": 1
  },
  "4-node-cluster": {
    "quorum_size_for_1_failure": 2,
    "quorum_size_for_2_failures": 1
  }
}
```

**Flow:**

1. 3-Node Cluster
2. 1 Failure
3. Quorum Size = 2
4. 1 Node for Majority
5. 4-Node Cluster
6. 1 Failure
7. Quorum Size = 2
8. 1 Node for Majority

