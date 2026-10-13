---
type: "practice"
difficulty: "Deep"
---

# You maintain a distributed counter that must be accurate under concurrent increments across nodes; where does a naive shared counter break, and how do sharded counters or CRDTs restore correctness?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Concurrent Counters and CAS](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Concurrent_Counters_and_CAS.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are tasked with maintaining an accurate distributed counter that can handle concurrent increments from multiple nodes. A naive approach might use a shared counter stored in a single location (e.g., a database or memory). However, this breaks down under concurrency because it cannot guarantee atomicity and leads to race conditions. To restore correctness, sharded counters or Conflict-free Replicated Data Types (CRDTs) are employed.

#### Triaging the Question

The core of this question revolves around understanding why shared counters fail in a distributed environment with concurrent access and exploring alternative solutions that ensure consistency and accuracy. The interviewer is probing your knowledge of concurrency issues, distributed systems design, and data structures that can handle such scenarios.

Step-by-Step Solution from First Principles

Naive Shared Counter Failure:

A naive shared counter stored in a single location (e.g., a central database or memory) cannot ensure atomicity during concurrent operations.
Race conditions occur when multiple nodes attempt to increment the counter simultaneously, leading to incorrect results.

Sharded Counters:

Sharding distributes the counter across multiple shards (sub-counters).
Each shard can be independently incremented without interfering with other shards.
Periodically, the counts are aggregated or rebalanced to maintain overall accuracy and consistency.

Conflict-free Replicated Data Types (CRDTs):

CRDTs use a distributed approach that allows for eventual consistency by ensuring commutativity and idempotency.
Each node maintains its own copy of the counter, and operations are applied locally without conflict resolution.
Periodic reconciliation ensures all nodes converge to the same value.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, a naive shared counter is prone to race conditions in concurrent environments. Sharded counters and CRDTs offer robust solutions by distributing the workload and ensuring eventual consistency. However, sharding requires careful design for aggregation and rebalancing, while CRDTs handle conflicts locally but may require more complex reconciliation mechanisms.

#### Common mistakes
include:

Assuming a single shared counter can handle concurrency without issues.
Overlooking the need for periodic synchronization in sharded counters.
Ignoring conflict resolution strategies required by CRDTs to ensure consistency across nodes.

#### Follow-ups
might explore specific implementation details, trade-offs between different approaches, and real-world applications where these techniques are used effectively.

**Comparison:**

```
Approach	Pros	Cons
Shared Counter	Simple to implement	Race conditions, lack of concurrency support
Sharded Counters	Scalable and concurrent	Requires aggregation/rebalancing logic
CRDTs	Eventual consistency with local operations	More complex conflict resolution mechanisms
```

**Code:**

```python
# Example of a simple increment operation for a sharded counter (pseudo-code)
def increment_counter(shard_id):
    shard = shards[shard_id]
    new_count = shard.get_count() + 1
    shard.set_count(new_count)

# Example of reconciliation in CRDTs (pseudo-code)
def reconcile_counters():
    global_total = sum([shard.count for shard in shards])
    # Apply necessary operations to ensure all nodes converge to the same value
```

**Example input / output:**

```text
Input: Shard increments (0, +1), (1, +1)
Output:
Shard 0 count: 11
Shard 1 count: 21

Reconciliation step:
Global total: 31
```

**Data:**

```json
{
  "shards": [
    {"id": 0, "count": 10},
    {"id": 1, "count": 20}
  ]
}
```

**Flow:**

1. Node Increment
2. Shard Counter
3. Increment Local Shard
4. PeriodicAggregation/Rebalancing
5. Global Count Consistent
6. CRDT Operation
7. Local Operations
8. Commuteable and Idempotent
9. Periodic Reconciliation forGlobal Consistency

