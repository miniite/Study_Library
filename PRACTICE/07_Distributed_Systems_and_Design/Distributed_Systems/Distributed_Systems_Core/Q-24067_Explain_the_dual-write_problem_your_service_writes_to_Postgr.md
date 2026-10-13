---
type: "practice"
difficulty: "Deep"
---

# Explain the dual-write problem: your service writes to Postgres and publishes to Kafka in the same request. Why can these two writes diverge, and how does the transactional outbox pattern (single DB transaction plus a CDC relay) fix it?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Zero Downtime Sharding Migrations](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Zero_Downtime_Sharding_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In distributed systems where microservices interact with multiple databases or message brokers like Kafka, ensuring data consistency can be challenging. One common issue arises when your service writes to Postgres (a relational database) and publishes an event to Kafka (a messaging broker) in the same request, known as a dual-write problem. This can lead to divergence because the two systems might not maintain consistent state due to their inherent differences and operational characteristics.

To understand this better, let's break it down step by step using first principles:

Understanding Dual-Write: When your service writes to Postgres and publishes an event to Kafka in a single request, there is no guarantee that both operations will succeed or fail together. If the database write succeeds but the Kafka publish fails, you end up with inconsistent data.

Reasoning from First Principles:

Postgres: ACID properties ensure consistency within the transaction; if the transaction commits, all changes are applied to the database.
Kafka: Eventual consistency is its default behavior; messages might not be delivered immediately or in order due to network delays, retries, etc.

Failure Modes:

If your service writes to Postgres and publishes to Kafka within a single transaction, what happens if there's a network partition? The database might commit the transaction but Kafka might fail to deliver the message.
Conversely, if Kafka is able to publish the event, but the database write fails due to some error or timeout, you end up with an inconsistent state.

The Transactional Outbox Pattern: This pattern solves this issue by ensuring that both writes (database and Kafka) are part of a single transaction. Here’s how it works:

The service starts a database transaction.
It performs the write operation to Postgres within this transaction.
If successful, it publishes an event to Kafka using a CDC relay mechanism that is part of the same transaction.
If any step fails (e.g., network partition), both writes are rolled back.

Benefits:

Ensures atomicity and consistency: Both operations must succeed or fail together, maintaining data integrity.
Simplifies error handling and rollback mechanisms since you can rely on the database's transaction management.

Challenges:

Complexity in managing transactions across multiple systems.
Performance overhead due to additional coordination required.

**Code:**

```
MERMAID
Copy
sequenceDiagram
    participant Service
    participant Postgres
    participant Kafka
    participant CDC_Relay

    Service->>Postgres: Start Transaction, Write Data
    Postgres-->>Service: Acknowledge
    Service->>CDC_Relay: Publish Event
    CDC_Relay-->>Kafka: Publish Event
    Kafka-->>Service: Acknowledge
    Service->>Postgres: Commit Transaction
```

