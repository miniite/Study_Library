---
type: "practice"
difficulty: "Deep"
---

# You're running an A/B test across a distributed serving fleet and must guarantee a given user deterministically and stickily gets the same model variant even as replicas scale and rollouts happen. Explain the hash-bucketing design and why non-sticky assignment silently invalidates your experiment statistics.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Mixed Version Fleets](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure deterministic and sticky assignment of users to model variants in a distributed A/B testing environment, we need to use a consistent hashing mechanism known as hash-bucketing. The core idea is that each user is mapped to a specific "bucket" using a hash function, ensuring they always receive the same variant throughout their session. This design guarantees reproducibility and accuracy of experiment statistics by preventing silent invalidation.

#### Triaging the Question

The question aims to understand how distributed systems handle sticky sessions in A/B testing scenarios. It specifically probes understanding of:

#### Cons
istent Hashing Mechanism: How user IDs are mapped to model variants.
Determinism and Stickiness: Ensuring the same user always gets the same variant despite scaling or rolling updates.
Experimental Validity: Why non-sticky assignment can lead to biased results.

#### Assumptions
We are dealing with a high-scale, distributed serving fleet where multiple replicas exist.
Each replica runs an updated version of the model based on ongoing A/B tests.
The system must handle user sessions consistently without disruptions.

#### First Principles Solution
Step 1: Understanding User ID Mapping

Each user is assigned a unique identifier (e.g., session ID, IP address). This identifier needs to be mapped to one of several "buckets" corresponding to different model variants. We use consistent hashing to achieve this mapping deterministically.

Step 2: Consistent Hashing Algorithm

#### Cons
istent hashing works by distributing the hash values of user IDs across a ring of nodes. Each node holds a contiguous segment of the hash space, and user mappings are routed to the first node in the segment containing their hash value.

Hash Function: Use a cryptographic hash function (e.g., SHA-256) to map each user ID to a unique 256-bit integer.
Ring Construction: Create a circular ring where nodes are placed at equal intervals.
Bucket Assignment: For a given user, find the first node in the ring that contains their hash value.
Step 3: Handling Node Additions and Removals

When new replicas (nodes) are added or existing ones are removed:

If a replica is added, it takes ownership of part of an adjacent node's segment.
If a replica is removed, its segment is reassigned to the neighboring nodes. This ensures minimal disruption in user mappings.
Step 4: Ensuring Stickiness

By using consistent hashing, we ensure that as long as the hash function and ring configuration remain unchanged, the same user ID will always map to the same bucket (model variant). This stickiness is critical for maintaining experiment validity because it prevents changes in the fleet from reassigning users randomly.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In a distributed system, using consistent hashing with hash-bucketing ensures that each user gets the same model variant deterministically. This design is crucial for accurate A/B testing statistics and avoiding silent invalidation issues that can arise from non-sticky assignments. Common pitfalls include:

Incorrect Hash Function: Using poor or non-cryptographic hash functions.
Inadequate Ring Configuration: Failing to properly manage node additions and removals, leading to inconsistent bucket distributions.

#### Follow-up
questions could include:

How does this impact system scalability and performance?
What are the trade-offs between consistent hashing and other sticky session methods?

**Comparison:**

```
User ID	Hashed Value (Hex)	Replica
12345	aabbcc...	node1
67890	deefgh...	node2
54321	ijklmn...	node3
```

**Code:**

```python
import hashlib

def hash_user_id(user_id):
    return int(hashlib.sha256(user_id.encode()).hexdigest(), 16)

def find_bucket(user_id, replicas):
    hashed_id = hash_user_id(user_id)
    ring = sorted(replicas) + [replicas[0]]  # Create a circular ring
    for i in range(len(ring) - 1):
        if (hashed_id >= hash_user_id(ring[i]) and
            hashed_id < hash_user_id(ring[i+1])):
            return replicas[i]
    return replicas[-1]  # Fallback to the last replica

replicas = ['node1', 'node2', 'node3']
user_id = '12345'
model_variant = find_bucket(user_id, replicas)
print(f"User ID: {user_id} -> Model Variant: {model_variant}")
```

**Example input / output:**

```text
| User ID | Replica |
|---------|---------|
| 12345   | node1   |
| 67890   | node2   |
| 54321   | node3   |

### Example Dry-Run
Given the replicas `["node1", "node2", "node3"]` and user IDs `["12345", "67890", "54321"]`, we map them as follows:
- User ID 12345 -> node1
- User ID 67890 -> node2
- User ID 54321 -> node3
```

**Data:**

```json
{
  "replicas": ["node1", "node2", "node3"],
  "user_ids": ["12345", "67890", "54321"]
}
```

**Flow:**

1. Yes
2. No
3. User ID
4. Is user ID in hash ring?
5. Stickily assigned to replica
6. Rehash or fallback
7. Model Variant
8. New replica handling

