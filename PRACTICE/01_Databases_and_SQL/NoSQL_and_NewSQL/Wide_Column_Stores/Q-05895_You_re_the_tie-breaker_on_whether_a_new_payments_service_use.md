---
type: "practice"
difficulty: "Expert"
---

# You're the tie-breaker on whether a new payments service uses Cassandra LWT or DynamoDB TransactWriteItems for its critical invariants. Walk through the head-to-head on correctness guarantees, cost, latency, and multi-region behavior, and make the call.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Wide Column Design and Migration](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/DynamoDB_Modeling_and_Capacity/Wide_Column_Design_and_Migration.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

As a backend architect tasked with choosing between Cassandra LWT (Last Write Wins) and DynamoDB TransactWriteItems for critical invariants in a new payments service, we need to evaluate several factors including correctness guarantees, cost, latency, and multi-region behavior.

Firstly, let's clarify the key terms:

Cassandra LWT: Provides eventual consistency with Last Write Wins semantics.
DynamoDB TransactWriteItems: Offers strongly consistent writes within a single partition or global region using transactional coordination capabilities.

#### Triage
What Are We Comparing?

The core of this question is to decide between two NoSQL databases that provide different levels of data consistency and transactional guarantees. The key aspects we need to consider are:

Correctness Guarantees: How do the systems ensure that critical invariants are maintained, especially in a distributed environment.
Cost: What are the operational costs associated with each system.
Latency: How much time does it take for writes and reads to complete.
Multi-Region Behavior: How well each database handles data replication across multiple regions.

#### First Principles Solution

Let's break down each factor:

Correctness Guarantees

Cassandra LWT:

#### Cons
istency Model: Cassandra uses a distributed model with eventual consistency, meaning writes may not be immediately visible to all nodes.
Invariants Maintenance: To maintain critical invariants, you need to ensure that your application logic handles the eventual consistency model correctly. This can be complex and error-prone.

DynamoDB TransactWriteItems:

#### Cons
istency Model: DynamoDB provides strongly consistent writes within a single partition or global region.
Invariants Maintenance: With strong consistency, DynamoDB ensures that transactions are committed atomically, which simplifies maintaining critical invariants. This makes the system more predictable and easier to reason about.
Cost

Cassandra LWT:

Operational Costs: Cassandra is open-source and free to use. However, managing a highly available and performant setup requires significant expertise.
Scalability Costs: Scaling Cassandra involves careful tuning of partitioning and replication factors, which can be resource-intensive.

DynamoDB TransactWriteItems:

Operational Costs: AWS offers managed services, reducing the operational burden. However, DynamoDB is a paid service with non-trivial costs depending on throughput capacity.
Scalability Costs: AWS manages the underlying infrastructure, making it easier to scale out as needed.
Latency

Cassandra LWT:

Write Latency: Writes are generally faster because they can be sent to multiple nodes in parallel. However, eventual consistency means that reads may return stale data.
Read Latency: Reads can be slower due to the need for a quorum of nodes to agree on the latest value.

DynamoDB TransactWriteItems:

Write Latency: Writes are strongly consistent, meaning they are committed before returning. This ensures low latency writes but may require coordination between partitions.
Read Latency: Reads are also strongly consistent and can be very fast within a partition or globally if using global tables.
Multi-Region Behavior

Cassandra LWT:

Multi-Region Replication: Cassandra supports multi-region replication, but it requires careful configuration to ensure data consistency across regions.

#### Cons
istency Across Regions: Ensuring strong consistency across multiple regions is more complex and may require additional logic in the application.

DynamoDB TransactWriteItems:

Multi-Region Replication: DynamoDB Global Tables automatically handle multi-region replication, ensuring that writes are globally consistent.

#### Cons
istency Across Regions: Using global tables simplifies the maintenance of strong consistency across regions.

#### Closing Summary

Based on our analysis:

Correctness Guarantees: DynamoDB TransactWriteItems offers stronger guarantees and simplifies maintaining critical invariants. Cassandra LWT requires more application-level logic to handle eventual consistency.
Cost: While both systems have operational costs, DynamoDB's managed service model can be simpler and potentially less expensive for consistent workloads.
Latency: Both systems offer low latency writes, but DynamoDB provides stronger guarantees which can simplify the overall system design.
Multi-Region Behavior: DynamoDB Global Tables make multi-region operations easier and more reliable compared to Cassandra’s manual configuration.

In conclusion, if we prioritize correctness and ease of maintenance, DynamoDB TransactWriteItems is the better choice for a critical invariants-based payments service. However, if you have existing expertise with Cassandra and can manage eventual consistency effectively, it remains a viable option but requires careful application design.

#### Common Mistakes/Follow-Ups
Mistake: Overlooking the complexity of managing eventual consistency in Cassandra.

#### Follow-Up
Discussing edge cases where strong consistency is crucial (e.g., financial transactions) and how DynamoDB handles them more straightforwardly.

**Comparison:**

| Consistency Model | Cassandra LWT | DynamoDB TransactWriteItems |
|-------------------|---------------|-----------------------------|
| Write Latency     | Eventual      | Strong                      |
| Read Latency      | High          | Low                         |
| Maintenance       | High          | Low                         |
| Global Replication| Manual        | Automated                   |

**Code:**

```
MERMAID
Copy
graph LR;
    A[Payments Service] --> B[Cassandra LWT];
    A --> C[DynamoDB TransactWriteItems];
    B --> D[Eventual Consistency];
    C --> E[Strong Consistency];
    B --> F[Maintenance Complexity];
    C --> G[Operational Simplicity];
    D --> H[Complex Application Logic];
    E --> I[Simplified Design];
    F --> J[Higher Operational Costs];
    G --> K[Lower Operational Costs];
    A --> L[Global Replication];
    C --> M[Automated Global Tables];
```

**Example input / output:**

```text
Input:
- For Cassandra LWT: Write a payment transaction to two nodes in the same region.
- For DynamoDB TransactWriteItems: Perform a transacted write operation across multiple tables.

Output:
- Cassandra LWT: Writes are eventually consistent; transactions may need to be retried due to eventual consistency.
- DynamoDB TransactWriteItems: Writes are strongly consistent and can be confirmed immediately after the transaction completes.
```

**Data:**

```json
{
  "service": {
    "name": "Payments Service",
    "storage_options": [
      {"name": "Cassandra LWT", "consistency": "Eventual", "maintenance_complexity": "High"},
      {"name": "DynamoDB TransactWriteItems", "consistency": "Strong", "maintenance_complexity": "Low"}
    ],
    "regions": {
      "local": {
        "storage_options": [
          {"name": "Cassandra LWT", "cost": 500, "latency_ms": 100},
          {"name": "DynamoDB TransactWriteItems", "cost": 300, "latency_ms": 80}
        ]
      },
      "global": {
        "storage_options": [
          {"name": "Cassandra LWT", "cost": 700, "latency_ms": 200},
          {"name": "DynamoDB TransactWriteItems", "cost": 500, "latency_ms": 150}
        ]
      }
    }
  }
}
```

**Flow:**

1. Write Transaction
2. LWT Cassandra Write
3. Eventual Consistency
4. DynamoDB TransactWriteItems
5. Strong Consistency
6. Maintain Invariants
7. Cassandra Logic
8. Application Complexity
9. DynamoDB Global Tables
10. Simplified Design

