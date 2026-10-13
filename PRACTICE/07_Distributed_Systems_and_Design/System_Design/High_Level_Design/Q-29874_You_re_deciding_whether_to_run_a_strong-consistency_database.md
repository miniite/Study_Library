---
type: "practice"
difficulty: "Deep"
---

# You're deciding whether to run a strong-consistency database everywhere for simplicity or mix in eventually-consistent stores for scale — a bar-raiser asks you to defend the operational complexity of a mixed model; do it.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-region Write Paths](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Cross-region_Write_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When considering a mixed model where both strong-consistency databases and eventually-consistent data stores are used, the primary concern is indeed the operational complexity. The decision to use a mix of these systems involves understanding their trade-offs and ensuring that the overall system can be managed effectively. In this context, we need to consider factors such as consistency requirements, performance needs, and cost optimization.

#### Triage
This question is probing your ability to balance simplicity with scalability in database design. Specifically, it aims to test whether you understand the operational complexities involved when integrating different types of databases and how these complexities can impact system maintenance and reliability.

Key Assumptions:

The application requires both high consistency for critical transactions and scalable read/write operations.
There are performance bottlenecks or cost constraints that cannot be addressed solely with strong-consistency databases.
The risk tolerance for eventual consistency is acceptable for non-critical data.

#### First-principles Solution
To support the mixed model, we need to carefully design the system architecture. Let's break this down step-by-step:

Identify Critical vs Non-Critical Data: Define which parts of your application require strong consistency and which can tolerate eventual consistency.

Partitioning Strategy: Use logical partitions or sharding techniques to separate critical data from non-critical data.

Data Consistency Models:

For critical data, use a strongly consistent database (e.g., Cassandra with serial isolation level).
For non-critical data, leverage eventually-consistent stores (e.g., AWS DynamoDB with eventual consistency settings).

Synchronization Mechanisms: Implement mechanisms to ensure that data is synchronized between the two types of databases when necessary.

Monitoring and Alerting: Set up comprehensive monitoring for both types of databases to detect and address issues promptly.

#### Closing Summary
A mixed model can offer a balance between simplicity and scalability, but it comes with significant operational complexity. It's crucial to have robust strategies in place to manage the different consistency models effectively.

#### Common Mistakes/Follow-Ups

Overlooking the need for careful partitioning and synchronization.
Failing to implement adequate monitoring and alerting.
Not considering the performance implications of mixed systems.

**Comparison:**

```
Database Type	Characteristics	Use Case
Strongly Consistent	High consistency, low latency	Critical transactions
Eventually Consistent	Low cost, high scalability	Aggregated reporting
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Identify Critical vs Non-Critical Data] --> B[Partition Data];
    B --> C[Choose Strongly Consistent DB for Critical Data];
    B --> D[Use Eventually Consistent Stores for Non-Critical Data];
    C --> E[Synchronize Between Databases];
    D --> F[Maintain Separate Monitoring and Alerting];
```

**Example input / output:**

```text
- **Input**: A request to write a high-value transaction.
- **Output**: The system routes the request to the strong-consistency database, ensuring atomicity and consistency.

- **Input**: A read operation for an aggregated report.
- **Output**: The system retrieves data from the eventually-consistent store, providing timely but potentially stale information.
```

**Data:**

```yaml
# Example Schema for Partitioned Data
data:
  - type: critical
    db_type: strongly_consistent
  - type: non_critical
    db_type: eventually_consistent
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Data Critical?
5. Use Strongly Consistent DB
6. Use Eventually ConsistentStore
7. Synchronize with Main DB
8. Maintain Separate Monitoring

