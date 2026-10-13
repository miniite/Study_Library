---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd design a 'consistency SLA tier' menu (eventual / read-committed / serializable) that product teams choose from, with cost and latency clearly attached to each tier.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Isolation with Read Replicas](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/Isolation_with_Read_Replicas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a "consistency SLA tier" menu for database isolation levels in an application, we need to carefully balance consistency requirements with performance and cost considerations. The key tiers are eventual consistency (eventual), read-committed (RC), and serializable (Serializable). Each tier has distinct trade-offs that product teams can choose from based on their specific needs.

Firstly, let's define each isolation level:

Eventual Consistency: This is the least restrictive option where reads do not necessarily see the results of recent writes until those results have been propagated to all relevant nodes in the system.
Read-Committed (RC): Reads cannot read uncommitted data but can still see "dirty" rows from previous transactions that were later rolled back.
Serializable: This is the most restrictive level, ensuring that each transaction appears as if it were executed serially and independently of other transactions.

The design involves several steps:

Identify the Tiers:

Eventual: Lowest cost, highest latency but ensures no write collisions.
RC: Moderate cost, moderate latency with some risk of reading uncommitted data.
Serializable: Highest cost, lowest latency at the expense of performance and complexity.

Cost and Latency Metrics:

Cost is primarily based on infrastructure (storage, compute) and network overhead.
Latency includes query execution time, network delay between nodes, and potential retries due to conflicts.

Design Decisions:

Eventual Consistency: Use a distributed system with eventual consistency guarantees like Cassandra or DynamoDB. This tier is ideal for scenarios where the business logic can tolerate occasional stale reads.
RC: Implement through optimistic concurrency control (OCC) in a traditional RDBMS. Use MVCC to manage conflicting writes and reduce lock contention, but still allow some risk of reading uncommitted data.
Serializable: Optimize with pessimistic locking or fine-grained row-level locks. This tier is suitable for high-concurrency transactional workloads where data integrity is critical.

Cost Analysis:

Eventual Consistency: Lower cost due to reduced need for synchronization and retries but requires careful design to ensure eventual consistency.
RC: Moderate cost with the added complexity of OCC, potentially higher infrastructure costs due to more nodes involved in conflict resolution.
Serializable: Highest cost due to extensive locking mechanisms and potential performance bottlenecks under high concurrency.

Latency Considerations:

Eventual Consistency: Higher latency as data propagates across nodes but generally lower than RC/Serializable.
RC: Moderate latency with occasional spikes during conflict resolution.
Serializable: Lowest latency due to fewer conflicts but can suffer from performance bottlenecks under high concurrency.

Monitoring and Optimization:

Implement monitoring tools to track consistency violations, query performance, and resource usage.
Use database tuning techniques like indexing, partitioning, and sharding to optimize performance while maintaining consistency levels.

**Comparison:**

```
Isolation Level	Description	Max Cost	Min Latency (ms)
Eventual	Lowest cost, highest latency	100	500
RC	Moderate cost, moderate latency	500	200
Serializable	Highest cost, lowest latency	1000	100
```

**Code:**

```python
# Example function to simulate transaction isolation level selection based on cost and latency
def choose_isolation_level(cost_budget, latency_requirement):
    if cost_budget <= 100:
        return "Eventual"
    elif cost_budget <= 500:
        if latency_requirement < 200:  # ms
            return "Serializable"
        else:
            return "RC"
    else:
        return "Serializable"
```

**Example input / output:**

```text
# Example input -> output for transaction isolation level selection
inputs = [
    (50, 300),   # Low cost, low latency requirement -> Eventual
    (400, 250),  # Moderate cost, moderate latency requirement -> RC
    (800, 150)   # High cost, high latency requirement -> Serializable
]
outputs = [
    "Eventual",
    "RC",
    "Serializable"
]

# Dry-run of the function
for input, output in zip(inputs, outputs):
    print(f"Input: {input} -> Output: {output}")
```

**Data:**

```json
{
  "cost_tiers": {
    "eventual": {"description": "Lowest cost, highest latency", "max_cost": 100},
    "rc": {"description": "Moderate cost, moderate latency", "max_cost": 500},
    "serializable": {"description": "Highest cost, lowest latency", "max_cost": 1000}
  },
  "latency_tiers": {
    "eventual": {"min_latency": 500},  # ms
    "rc": {"min_latency": 200},
    "serializable": {"min_latency": 100}
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Transaction Starts
6. Is Cost Budget <= $100?
7. Use Eventual Consistency
8. Is Latency Requirement <$200?
9. Use Serializable Isolation
10. Use Read-Committed (RC)

