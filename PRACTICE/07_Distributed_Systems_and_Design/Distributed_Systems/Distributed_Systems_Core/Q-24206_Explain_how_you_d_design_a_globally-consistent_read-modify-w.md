---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd design a globally-consistent read-modify-write on a counter (inventory, wallet balance) in a distributed database that must never lose an update, comparing an atomic conditional update, a distributed lock, and a CRDT counter on correctness and availability.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Distributed Counters and Consistency](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Distributed_Counters_and_Consistency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a globally-consistent read-modify-write operation on a counter in a distributed database that must never lose an update, we need to consider three approaches: atomic conditional updates, distributed locks, and conflict-free replicated data types (CRDTs). Each has its own trade-offs in terms of correctness and availability. Let's explore each method in detail.

First, I will provide a brief overview of the problem and then delve into each approach, comparing their strengths and weaknesses from first principles.

**Comparison:**

```
Approach	Correctness	Availability	Consistency Model
Atomic Conditional Update	High	Low	Sequential consistency
Distributed Lock	Medium	High	Eventual consistency (with retries)
CRDT Counter	High	Low	Eventual consistency
```

**Code:**

```
MERMAID
Copy
graph TB
    A[Read-Modify-Write] -->|Atomic Conditional Update| B1[Consistent but less Availability]
    A -->|Distributed Lock| C1[High Availability but Potential Deadlocks]
    A -->|CRDT Counter| D1[High Consistency but Lower Availability in some cases]
```

**Example input / output:**

```

Input:

Read the current value of the counter.
Modify it by subtracting a certain amount (e.g., subtract(25)).
Write back to the database.

Output for each approach:

Atomic Conditional Update:

Read the current value from the database.
Check if the value has been updated since reading; if so, retry until consistent.
Modify and write the counter in a single transactional operation.

Distributed Lock:

Acquire a lock on the counter before performing any read-modify-write operations.
Perform the modification under the lock's protection.
Release the lock after the operation is complete.

CRDT Counter:

Use a CRDT structure that ensures eventual consistency but maintains local atomicity during operations.
Apply an update to the local copy and replicate it across nodes as they become available.
```

**Data:**

```json
{
  "counter": {
    "id": "inventory_balance",
    "value": 100,
    "last_updated_by": ""
  }
}
```

**Flow:**

1. Yes
2. No
3. Read Counter
4. Is Value Stale?
5. Aquire Lock
6. Modify Counter and WriteBack
7. Release Lock
8. Modify Counter and WriteBack
9. CRDT Operation
10. Apply to Local Copy, Replicate

