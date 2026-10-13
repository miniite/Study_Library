---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd design failover for a database that participates in distributed transactions (2PC/Saga) with other systems. What happens to in-doubt transactions during a promote?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ HA Failover Design](<../../../../01_Databases_and_SQL/Database_Operations/High_Availability/HA_Failover_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a robust failover architecture for a database involved in distributed transactions, we need to ensure that transactions can be committed or rolled back seamlessly without data corruption or inconsistencies. The key concept here is two-phase commit (2PC) and sagas, which are widely used for managing complex transactional interactions across multiple systems.

#### Triaging the Question

The core of this question involves understanding how distributed transactions are managed during a failover scenario. Specifically, we need to ensure that in-doubt transactions—those that were started but not yet resolved—are handled correctly without causing data inconsistencies or lost updates. The architecture must be designed with high availability and fault tolerance in mind.

#### First-Principles Solution
Database Cluster Setup: Use a database cluster for redundancy, such as PostgreSQL with pgpool-II or Oracle RAC.
Distributed Transaction Management: Implement a distributed transaction manager that can handle 2PC (two-phase commit) and sagas.

#### Cons
istent Failover Mechanism: Design a mechanism to promote the secondary node as primary without disrupting ongoing transactions.
Handling In-Doubt Transactions: Ensure proper management of in-doubt transactions during failover.
Detailed Solution

Database Cluster Setup:

Use a database cluster where nodes are peers, and any one can take over as the primary node in case of failure.
For example, PostgreSQL with pgpool-II or Oracle RAC (Real Application Clusters) provide built-in failover capabilities.

Distributed Transaction Management:

Implement a distributed transaction manager that supports 2PC and sagas.
Use tools like Apache Kafka for saga orchestration to manage complex transactions across multiple services.

#### Cons
istent Failover Mechanism:

Design the failover mechanism to ensure that when a primary node fails, another node can take over seamlessly.
Implement heartbeat monitoring and automatic failover mechanisms using tools like pgpool-II or Oracle RAC.

Handling In-Doubt Transactions:

Ensure that during failover, all in-progress transactions are properly marked as in-doubt.
Use a mechanism to coordinate the resolution of these transactions between the old and new primary nodes.

#### Example Scenario

Let's consider a scenario where we have two database nodes: Node A (primary) and Node B (secondary). Assume both nodes participate in distributed transactions with other systems, e.g., through Kafka.

Initial State:

Node A is the primary node.
Distributed transaction T1 is initiated across multiple services.

Failover Trigger:

Node A fails.
The monitoring system detects the failure and promotes Node B to primary.

Transaction Resolution:

All transactions in progress on Node A are marked as in-doubt.
The distributed transaction manager starts a recovery process.
Node B needs to coordinate with other services involved in T1 to resolve the in-progress state of this transaction.

#### Closing Summary

In summary, designing a failover architecture for databases participating in distributed transactions involves setting up a robust database cluster, using a reliable distributed transaction manager, and implementing consistent failover mechanisms. Proper handling of in-doubt transactions is critical to maintaining data integrity during such scenarios.

#### Common Mistakes/Follow-Ups
Mistake 1: Not properly marking transactions as in-doubt, leading to data inconsistencies.
Mistake 2: Failing to coordinate with other services involved in distributed transactions, causing lost updates.

#### Follow-Up
Discussing the trade-offs between different failover mechanisms and transaction management strategies.

**Code:**

```sql
-- Example SQL for marking a transaction as in-doubt
UPDATE transaction SET status = 'IN_DOUBT' WHERE id = 'T1';
```

**Example input / output:**

| Input (Initial State)        | Output (After Failover)       |
|------------------------------|-------------------------------|
| Node A as primary, T1 in-progress | Node B promoted to primary, T1 marked IN_DOUBT |

**Data:**

```json
{
  "transaction_id": "T1",
  "status": "IN_PROGRESS",
  "participants": ["NodeA", "ServiceX"]
}
```

