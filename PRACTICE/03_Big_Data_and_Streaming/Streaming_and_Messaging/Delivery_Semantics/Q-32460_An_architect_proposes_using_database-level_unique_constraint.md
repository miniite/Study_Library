---
type: "practice"
difficulty: "Deep"
---

# An architect proposes using database-level unique constraints as the dedup mechanism instead of a dedup store. Adjudicate this: when is 'let the DB reject the duplicate insert' the right idempotency primitive, and when does it fail?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Idempotency and Schema Evolution](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Idempotency_and_Schema_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When considering whether to use database-level unique constraints as a deduplication mechanism instead of a dedicated dedup store, we need to weigh several factors including reliability, performance, simplicity, and maintainability. The core idea here is understanding when database-level constraints provide sufficient guarantees for idempotency and when they fall short.

The proposed approach leverages the fact that unique constraints are designed to ensure data integrity by preventing duplicate entries from being inserted into a table. However, using this mechanism alone has limitations in ensuring at-least-once or exactly-once semantics, which are crucial in many message delivery scenarios.

#### Triaging the Question
What is being probed: The interviewer is assessing your understanding of idempotency and its implementation via unique constraints versus dedicated deduplication stores.
Key assumptions: We assume a typical transactional database setup with ACID properties, but are also considering potential edge cases like network partitions or concurrent transactions.
Step-by-First Principles Solution

Idempotency Basics:

At-Least-Once (ALO): Ensures each message is processed at least once.
Exactly-Once (EO): Ensures each message is processed exactly once, which requires additional handling for retries and failures.

Database Constraints as Idempotency Mechanism:

#### Pros
Simple to implement; leverages existing database features.
Can handle a high volume of writes efficiently.

#### Cons
Limited in ensuring idempotency, especially under distributed systems or during concurrent transactions.

Database Unique Constraints vs Dedup Store:

Unique Constraint: Guarantees that no duplicate primary keys will be inserted into the table.
Dedup Store: Manages deduplication logic explicitly and can provide stronger guarantees about exactly-once delivery, often with compensating actions for retries.
Step-by-Step Solution

Transaction Safety:

Single Database Node: In a single database node setup, unique constraints work well as they ensure data integrity.
Multi-Node Setup: In distributed databases or across multiple nodes, the same transaction might be replayed in different nodes, leading to duplicate entries even with unique constraints.

Transaction Isolation Levels:

Read Committed: Allows dirty reads and can miss updates by concurrent transactions.
Repeatable Read (RC): Ensures that a transaction sees consistent data throughout its execution but may still see stale data in the presence of concurrent inserts.
Serializable (S): Ensures that each transaction appears to execute serially, avoiding conflicts. However, it is very restrictive and may impact performance.

Network Partitions:

In a network partition scenario, if one node fails or disconnects while another processes the same message, both might insert the duplicate entry due to lack of distributed coordination.

Handling Retries:

Transient Errors: A unique constraint will fail during retries but won't necessarily ensure exactly-once processing.
Persistent Errors: If a transient error is not detected, retries can lead to duplicates, violating exactly-once semantics.

#### Closing Summary

Using database-level unique constraints as the dedup mechanism is practical and simple in single-node setups or when transactional safety guarantees are sufficient. However, it fails to provide strong idempotency guarantees in distributed systems due to issues like concurrent transactions, network partitions, and retry handling. A dedicated dedup store offers better control over exactly-once processing but comes with increased complexity.

#### Common Mistakes/Follow-Ups
Misunderstanding ACID Properties: Failing to recognize that unique constraints do not inherently provide the necessary guarantees for distributed systems.
Overlooking Transaction Isolation Levels: Assuming all transactions behave as if they are in a serializable environment without considering RC or RR modes.
Ignoring Network Partitions: Not accounting for scenarios where nodes might process messages independently, leading to duplicates.

**Comparison:**

| Transaction | Isolation Level | Result |
| ----------- | -------------- | ------ |
| Single Node | Any            | ✅      |
| Multi-Node  | RC             | ❌      |
|            | S              | ✅ (with distributed coordination) |

**Code:**

```sql
-- Example SQL Schema
CREATE TABLE Messages (
    message_id BIGINT PRIMARY KEY,
    content TEXT NOT NULL
);
```

**Example input / output:**

| Input Message | Database Response |
|---------------|------------------|
| {"message_id": 12345, "content": "Hello, world!"} | Duplicate key error |
| Retrying with same message | Duplicate key error |

**Data:**

```json
{
  "message_id": 12345,
  "content": "Hello, world!"
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Message Received
6. Is Unique Constraint Enabled?
7. Record Inserted
8. Use Dedup Store
9. Network Partition Present?
10. Duplicate Entry Possible
11. Unique Insert Succeeded

