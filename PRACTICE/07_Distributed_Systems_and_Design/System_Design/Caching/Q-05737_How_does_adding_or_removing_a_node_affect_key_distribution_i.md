---
type: "practice"
difficulty: "Expert"
---

# How does adding or removing a node affect key distribution in a sharded cache?

**Difficulty:** Expert · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Caching Expensive Queries](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Caching_Expensive_Queries/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a sharded cache, nodes are distributed across multiple servers to handle high loads and improve performance. The addition or removal of a node can significantly impact how keys are distributed among these nodes. Understanding this distribution is crucial for maintaining data locality and ensuring efficient cache operations.

#### Triaging the Question

The core of this question revolves around understanding how dynamic changes in the cache cluster (such as adding or removing nodes) influence key distribution, which directly impacts performance and consistency. The interviewer is probing your knowledge on sharding mechanisms, load balancing strategies, and the implications of node state changes on data management.

Key Assumptions
Sharded Cache: Data is distributed across multiple nodes.
Node State Changes: We consider scenarios where a node is added or removed from the cluster.
Key Distribution Strategy: The cache employs some form of hashing or consistent hashing to distribute keys among nodes.

#### First-Principles Solution
1. Initial Key Distribution

In a sharded cache, each key is mapped to a specific node using a hash function (or consistent hashing for more robust distribution). For example, if you have ( N ) nodes and ( K ) keys, each node will hold approximately ( \frac{K}{N} ) keys.

2. Adding a Node

When adding a node:

Key Rebalance: Some keys that were mapped to the removed node(s) need to be reassigned.
Hash Function Adjustments: The hash function or consistent hashing ring needs to be updated to include the new node.
Load Balancing: Ensure that the addition of a node does not cause an imbalance in key distribution. For instance, if you add a single node, it might take keys from one or more existing nodes.
3. Removing a Node

When removing a node:

Key Rebalance: Keys previously held by the removed node need to be reassigned.
Hash Function Adjustments: Update the hash function or consistent hashing ring to exclude the removed node.
Load Balancing: Ensure that removing a node does not leave gaps in key distribution. If multiple nodes are involved, this can lead to uneven load.
4. Impact on Performance

Node Addition:

Benefits: Improved capacity and fault tolerance.
Risks: Potential data migration overhead and temporary performance degradation during the rebalance phase.

Node Removal:

Benefits: Reduced resource usage if a faulty or underutilized node is removed.
Risks: Increased load on remaining nodes, potentially leading to overloading.

#### Closing Summary

Adding or removing a node in a sharded cache system requires careful consideration of key rebalancing and hash function adjustments. The goal is to maintain data locality and performance while ensuring that the cluster remains balanced and resilient.

#### Common Mistakes/Follow-Ups
Overlooking Key Rebalance: Failing to account for keys that need to be redistributed.
Neglecting Load Balancing: Ignoring potential imbalances caused by node state changes.
Insufficient Error Handling: Not having mechanisms in place to handle failures during key migration or rebalancing.

#### Example

#### Cons
ider a cache with 4 nodes and 100 keys. If we add one more node, the new distribution should be approximately:

Original Nodes: ~25 keys each
New Node: ~25 keys

If we remove one node, the remaining 3 nodes will need to handle the entire load of 100 keys.

**Comparison:**

```
Node	Keys
0	k1, k5, k9
1	k2, k6, k3
2	k4, k7
3	k8
```

**Code:**

```python
# Example function for consistent hashing
def hash_key(key):
    return hash(key) % 4  # Simplified example with 4 nodes

# Initial key distribution
keys = ['k1', 'k2', 'k3', 'k4', 'k5', 'k6', 'k7', 'k8', 'k9', 'k10']
nodes = {i: [] for i in range(4)}

for key in keys:
    nodes[hash_key(key)].append(key)

# Display initial distribution
print(nodes)
```

**Example input / output:**

```

Initial key distribution before adding a node:

JSON
Copy
{
  "0": ["k1", "k5", "k9"],
  "1": ["k2", "k6", "k3"],
  "2": ["k4", "k7"],
  "3": ["k8"]
}

After adding one more node (assuming consistent hashing), the distribution might look like:

JSON
Copy
{
  "0": ["k1", "k5", "k9"],
  "1": ["k2", "k6", "k3"],
  "2": ["k4", "k7"],
  "3": ["k8"]
}

Keys are now balanced across all nodes.
```

**Data:**

```json
{
  "keys": ["k1", "k2", "k3", "k4", "k5", "k6", "k7", "k8", "k9", "k10"],
  "nodes": {
    "0": ["k1", "k5", "k9"],
    "1": ["k2", "k6", "k3"],
    "2": ["k4", "k7"],
    "3": ["k8"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Add Node
6. Rebalance Keys
7. Is Key Mapped to RemovedNode?
8. Reassign Key
9. E
10. Remove Node
11. Rebalance Keys
12. Does Node Hold Critical Data?
13. Migrate Data
14. J

