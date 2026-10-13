---
type: "practice"
difficulty: "Expert"
---

# How do you architect for the case where MDM must serve both low-latency operational lookups (single entity by key) and heavy analytical match jobs — one store, CQRS, or two stores, and why?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ MDM for Batch and Streaming](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Master_Data_Management/MDM_for_Batch_and_Streaming.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of architecting a Master Data Management (MDM) system that serves both low-latency operational lookups and heavy analytical match jobs, we need to carefully consider our design choices. The primary decision points are whether to use one store, Command Query Responsibility Segregation (CQRS), or two separate stores. Each approach has its trade-offs in terms of consistency, performance, and complexity.

#### Triaging the Question

The question is asking for an architectural design that balances the need for low-latency operational queries with the requirement for heavy analytical matching. The interviewer is probing your understanding of MDM principles, data architecture patterns like CQRS, and how to handle different types of workloads efficiently. Key assumptions include:

Both operational lookups and analytics are critical business requirements.
We have a single set of master data that needs to be managed.

#### First Principles Solution
One Store Approach:
This is the simplest design where we use a single database or system to handle both operations. However, it can lead to performance bottlenecks and potential consistency issues during write-heavy periods.
CQRS (Command Query Responsibility Segregation):
CQRS involves separating the read model from the write model. The operational queries (reads) are served by a highly optimized read model that is decoupled from the write model, which handles updates.
Two Stores Approach:
Using two separate stores can ensure that operational lookups are fast and analytical jobs have their own optimized environment. However, this introduces complexity in ensuring data consistency between the two.
Detailed Solution
One Store Approach

#### Pros
Simplicity; easier to maintain a single source of truth.

#### Cons
During heavy write operations, read performance can degrade significantly due to locking and contention.

#### Example
A single SQL database or NoSQL collection could be used for both operational lookups and analytics.
CQRS Approach
Write Model (Operational Data):
Use a transactional system like a relational database or an event sourcing system. Ensure that writes are optimized but can lead to slower reads.
Read Model (Analytical Data):
Build an optimized read model that uses denormalization, indexing, and materialized views to ensure fast queries for analytics. This could be another database instance running alongside the write model or a separate data warehouse.
Two Stores Approach
Operational Store:
Use a high-performance system like Redis or a distributed key-value store optimized for read-heavy workloads.
Analytical Store:
Use a data warehouse like Amazon Redshift, Google BigQuery, or Snowflake to handle complex analytical queries.
Choosing the Right Approach

The choice between these approaches depends on the specific business requirements and constraints. For instance:

If operational lookups are extremely critical (e.g., real-time financial transactions), then CQRS with a well-designed read model might be necessary.
If there is more tolerance for slightly slower reads but lower write latency, a two-store approach could be beneficial.

#### Closing Summary

In summary, the architecture for an MDM system that balances operational lookups and heavy analytical jobs requires careful consideration of trade-offs. CQRS with optimized read models or using separate stores can provide a good balance between performance and consistency. The specific choice will depend on the nature of the workload and business priorities.

#### Common Mistakes/Follow-Ups
Failing to recognize that a single store might not scale well for both types of operations.
Not considering the complexity introduced by CQRS or two stores, which can lead to more maintenance overhead.
Overlooking the importance of data consistency during migrations between different models/stores.

**Comparison:**

| Operation | One Store | CQRS | Two Stores |
|-----------|-----------|------|------------|
| Consistency | High | Medium | Low |
| Read Performance | Variable, can degrade | Good (optimized read model) | Excellent (separate stores) |
| Write Performance | Fast but can cause bottlenecks during writes | Slower writes, optimized reads | Very fast writes |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Write Model (Operational Data)] --> B{Use Relational DB?}
    B -- Yes --> C[Transaction Handling]
    B -- No --> D[Event Sourcing]

    A --> E[Read Model (Analytical Data)]
    E -- Use Denormalization --> F[Materialized Views]
    E -- Use Indexing --> G[Optimized Queries]
```

**Example input / output:**

```text
Input (Write Model):
POST /data
{
  "entityKey": "abc123",
  "attributes": {
    "name": "John Doe",
    "age": 30,
    "address": "123 Street"
  }
}

Output (Write Model):
{
  "status": "success",
  "message": "Data updated successfully."
}
```

**Data:**

```json
{
  "operationalStore": {
    "type": "SQL",
    "capacity": 1000,
    "queriesPerSecond": 5000
  },
  "analyticalStore": {
    "type": "Data Warehouse",
    "capacity": 2000,
    "queriesPerSecond": 10000
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Data Update
6. Use Write Model?
7. Write to Operational Store
8. Generate Events
9. Write to Event Store
10. Use Read Model?
11. Query Analytical Store
12. Direct Query Operational Store

