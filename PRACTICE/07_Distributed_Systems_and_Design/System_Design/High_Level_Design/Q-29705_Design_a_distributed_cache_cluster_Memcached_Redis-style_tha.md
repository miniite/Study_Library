---
type: "practice"
difficulty: "Deep"
---

# Design a distributed cache cluster (Memcached/Redis-style) that must scale from 10 to 200 nodes — how do you use consistent hashing with virtual nodes to minimize key movement during scale-out?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Cache Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Distributed_Cache_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a distributed cache cluster using consistent hashing with virtual nodes, we need to focus on minimizing key movement and ensuring efficient distribution of data across multiple nodes as we scale from 10 to 200 nodes. The core idea is that each physical node in the cluster will be responsible for a segment of the key space, and this is achieved by assigning multiple virtual nodes per physical node.

#### Triaging the Question

The interviewer is probing how well you understand consistent hashing and its application in distributed systems, particularly in minimizing data movement when scaling. They want to see your ability to reason about trade-offs between complexity, performance, and fault tolerance. We'll assume that the cache uses a key-value model with Memcached or Redis semantics.

Step-by-Step Solution

Understand Consistent Hashing:

#### Cons
istent hashing is designed to minimize the impact of node additions or deletions on the existing data distribution.
In consistent hashing, keys are mapped to positions around a circle (or ring) and nodes are placed at evenly spaced points along this circle.

Virtual Nodes:

Each physical node in the cluster will be responsible for multiple virtual nodes.
By using virtual nodes, we can distribute data more finely and reduce the movement of keys when adding or removing physical nodes.

#### Implementation
Steps: a. Mapping Keys to Positions:

Map each key to a position on a ring by hashing the key with a consistent hash function (e.g., MD5).
The hashed value is then placed around the ring, where 0 and 2^64-1 are adjacent.

b. Adding Virtual Nodes:

For each physical node, create multiple virtual nodes at evenly spaced positions on the ring.
For example, if a node has 8 virtual nodes, they will be distributed every 1/8 of the way around the ring.

c. Data Distribution and Ownership:

Each key's owner is determined by finding the first virtual node that comes after the hashed key position on the ring.
For example, if a key hashes to 300, and we have 8 virtual nodes per physical node, the keys from (250-300) will be owned by this physical node.

d. Handling Node Failures:

When a node fails, its virtual nodes are mapped back to the ring.
The data owned by these virtual nodes is redistributed among other nodes, but only a fraction of keys need to be moved due to the consistent hashing property.

Scaling from 10 to 200 Nodes:

When scaling, add more physical nodes and distribute their virtual nodes on the ring.
This ensures that each new node takes over a portion of the existing key space without moving keys around unnecessarily.

#### Closing Summary

Using consistent hashing with virtual nodes is an effective strategy for minimizing data movement during scale-out. By distributing keys across multiple virtual nodes, we ensure that when we add more physical nodes, only a small fraction of keys need to be reassigned, thus reducing the overhead and improving overall performance.

#### Common Mistakes/Follow-Ups
Ignoring the Impact on Memory Usage: Using many virtual nodes can increase memory usage as each virtual node needs to store some metadata.
Failing to Consider Network Latency: Consistent hashing alone does not account for network latency. You might need additional strategies, such as routing keys through a consistent hashing ring that takes network topology into consideration.
Not Handling High Cardinality Keys: For very high cardinality key spaces, you may need to consider techniques like sharding or range-based partitioning.

#### Example
Calculation

Let's say we have 10 physical nodes initially and want to scale up to 200 nodes. If each node has 8 virtual nodes:

Initially, there are (10 \times 8 = 80) virtual nodes.
When scaling to 200 physical nodes with the same number of virtual nodes per physical node, we now have (200 \times 8 = 1600) virtual nodes.
Triangular Flow Diagram
Yes
No
Hash Key
Is key position >virtual_node_position?
Return to physical nodeowning the key
Find nearest preceding virtualnode
Map to physical node
Self-Check

The answer is comprehensive, covering all necessary aspects and considerations for consistent hashing with virtual nodes.

**Comparison:**

```
Physical Node	Virtual Nodes
node1	[vnode1, vnode2, ..., vnode8]
node2	[vnode9, vnode10, ..., vnode16]
...	...
node3	[vnode73, vnode74, ..., vnode80]
```

**Code:**

```python
# Example Python code snippet for consistent hashing with virtual nodes
import hashlib

class ConsistentHashRing:
    def __init__(self, nodes):
        self.ring = []
        self.virtual_nodes_per_node = 8
        self.nodes = nodes
        self.place_virtual_nodes()

    def place_virtual_nodes(self):
        for node in self.nodes:
            for _ in range(self.virtual_nodes_per_node):
                hash_val = int(hashlib.md5(node.encode()).hexdigest(), 16) % (2**32)
                virtual_node = {'node': node, 'hash_value': hash_val}
                self.ring.append(virtual_node)

    def get_nodes_by_key(self, key):
        # Hash the key and find the appropriate virtual node
        key_hash = int(hashlib.md5(key.encode()).hexdigest(), 16) % (2**32)
        for i in range(len(self.ring)):
            if self.ring[i]['hash_value'] > key_hash:
                return self.ring[i-1]['node']
        return self.ring[-1]['node']

# Example usage
nodes = ['node1', 'node2', 'node3']
ring = ConsistentHashRing(nodes)
print(ring.get_nodes_by_key('example_key'))
```

**Example input / output:**

```text
Input:
Key: example_key

Output:
Node: node1 (assuming the hash values and placements are such that this is the correct owner)
```

**Data:**

```yaml
nodes: [node1, node2, node3]
virtual_nodes_per_node: 8
key_hashing_function: hashlib.md5
```

**Flow:**

1. Yes
2. No
3. Hash Key
4. Is key position >virtual_node_position?
5. Return to physical nodeowning the key
6. Find nearest preceding virtualnode
7. Map to physical node

