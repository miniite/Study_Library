---
type: "practice"
difficulty: "Deep"
---

# A per-user quota tracker for API-based LLM calls must stay accurate under concurrent writes across nodes; where does a naive per-node counter break and what structure gives correctness plus cheap reads at scale?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Concurrent Counters and CAS](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Concurrent_Counters_and_CAS.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the problem of maintaining an accurate per-user quota tracker for API-based LLM calls, especially under concurrent writes across multiple nodes, we need to address a key limitation of a naive per-node counter and explore a more robust solution.

TL;DR:

A naive per-node counter breaks down when multiple nodes attempt to increment the same user's quota simultaneously. A solution using a distributed data structure like a distributed counter or an atomic increment operation can provide both correctness and efficient reads at scale.

#### Triaging the Question

The question is essentially asking how we can ensure that our quota tracking system remains accurate even when multiple nodes are updating the same user’s quota concurrently. The core issue with naive per-node counters is that they may lead to race conditions where concurrent writes result in incorrect values.

#### Assumptions
and Key Concepts:
We need a distributed solution since updates will occur across multiple nodes.
Reads must be cheap and efficient, even at scale.
The system should maintain accuracy under concurrent writes.

#### Solution
Distributed Counter or Atomic Increment

To address the concurrency issues, we can use a distributed counter or an atomic increment operation. This approach ensures that all operations are performed atomically, preventing race conditions and maintaining data consistency.

Step-by-Step Reasoning from First Principles:

Identify the Problem with Per-Node Counters:

A naive per-node counter will have one counter for each node.
If multiple nodes attempt to increment the same user's quota simultaneously, a race condition can occur, leading to incorrect values.

Distributed Counter or Atomic Increment Operation:

Use a distributed counter that is shared across all nodes.
Each update operation should be atomic and performed on a single authoritative source (e.g., a database or a centralized service).

#### Implementation
Details:

Store the user's quota in a database or key-value store like Redis, which supports atomic operations.
Use a consistent hashing algorithm to distribute users across nodes for read operations but ensure that write operations are coordinated.
Detailed Explanation:

Distributed Counter Example (Redis):

RedisDB
Node2
Node1
UserA
RedisDB
Node2
Node1
UserA
Call API to update quota
Increment user's quota atomically
Return updated quota (if successful)

Atomic Increment Operation Example:

PYTHON
# Pseudocode for atomic increment in a distributed system using Redis
def update_quota(user_id, delta):
    current_value = redis_client.get(f'quota:{user_id}')
    if not current_value:
        new_value = 0 + delta
    else:
        new_value = int(current_value) + delta

    # Perform atomic operation to set the new value
    success = redis_client.setnx(f'quota:{user_id}', new_value)
    return success, new_value

#### Closing Summary
and Common Mistakes/Follow-Ups

By using a distributed counter or an atomic increment operation, we can ensure that the per-user quota tracker remains accurate even under concurrent writes across nodes. However, there are potential pitfalls:

Race Conditions: While atomic operations prevent race conditions, they must be carefully designed to handle edge cases.
Latency: Centralized solutions might introduce additional latency due to network round trips, so optimizations like sharding or caching can help.

#### Common Mistakes
Ignoring the need for atomicity in concurrent updates.
Assuming that a naive per-node counter will suffice without considering race conditions.
Overlooking the importance of centralized coordination in distributed systems.

**Code:**

```python
# Pseudocode for atomic increment operation using Redis
def update_quota(user_id, delta):
    current_value = redis_client.get(f'quota:{user_id}')
    if not current_value:
        new_value = 0 + delta
    else:
        new_value = int(current_value) + delta

    # Perform atomic operation to set the new value
    success = redis_client.setnx(f'quota:{user_id}', new_value)
    return success, new_value
```

**Data:**

```json
{
  "user_id": "12345",
  "quota": 10
}
```

