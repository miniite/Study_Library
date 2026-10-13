---
type: "practice"
difficulty: "Deep"
---

# Explain how a network-partition chaos experiment between two AZs would reveal whether your quorum-based store is genuinely CP. What client behavior on the minority side confirms it's rejecting writes?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Quorums and Consensus](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Quorums_and_Consensus.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand if our distributed system with a quorum-based consensus mechanism truly adheres to the CAP theorem’s consistency and partition tolerance (CP) principles, we need to conduct a network-partition chaos experiment between two Availability Zones (AZs). The key insight is that during such an experiment, the minority side should reject writes, thereby confirming true CP behavior.

The core of this problem lies in understanding how quorum-based consensus works. In a distributed system, a majority decision is reached by ensuring that more than half of the nodes agree on a piece of data. This mechanism ensures consistency and partition tolerance but can lead to situations where the minority side may encounter write failures during network partitions.

Let's break down the steps involved in conducting this experiment and analyzing its outcome:

Understand Quorum-Based Consensus: In a distributed system, a quorum is a subset of nodes that meets or exceeds the majority threshold for making decisions. For example, if you have 5 nodes (A, B, C, D, E), a simple majority would be any set of 3 nodes (e.g., A, B, C). The quorum-based mechanism ensures that writes are only accepted if a majority of nodes agree.

Network Partition Scenario: Imagine we partition the network between two AZs, say AZ1 and AZ2. Assume AZ1 contains nodes A, B, and C, while AZ2 has D and E. In this setup, any operation requiring a quorum will fail because it cannot achieve the necessary majority across both AZs.

Experiment Setup: To test CP behavior, we need to simulate a network partition where writes from one side (e.g., AZ1) are isolated from the other side (AZ2). We can use tools like Chaos Engineering platforms or manual network segmentation techniques to create this condition.

Client Behavior Analysis:

When the network partition occurs, write operations initiated on the minority side (AZ2 in our example) should fail.
On the majority side (AZ1), writes may succeed because it can form a quorum internally.
Clients on the minority side will experience error messages or timeouts indicating that their writes were not acknowledged.

Confirming CP Behavior:

If the client behavior on AZ2 consistently rejects writes, this confirms true CP behavior.
Conversely, if clients on AZ2 do not reject writes (e.g., they still succeed due to cached data or other mechanisms), it indicates that the system is not truly CP.

Why This Matters: Ensuring genuine CP behavior is crucial for maintaining data integrity and consistency across partitions. It helps prevent data loss in highly partitioned conditions, ensuring that no write operations are lost.

In summary, by conducting a network-partition chaos experiment between AZs and observing client behavior on the minority side, we can validate whether our quorum-based store truly implements CP principles. A consistent rejection of writes on the isolated side is the hallmark of true CP behavior.

**Comparison:**

```
Node	AZ1 (Connected)	AZ2 (Disconnected)
A	Yes	No
B	Yes	No
C	Yes	No
D	No	Yes
E	No	Yes
```

**Code:**

```python
# Example Python code to simulate a network partition for testing
def simulate_network_partition():
    # Assume initial state where all nodes are connected
    nodes = {'A': True, 'B': True, 'C': True, 'D': True, 'E': True}

    # Simulate a network partition between AZ1 and AZ2
    az1_nodes = ['A', 'B', 'C']
    az2_nodes = ['D', 'E']

    for node in nodes:
        if node not in ('D', 'E'):  # Assume only nodes D and E are affected by the partition
            nodes[node] = False

    return nodes, az1_nodes, az2_nodes
```

**Example input / output:**

```

Input:

Initial state of nodes
Network partition between AZ1 and AZ2

Output:

Partitioned state with only nodes D and E isolated
MARKDOWN
Copy
| Node | AZ1 (Connected) | AZ2 (Disconnected) |
|------|----------------|--------------------|
| A    | Yes            | No                 |
| B    | Yes            | No                 |
| C    | Yes            | No                 |
| D    | No             | Yes                |
| E    | No             | Yes                |
```

**Data:**

```json
{
  "nodes": ["A", "B", "C", "D", "E"],
  "AZ1": ["A", "B", "C"],
  "AZ2": ["D", "E"]
}
```

**Flow:**

1. Connects
2. Connects
3. Connects
4. Disconnects
5. Disconnects
6. Node A
7. AZ1
8. Node B
9. Node C
10. Node D
11. AZ2
12. Node E

