---
type: "practice"
difficulty: "Deep"
---

# Your gateway uses consistent hashing to route sticky sessions to upstream nodes, and adding one node reshuffled 30% of keys causing cache misses; how do you bound reshuffling and why does naive modulo fail here?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Response Transformation and Upstream Errors](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Response_Transformation_and_Upstream_Errors/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

#### Cons
istent hashing is a technique used in API gateways to efficiently manage key-to-node mappings, ensuring that adding or removing nodes minimally impacts existing sessions. However, it can lead to significant reshuffling of keys when a node is added, as seen in the scenario where 30% of keys were affected by adding one node. In this response, we will delve into why naive modulo-based hashing fails and discuss strategies to bound reshuffling.

#### Triage

The question revolves around understanding consistent hashing and how it differs from simple modulo-based hashing. Specifically, the goal is to explain why a single node addition can cause such a large percentage of keys to be remapped and to propose methods to minimize this impact.

#### Assumptions
We are dealing with an API gateway that uses consistent hashing for sticky sessions.
The current implementation is naive modulo-based hashing.
The gateway manages a significant number of nodes and requests.

#### First-Principles Solution
Why Naive Modulo Fails Here

Naive modulo hashing, which maps keys to node indices using ( \text{key} % N ), where (N) is the number of nodes, has several limitations:

High Reshuffling on Node Addition: When a new node is added, every key that hashes to an index greater than or equal to the old index threshold will be remapped. This leads to a large percentage of keys needing to be reassigned.

Uniform Distribution Assumption: Naive modulo assumes uniform distribution but fails in scenarios where the hash function isn't uniformly distributed, leading to uneven load balancing and frequent reshuffling.

Bounding Reshuffling with Consistent Hashing

#### Cons
istent hashing achieves better performance by maintaining a circular ring of nodes and mapping keys onto this ring. Here’s how it works:

Node Ring: Nodes are placed on a virtual circle (ring), and each node has a specific range of key values.
Key Placement: Keys are also mapped to the same ring, ensuring they fall within one of these ranges.
Addition Logic:
When adding a new node, it is inserted into the ring at an appropriate position based on its hash value.
The keys associated with the old and new nodes that overlap will be remapped.

#### Example
Adding One Node

Suppose we have 4 nodes (A, B, C, D) in order around the ring. Let’s add a new node (E), which falls between (B) and (C). The key ranges for the nodes change as follows:

(A \rightarrow A, B, E, C)
(B \rightarrow B, E, C, D)

Keys that were originally mapped to (C) or above now map to different nodes. This explains why 30% of keys might need remapping.

Minimizing Reshuffling
Sparsely Placed Nodes: If nodes are placed more sparsely on the ring, fewer keys will be affected by each addition.
Ring Size: Using a larger ring size (more virtual nodes) can also help in minimizing reshuffling since it spreads out the key space.

#### Closing Summary
and Common Mistakes
Common Mistake 1: Assuming naive modulo hashing is sufficient for load balancing, without understanding its limitations.
Common Mistake 2: Failing to realize that consistent hashing requires a different approach when adding or removing nodes.
Key Takeaway: Consistent hashing provides better load distribution and minimizes reshuffling by leveraging the circular nature of the ring.
Common Follow-Ups

#### Trade-offs
Between Load Balancing and Reshuffling: Discuss how increasing the number of virtual nodes can improve load balancing but may also introduce more overhead.

#### Implementation
Details: Explain how consistent hashing is implemented in practice, including hash functions and key placement strategies.
Advanced Topics: Explore topics like distributed consensus algorithms that further enhance resilience and efficiency.

**Comparison:**

```
Node	Key Range (Naive Modulo)	Key Range (Consistent Hashing)
A	0 - 24	0 - 19, 45 - 60
B	25 - 49	20 - 39
C	50 - 74	40 - 44, 61 - 75
D	75 - 99	76 - 99
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Node Ring] -->|Key Placement| B[Consistent Hashing]
    B --> C{Add Node}
    C --> D[Re-map Keys]
```

**Example input / output:**

| Original | New |
|----------|-----|
| A        | A   |
| B        | B   |
| C        | E   |
| D        | C   |

**Data:**

```json
{
  "nodes": ["A", "B", "C", "D"],
  "new_node": "E",
  "key_ranges": {
    "A": [0, 25],
    "B": [25, 50],
    "C": [50, 75],
    "D": [75, 100],
    "E": [45, 60]
  }
}
```

**Flow:**

1. No
2. Yes
3. Node Addition
4. Does Node Fit?
5. Reshuffle Keys
6. Re-map Overlapping Keys

