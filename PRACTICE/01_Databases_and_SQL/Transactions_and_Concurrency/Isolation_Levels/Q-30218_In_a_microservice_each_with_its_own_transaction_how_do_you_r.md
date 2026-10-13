---
type: "practice"
difficulty: "Deep"
---

# In a microservice each with its own transaction, how do you reason about isolation across service boundaries? Why doesn't a single DB isolation level save you?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Setting Isolation Levels](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a microservice architecture where each service runs transactions independently, ensuring isolation between services is crucial to maintain data consistency and integrity. Unlike a monolithic application, isolating transactions across multiple databases or even multiple instances of the same database can be challenging due to the distributed nature of these systems. A single DB isolation level like Repeatable Read (RR), Read Committed (RC), or Serializable (Serializable) does not suffice because each service’s transaction is isolated from the others.

To reason about isolation across service boundaries, we need to consider the broader context of consistency models and the specific challenges they pose in a distributed system. Let's break it down step by step:

#### Triaging the Question

The question revolves around understanding how isolation levels work within microservices and why a single DB isolation level cannot ensure overall data consistency when transactions span multiple services.

#### First-Principles Solution

Understanding Isolation Levels:

Read Uncommitted (RU): Allows dirty reads, which can lead to inconsistent states.
Read Committed (RC): Ensures that a transaction only sees committed changes, preventing dirty reads but not ensuring repeatable reads or snapshot isolation.
Repeatable Read (RR): Guarantees that multiple identical queries within the same transaction return the same results, but it may still suffer from phantom reads and serialization anomalies.
Serializable (S): Ensures all transactions are executed as if they were run in a serial order, providing the strongest isolation level.

Challenges of Distributed Transactions:

Non-Atomicity: A single transaction cannot be guaranteed to execute atomically across multiple services due to network latency and potential failures.

#### Cons
istency vs. Availability: Achieving strong consistency (like Serializable) in a distributed system often comes at the cost of availability, as seen in CAP Theorem implications.
Detailed Reasoning

Isolation Levels in Monolithic Systems:

In monoliths, each transaction operates on a single database, and isolation levels like RR or S can be applied to ensure data consistency.

Distributed Transactions Across Services:

When transactions span multiple services, you must consider how changes are propagated between them.
Each service might have its own database with its own transaction, but these transactions need to coordinate effectively.

Propagating Isolation Levels:

If a service A updates data in DB1 and expects an update in DB2 as part of the same transaction, it must ensure that both operations succeed or fail atomically.
However, applying isolation levels like RR or S directly across services is complex due to network delays and potential failures.

Eventual Consistency:

In distributed systems, it’s often more practical to aim for eventual consistency using techniques like event sourcing and compensating transactions.
Services can asynchronously publish events that other services consume and act upon, ensuring that changes are propagated correctly over time.

Two-Phase Commit (2PC):

2PC is a protocol used in distributed transaction management but comes with its own challenges such as timeouts and the complexity of managing distributed locks.
It ensures ACID properties across multiple resources but may lead to performance bottlenecks and potential deadlocks.

#### Closing Summary

Isolation levels like RR or S are essential for ensuring data consistency within a single service, but they do not inherently extend their guarantees to other services in a microservice architecture. To achieve consistent state changes across multiple services, you must design distributed transaction management strategies that handle asynchronous communication, eventual consistency, and possibly 2PC with careful consideration of performance and availability trade-offs.

#### Common Mistakes/Follow-Ups
Mistake: Assuming that a single DB isolation level can ensure consistency across microservices.

#### Follow-Up
Discuss how to design compensating transactions or use event sourcing to manage eventual consistency in distributed systems.

