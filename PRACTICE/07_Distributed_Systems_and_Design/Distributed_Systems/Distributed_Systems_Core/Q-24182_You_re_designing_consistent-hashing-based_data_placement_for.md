---
type: "practice"
difficulty: "Deep"
---

# You're designing consistent-hashing-based data placement for a distributed store and want to bound the load any single node can take. Explain why plain consistent hashing with vnodes is insufficient under a hot key, and how bounded-load consistent hashing or explicit hot-key sharding fixes it.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Sharding Vector Indexes](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Sharding_Vector_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a distributed store using consistent hashing, we often use virtual nodes (vnodes) to improve load distribution. However, while vnodes help with many cases of uneven data distribution, they fall short when dealing with "hot keys" – keys that are heavily accessed. Bounded-load consistent hashing or explicit hot-key sharding can address this issue.

#### Triage
and Assumptions

The question is probing our understanding of load balancing in distributed systems and the trade-offs between different hashing strategies. We must assume a scenario where some keys generate significantly more traffic than others, causing an imbalance in node loads. Our goal is to explain why plain consistent hashing with vnodes cannot adequately address this issue and introduce alternatives that do.

#### First-Principles Solution
Plain Consistent Hashing with VNodes

#### Cons
istent Hashing: This technique maps keys to nodes using a hash function, creating a circular ring where closer keys are stored on adjacent nodes.
VNodes: To handle uneven key distribution, each physical node is assigned multiple virtual nodes (e.g., 100 vnodes). If a key hashes to the range of one of these vnodes, it is placed on that node.
Hot Key Issue

#### Cons
ider a scenario where there are many "hot keys" – keys that generate disproportionately high access traffic. In this case:

Key Clustering: When a hot key maps to the same vnode across multiple physical nodes (due to the ring's circular nature), these nodes can become overloaded.
Load Imbalance: The other vnodes and keys will remain relatively idle, leading to inefficient use of resources.
Bounded-Load Consistent Hashing
Dynamic Load Management: This approach involves dynamically adjusting the number of nodes each hot key is mapped to based on the current load distribution.
Node Assignment Flexibility: Hot keys are placed across more nodes, ensuring that no single node bears an excessive load. For example, if a node becomes too busy, its vnodes can be redistributed to other nodes.
Explicit Hot-Key Sharding
Predefined Nodes for Hot Keys: Instead of relying on the ring and vnodes, explicitly assign hot keys to specific nodes.
Fixed Load Distribution: This ensures that no single node is overwhelmed by a high volume of requests from hot keys, maintaining even load distribution.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, while consistent hashing with vnodes improves load balancing, it cannot perfectly handle scenarios with hot keys due to key clustering. Bounded-load consistent hashing and explicit hot-key sharding provide more robust solutions by dynamically adjusting node assignments or explicitly managing the placement of hot keys.

#### Common mistakes
include assuming that plain consistent hashing will inherently solve all load imbalance issues without additional strategies for hot keys. Interviewers might follow up on specific implementation details, such as how to detect when a node is becoming overloaded and how to efficiently redistribute vnodes.

**Comparison:**

| Node | Current Load |
|------|--------------|
| node-1 | 30          |
| node-2 | 25          |
| node-3 | 45          |
| Node | New Load    |
|------|-------------|
| node-1 | 38         |
| node-2 | 27         |
| node-3 | 39         |

**Code:**

```python
# Example of dynamic load management in Bounded-Load Consistent Hashing

def distribute_hot_keys(hot_keys):
    current_load = {node: 0 for node in nodes}
    for key in hot_keys:
        # Simulate load detection and redistribution
        for node, vnodes in vnode_map.items():
            if any(key == vnode for vnode in vnodes):
                current_load[node] += 1

    threshold = max(current_load.values())
    overburdened_nodes = [node for node, load in current_load.items() if load > (threshold * 0.8)]

    # Redistribute hot keys to balance load
    for node in overburdened_nodes:
        for key in hot_keys:
            if key in vnode_map[node]:
                new_node = find_balanced_node(current_load)
                vnode_map[new_node].append(key)
                current_load[new_node] += 1

def find_balanced_node(current_load):
    # Implement a load-balancing function to choose the least loaded node
    pass
```

**Example input / output:**

```text
Input:
- Current load distribution: {node-1: 30, node-2: 25, node-3: 45}
- Hot keys to be managed: ["key-1", "key-2"]

Output:
- After detection and redistribution:
  - New load distribution: {node-1: 38, node-2: 27, node-3: 39}
```

**Data:**

```json
{
  "nodes": ["node-1", "node-2", "node-3"],
  "hot_keys": ["key-1", "key-2"],
  "vnode_map": {
    "node-1": [0, 1, 5],
    "node-2": [2, 6, 7],
    "node-3": [3, 4, 8]
  }
}
```

**Flow:**

1. Yes
2. No
3. Detect Overload
4. Is Node Overburdened?
5. Redistribute Hot Key
6. Check New Load
7. Continue Monitoring

