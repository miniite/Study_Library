---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd implement a consistent-hashing ring for sharding sessions across cache nodes, and what happens to the key distribution when you add or remove a node mid-traffic.

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Hashing and Collisions](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hashing_and_Collisions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

#### Cons
istent hashing is an essential technique for distributed systems like caching services where we need efficient load balancing across multiple nodes while minimizing the impact of adding or removing nodes. By using a consistent hash ring, we can ensure that sessions are evenly distributed and that changes to the node set have minimal disruption.

#### Triaging the Question

The question asks how to implement a consistent-hashing ring for sharding sessions among cache nodes. The interviewer is likely probing your understanding of hashing techniques, data structure design, load balancing principles, and the practical considerations of managing node additions or removals in a live system.

#### Assumptions
We are using a ring-based approach where keys (session IDs) are hashed to determine their placement on the ring.
The goal is to distribute sessions as evenly as possible among cache nodes while minimizing disruption when adding or removing nodes.

#### First-Principles Solution
1. Ring Setup
Key Space: Define a fixed key space, typically from 0 to 2^32-1 or any other suitable range.
Hash Function: Choose a consistent hash function (e.g., MurmurHash) that maps keys to positions on the ring.
2. Node Insertion and Removal

Inserting a Node:

When a new node is added, it gets assigned its own segment of the ring.
The segments are chosen such that they cause minimal disruption to existing keys. For instance, if we add a new node N, we would need to calculate which segments in the existing key space will be transferred to this new node.

Removing a Node:

Removing a node can also disrupt the ring but similarly, by carefully managing the transfer of keys, we can minimize impact.
For example, if a node N is removed, its segment on the ring needs to be reassigned to existing nodes. This reassignment should be done in such a way that minimizes disruption.
3. Key Distribution
Each session key (k) gets hashed using the consistent hash function.
The hashed value determines which node k is mapped to on the ring.
When adding or removing nodes, we need to recalculate these mappings without causing significant disruptions.
Step-by-Step Solution
1. Initialization
Define a circular key space of size 2^32. For example:
PLAINTEXT
Key Space: [0, 4294967295]
Choose a consistent hash function like MurmurHash to map session keys to points on the ring.
2. Node Initialization
Each node gets assigned a unique identifier (ID).
Nodes are then mapped to positions on the key space. For instance, if there are N nodes, we can calculate their positions as:
PLAINTEXT
Node i: MurmurHash(node_id[i]) mod 2^32
3. Session Mapping
Map each session key k to a node using the consistent hash function.
PLAINTEXT
Node(session_k) = MurmurHash(k) mod N
This ensures that sessions are evenly distributed across nodes.
4. Adding a New Node
When adding a new node, we need to rehash all keys that fall within the new segment of the ring.
We can choose the position for the new node N such that it minimizes disruption. For example:
PLAINTEXT
Add a new node N at position MurmurHash(new_node_id) mod 2^32
5. Removing an Existing Node
When removing a node, we need to reassign its keys to existing nodes.
We can redistribute the segments of the removed node R among the remaining nodes:
PLAINTEXT
For each key k in Node(R), assign it to the closest available node on either side.

#### Closing Summary
and Common Mistakes/Follow-Ups

By using consistent hashing, we achieve a balanced distribution of sessions across cache nodes. The key benefits include minimal disruptions during node changes and efficient load balancing.

#### Common Mistakes
Ignoring Edge Cases: Forgetting to handle edge cases like hash collisions or ensuring that the reassignment logic is robust.
Inadequate Hash Function Choice: Using an inappropriate hash function that can lead to uneven distribution.
Insufficient Redundancy Management: Not adequately managing how keys are reassigned during node addition/removal.

#### Follow-Ups
How does this approach handle high traffic scenarios?
What if the key space is not fixed, and new nodes need to be dynamically added or removed?
How would you implement a failover mechanism in case of node failure?

**Comparison:**

```
Node ID	Key Range
0	[0, 5)
1	[5, 9)
...	...
N-1	[4294967290, 4294967295]
```

**Code:**

```python
import hashlib

def murmur_hash(key):
    return int.from_bytes(hashlib.murmurhash3_32(key.encode()), byteorder='big')

class ConsistentHashRing:
    def __init__(self, nodes):
        self.nodes = sorted(nodes)
        self.ring = {}
        for node in self.nodes:
            key = murmur_hash(node)
            self.ring[key] = node

    def add_node(self, new_node):
        key = murmur_hash(new_node)
        # Find the closest position on the ring
        index = min(range(len(self.nodes)), key=lambda i: abs((key - self.nodes[i]) % 4294967295))
        self.ring[key] = new_node
        self.nodes.insert(index, new_node)

    def remove_node(self, node_to_remove):
        # Find the closest positions on the ring before and after the removed node
        index_before = (self.nodes.index(node_to_remove) - 1) % len(self.nodes)
        index_after = (index_before + 1) % len(self.nodes)
        key_before = murmur_hash(self.nodes[index_before])
        key_after = murmur_hash(self.nodes[index_after])
        # Reassign the keys
        for key in self.ring.keys():
            if key >= key_before and key <= key_after:
                del self.ring[key]
        del self.nodes[index_before]

    def get_node_for_key(self, key):
        hash_key = murmur_hash(key)
        for node_key in sorted(self.ring.keys()):
            if node_key > hash_key or (hash_key < min(self.ring.keys()) and node_key == max(self.ring.keys())):
                return self.ring[node_key]
```

**Example input / output:**

```text
Input:
- Add a new node: "node4"

Output:
- Updated nodes: ["node1", "node2", "node3", "node4"]

Input:
- Remove an existing node: "node2"

Output:
- Updated nodes: ["node1", "node3"]
```

**Data:**

```json
{
  "nodes": ["node1", "node2", "node3"]
}
```

**Flow:**

1. Yes
2. No
3. Hash Key
4. Is key within a node's range?
5. Assign to Node
6. Reassign keys between nodes

