---
type: "practice"
difficulty: "Expert"
---

# A vendor claims their NewSQL database gives 'serializable at NoSQL scale.' As the architect signing the contract, what questions expose whether that's true or marketing?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ From Serializable to Sagas](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/From_Serializable_to_Sagas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In assessing a vendor’s claim of achieving "serializable at NoSQL scale" in their NewSQL database, as an Architect, I would focus on understanding how they achieve serializability and whether this is truly scalable to NoSQL volumes without compromising performance or data integrity. Serializability ensures that transactions occur in such a way that they cannot be interleaved with others in any manner that changes the outcome of running them sequentially. Here are key questions to expose the truth behind their claim:

Transaction Isolation Levels: NewSQL databases often offer multiple isolation levels like Read Uncommitted (RU), Read Committed (RC), Repeatable Read (RR), and Serializable. I would ask for a detailed explanation of how these levels map in their database, and particularly focus on how they achieve serializable transactions at scale.

Concurrency Control Mechanisms: Serializability typically relies on sophisticated concurrency control mechanisms such as locking or versioning. Questions should delve into the specific implementation, including whether they use row-level locks, multi-version concurrency control (MVCC), or other methods to ensure serializable transactions without causing bottlenecks at high concurrency.

Sharding and Distribution: NoSQL databases excel in horizontal scaling through sharding. I would explore how their system handles distributed transactions across shards. Would it require a distributed two-phase commit, which can significantly impact performance? Or do they use some form of optimistic or pessimistic concurrency control?

Latency and Throughput at Scale: Real-world performance metrics are crucial. Requesting benchmarks on common NoSQL workloads (like reads and writes) under various transaction isolation levels would provide insight into whether their database maintains serializable transactions efficiently.

Failure Modes and Recovery: Understanding how the system behaves during failures, especially in distributed environments, is critical. How do they ensure consistency and durability when nodes fail or network partitions occur? Do they have robust mechanisms for detecting and resolving conflicts?

Query Performance and Optimization: Even with serializable transactions, query performance can degrade if not optimized properly. I would ask how they handle complex queries at scale and whether there are any trade-offs in query execution time versus transaction isolation.

#### Implementation
Details and Code Review: For a more detailed understanding, reviewing the codebase or having an internal review session focusing on critical sections of their database could reveal implementation details that might not be publicly documented.

**Comparison:**

```
Failure Mode	Detection Time	Recovery Time
Node Failures	2ms	10s
Network Partitions	500ms	N/A (no recovery)
Query Complexity	Execution Time
Simple	10ms
Medium	50ms
```

**Code:**

```
MERMAID
Copy
graph TD
    A[Transaction Isolation Levels] --> B(Read Uncommitted)
    A --> C(Read Committed)
    A --> D(Repeatable Read)
    A --> E(Serializable)
    F[Concurrency Control Mechanisms] --> G(row-level locks)
    F --> H(MVCC)
    I[Sharding and Distribution] --> J(distributed two-phase commit)
    I --> K(optimistic concurrency control)
    L[Benchmarks] --> M(reads)
    L --> N(writes)
    O[Fault Tolerance] --> P(failure detection)
    O --> Q(conflict resolution)
    R[Query Optimization] --> S(complex queries)
    T[Implementation Details] --> U(code review)
```

**Example input / output:**

```text
| Workload | Reads (Simple) | Writes (Inserts) |
|----------|----------------|-----------------|
| Volume   | 10,000         | 2,000           |
| Time     | 5ms            | 3ms             |

Transaction Isolation Levels:
- RU: 98% read uncommitted transactions
- RC: 95% read committed transactions
- RR: 90% repeatable read transactions
- Serializable: 100% serializable transactions

Concurrency Control Mechanism:
- MVCC for most operations, with row-level locks for write-heavy workloads.
```

**Data:**

```json
{
  "workloads": {
    "reads": [
      { "type": "simple", "volume": 10000 },
      { "type": "complex", "volume": 5000 }
    ],
    "writes": [
      { "type": "inserts", "volume": 2000 },
      { "type": "updates", "volume": 3000 }
    ]
  },
  "failureModes": {
    "nodeFailures": true,
    "networkPartitions": false
  },
  "queryComplexity": [
    { "complexityLevel": "simple", "executionTime": 10ms },
    { "complexityLevel": "medium", "executionTime": 50ms }
  ]
}
```

**Flow:**

1. RC
2. RR
3. Serializable
4. Yes
5. No
6. RC
7. RR
8. Serializable
9. Transaction Starts
10. Isolation Level?
11. Read Committed
12. Repeatable Read
13. E
14. Distributed Two-Phase Commit
15. Conflict Detected?
16. Manual Intervention
17. Transaction Commits
18. Node Failure
19. Isolation Level?
20. Read Committed Recovery
21. Repeatable Read Recovery
22. Distributed Rollback

