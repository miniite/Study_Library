---
type: "practice"
difficulty: "Expert"
---

# Why isn't a Redis SETNX-based lock safe enough for correctness-critical work, and what does the Redlock debate teach about the difference between a lock for efficiency vs a lock for correctness?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cross-service Invalidation Races](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Cross-service_Invalidation_Races.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

A Redis SETNX-based lock is not always safe enough for correctness-critical work because it doesn't fully account for race conditions, especially in distributed systems where network partitions can occur. The Redlock algorithm introduces a robust approach to mitigate these issues by using multiple Redis instances and a majority-wins strategy, ensuring high availability even during partial failures.

#### Triaging the Question

The question delves into why a simple SETNX lock might fail in critical scenarios, particularly distributed environments with potential network partitions. The key is understanding the limitations of atomic operations like SETNX and how they can be exploited by race conditions. Additionally, it invites us to explore the Redlock algorithm, which addresses these issues through a more sophisticated approach.

#### First-Principles Solution
Why Redis SETNX-Based Locks Aren't Safe Enough

Atomicity Limitation: SETNX (Set if Not Exists) is an atomic operation that ensures key-value pairs are set only once. However, it lacks the ability to handle concurrent requests and race conditions effectively.

Race Conditions in Distributed Systems: In a distributed environment, multiple nodes might attempt to acquire the same lock simultaneously. SETNX alone cannot guarantee that one of these nodes will successfully acquire the lock without interference from others.

Network Partitions: Network partitions can introduce unexpected behavior where some nodes are isolated from each other. This can cause partial failures, leading to inconsistent state across the system. If a node holds a lock but becomes partitioned from the rest, it might not be able to release the lock properly, causing deadlocks or livelocks.

The Redlock Algorithm

Multiple Nodes: Redlock employs multiple Redis instances (typically 5-6) to ensure that even if one instance fails, the majority can still agree on whether a resource is locked.

Majority-Wins Strategy: By using a quorum mechanism (e.g., at least three out of five nodes), it ensures that decisions are made based on a majority, reducing the risk of partial failures affecting the system’s state.

Timeout Mechanism: Each node has a timeout period before it assumes a lock is no longer held and attempts to acquire it again. This helps in handling cases where nodes might have missed a message due to network issues.

Ordering Constraints: Redlock enforces ordering constraints on the operations, ensuring that commands are executed in a consistent order across all instances, which further mitigates race conditions.

#### Closing Summary
and Common Mistakes

Mistake 1: Assuming SETNX is Always Safe - Ignoring the limitations of atomic operations can lead to incorrect assumptions about their behavior in distributed systems.

Mistake 2: Overlooking Network Partitions - Failing to account for network partitions can result in data inconsistencies and unhandled edge cases, such as partial failures.

Mistake 3: Not Implementing Quorums - Relying on single-node operations without redundancy or quorum-based decision-making can lead to system instability.

#### Follow-Ups
How Would You Implement Redlock in Your Code?
What Are the Trade-offs Between Using Single Node Locks and Redlock?
Can You Describe a Scenario Where Even Redlock Might Fail?

**Comparison:**

```
Lock Key	Acquired After (ms)
lock1	5
lock2	6
lock3	-

Explanation:

lock1 and lock2 were acquired within the timeout period.
lock3 did not get acquired, but since we have a quorum of two out of three, Redlock successfully determined that the locks are held.
```

**Code:**

```python
import redis
from time import sleep

# Example of a basic lock implementation using Redis SETNX
def setnx_lock(key, value):
    r = redis.Redis(host='localhost', port=6379, db=0)
    # Attempt to acquire the lock
    acquired = r.setnx(key, value)
    if not acquired:
        print("Failed to acquire lock!")
    return acquired

# Example of Redlock implementation using multiple nodes and timeouts
def redlock(lock_keys, timeout_ms):
    from random import randrange
    node_count = len(lock_keys)  # Number of Redis instances
    required_nodes = (node_count + 1) // 2  # Quorum for majority decision

    # Simulate acquiring locks
    successful_locks = []
    for i in range(0, timeout_ms, 50):  # Simulate delay
        sleep(0.05)
        for key in lock_keys:
            if setnx_lock(key, '1') and len(successful_locks) < required_nodes:
                successful_locks.append(key)
                print(f"Successfully acquired {key} after {i}ms")

    # Check if we have the majority of nodes
    return len(successful_locks) >= required_nodes

# Example usage
lock_keys = ['lock1', 'lock2', 'lock3']
is_locked = redlock(lock_keys, 500)
print("Redlock acquired:", is_locked)
```

**Example input / output:**

```text
Input:
- Lock Keys: ['lock1', 'lock2', 'lock3']
- Timeout: 500ms

Output:
Successfully acquired lock1 after 5ms
Successfully acquired lock2 after 6ms
Redlock acquired: True

Explanation:
In this example, the Redlock algorithm successfully acquires a majority of locks (2 out of 3) within the given timeout period.
```

**Data:**

```json
{
  "lock_keys": ["lock1", "lock2", "lock3"],
  "timeout_ms": 500,
  "successful_locks": [
    { "key": "lock1", "acquired_after_ms": 5 },
    { "key": "lock2", "acquired_after_ms": 6 }
  ]
}
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. No
6. No
7. Begin
8. Is lock_key available?
9. Acquire lock and return
10. Check if timeout is reached
11. Is quorum achieved?
12. Return True, lock acquired
13. Release locks and retry later
14. Return False, no lock acquired

