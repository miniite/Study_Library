---
type: "practice"
difficulty: "Deep"
---

# What consistency guarantees does a synchronous replica give you, and what is the availability cost if that replica goes down?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Failback After Failover](<../../../../01_Databases_and_SQL/Database_Operations/Replication/Failback_After_Failover/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a database replication setup with synchronous master-slave replication, the primary concern is ensuring data consistency across all replicas. Synchronous replication guarantees strong consistency by ensuring that any write operation must be acknowledged by all replicas before it's considered successful on the client side. This means that if a replica goes down, the writes will still have been committed to other replicas or the master, but the availability of the system may suffer significantly.

#### Triaging the Question

The question is probing your understanding of consistency guarantees in database replication and how failures affect system availability. Key assumptions include:

The system uses synchronous replication.
Writes must be acknowledged by all replicas before returning a success to the client.
A replica failure means that writes cannot proceed until the replica comes back online.

#### First Principles Solution

#### Cons
istency Guarantees: Synchronous replication ensures that all replicas are up-to-date with the latest write operations. This is crucial for applications requiring strong consistency, such as financial transactions or real-time analytics.
Failure Impact on Availability:
If a single replica fails, writes may still succeed because they were already acknowledged by other replicas.
However, if enough replicas (typically more than half of the total) are unavailable, writes will fail until those replicas become available again.
Detailed Explanation
Synchronous Replication Workflow: In synchronous mode, once a write request is received by the master database, it waits for all selected replicas to acknowledge receipt and successful processing before returning a success response. This ensures that no write is lost even if some replicas are temporarily down.
Replica Availability: If a replica fails mid-write process (between receiving the write from the master and acknowledging it), the transaction will be rolled back on that replica, but the write will still have been processed by the master and other replicas. The master may also retry sending the write to the failed replica once it comes back online.
Impact of Failure:
Single Replica Failure: Writes can still succeed because they are acknowledged by at least one more replica, ensuring that the data is safe on multiple nodes.
Multiple Replicas Down: If the majority of replicas go down, writes will fail as not enough replicas can process and acknowledge them. This could lead to temporary downtime or reduced write availability.

#### Closing Summary

Synchronous replication provides strong consistency guarantees by ensuring all writes are acknowledged before returning success to the client. However, this comes at the cost of lower availability if a significant number of replicas become unavailable. The system must be designed with redundancy and failover mechanisms in mind to handle such scenarios effectively.

#### Common Mistakes/Follow-Ups
Misunderstanding Failure Modes: Failing to recognize that writes can still succeed as long as enough replicas are available.
Ignoring Recovery Mechanisms: Overlooking the need for automated recovery processes when a replica fails.
Not Considering Latency: Underestimating the impact of network latency on write performance and availability.

**Code:**

```sql
-- Example SQL for synchronous replication setup in PostgreSQL
CREATE REPLICATION SLOT 'replication_slot_name' ON database_name;
```

**Flow:**

1. Write Request
2. Acknowledge
3. Master
4. Replica 1
5. Replica 2
6. Replica 3
7. Replication Success
8. Processing Write
9. Processing Write

