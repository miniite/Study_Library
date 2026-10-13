---
type: "practice"
difficulty: "Deep"
---

# A model-serving router must map request keys to backend replicas so that scaling adds/removes minimal remapping; explain consistent hashing with virtual nodes and how you tune vnode count for even load.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Hashing and Collisions](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Hashing_and_Collisions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure efficient scaling of a model-serving system, we need to use consistent hashing with virtual nodes. This approach ensures that adding or removing replicas minimally affects existing mappings, leading to more balanced and robust load distribution. Let's break down the key concepts and how to tune the number of virtual nodes.

#### Triage

The core of this question is understanding consistent hashing with virtual nodes and determining an optimal number of virtual nodes for even load distribution. The interviewer is probing your knowledge of data structures, specifically hash tables and circular buffers, as well as your ability to balance performance and resource management in a distributed system.

#### First-Principles Solution

#### Cons
istent hashing works by placing keys into a ring structure where the space between key positions can be further divided using virtual nodes. Each replica is assigned multiple virtual nodes, and requests are routed based on the hash of the request key. This way, adding or removing replicas requires only a small percentage of remappings.

Tuning Virtual Nodes

To achieve even load distribution, we need to carefully choose the number of virtual nodes per backend replica. The number of virtual nodes should be a factor that balances between granularity and overhead:

Granularity: More virtual nodes provide finer-grained mapping, ensuring better load balancing.
Overhead: A larger number of virtual nodes increases memory usage and computational cost for hash calculations.
Detailed Explanation

Let’s consider a simple example with 4 backend replicas and 64 virtual nodes per replica:

Key Ring: Imagine the keys are placed in a circular ring, starting from 0 to 255 (or any other range that fits your key space).
Virtual Nodes Assignment: Each of the 4 replicas is assigned 16 virtual nodes. So, we have a total of (4 \times 16 = 64) virtual nodes.
Request Handling: When a request comes in, its key is hashed to find the corresponding position on the ring. The router then looks at the nearest virtual node clockwise and forwards the request to the replica that owns that virtual node.

#### Example
Calculation

Let’s assume we have the following keys: 10, 25, 30, ..., 246 (these are just placeholder values). Here’s how they map with 16 virtual nodes per replica:

Replica 1: Virtual Nodes 0-15 (Keys in range [0, 15])
Replica 2: Virtual Nodes 16-31 (Keys in range [16, 31])
Replica 3: Virtual Nodes 32-47 (Keys in range [32, 47])
Replica 4: Virtual Nodes 48-63 (Keys in range [48, 63])

If we add a new replica and keep the same number of virtual nodes:

The new replica will be assigned additional virtual nodes, say 16 more.
Most existing mappings remain unchanged as long as the new replica's virtual nodes do not overlap significantly with existing ones.

#### Closing Summary

By using consistent hashing with virtual nodes, we can efficiently manage backend replicas without frequent remappings. Tuning the number of virtual nodes involves balancing granularity and overhead to achieve even load distribution. In practice, you would experimentally determine the optimal number based on your key space size and performance requirements.

#### Common Mistakes/Follow-Ups
Not Understanding Overlap: Failing to recognize that adding or removing replicas can significantly affect existing mappings.
Ignoring Key Space Size: Not considering the size of the key space when choosing virtual nodes, which can lead to suboptimal load balancing.
Over-Engineering: Choosing an excessive number of virtual nodes without considering practical constraints like memory and computational costs.

To further validate this approach, you could implement a small-scale prototype or simulate different scenarios using provided sample data.

**Comparison:**

```
Key	Virtual Node Range	Replica Ownership
10-15	0-15	Replica 1
16-31	16-31	Replica 2
32-47	32-47	Replica 3
48-63	48-63	Replica 4
```

**Code:**

```python
# Python example for consistent hashing with virtual nodes
class ConsistentHashingRouter:
    def __init__(self, replicas, num_vnodes=16):
        self.replicas = replicas
        self.vnode_count = num_vnodes
        self.nodes = {}

    def add_replica(self, replica_id):
        # Add a new replica with virtual nodes
        for i in range(self.vnode_count):
            vkey = (replica_id * 100) + i  # Example vkey calculation
            self.nodes[vkey] = replica_id

    def remove_replica(self, replica_id):
        # Remove a replica and its virtual nodes
        for i in range(self.vnode_count):
            vkey = (replica_id * 100) + i
            if vkey in self.nodes:
                del self.nodes[vkey]

    def get_replica_for_key(self, key):
        ring_pos = hash(key) % 4096  # Example ring size
        closest_vnode = min(self.nodes.keys(), key=lambda x: abs(x - ring_pos))
        return self.nodes[closest_vnode]
```

**Example input / output:**

```text
Input:
- Add replica with id = 5

Output:
- Nodes after adding new replica:
{
  500: 5,
  501: 5,
  ...
  639: 5
}

Key "247" maps to Replica 5.
```

**Data:**

```json
{
  "replicas": [
    { "id": 1 },
    { "id": 2 },
    { "id": 3 },
    { "id": 4 }
  ],
  "virtual_nodes_per_replica": 16
}
```

**Flow:**

1. Yes
2. No
3. Hash Request Key
4. Is Key in Ring?
5. Find Nearest Virtual Node
6. Forward to CorrespondingBackend Replica
7. Log/Handle Out-of-Ring Case
8. Return Response

