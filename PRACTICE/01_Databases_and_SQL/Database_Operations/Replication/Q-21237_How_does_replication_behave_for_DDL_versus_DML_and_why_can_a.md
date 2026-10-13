---
type: "practice"
difficulty: "Deep"
---

# How does replication behave for DDL versus DML, and why can a slow ALTER on the master block the apply pipeline on replicas?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Schema Changes and Failover](<../../../../01_Databases_and_SQL/Database_Operations/Replication/Failback_After_Failover/Schema_Changes_and_Failover.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing database replication in a Master-Slave setup, it's essential to understand how different types of data manipulation language (DML) and data definition language (DDL) operations are handled. In this scenario, we'll focus on the behavior of DDL versus DML commands, with a particular emphasis on why an ALTER TABLE operation on the master can block replication pipelines on replicas.

#### Triaging the Question

The question is essentially asking about the differences in how CREATE, ALTER, and DROP (DDL) statements are treated compared to INSERT, UPDATE, and DELETE (DML) operations during database replication. The core issue revolves around understanding why a slow ALTER TABLE on the master can cause blockages in the apply pipeline of replicas.

#### Assumptions
We're dealing with a Master-Slave replication setup, where changes from the master are replicated to one or more slaves.
Replication is asynchronous by default but can be synchronous for critical operations like ALTER TABLE.
The primary focus will be on MySQL and PostgreSQL, as these two database systems handle DDL differently.

#### First-Principles Solution
1. DDL Operations:

CREATE/ALTER/DROP Statements: These are special because they define the structure of the database schema. Unlike DML operations that only modify data, DDL commands can cause significant changes to the database's schema.

Replication Behavior:

Master-Slave Replication: By default, CREATE, ALTER, and DROP statements are not replicated directly from master to slave. Instead, a mechanism called "statement-based replication" is used where only SQL commands that change the database state are sent from the master to slaves.
Synchronous vs Asynchronous: If synchronous replication is enabled for DDL operations (e.g., in MySQL's GTID mode), all transactions on the master must be committed before the slave acknowledges them. This can cause significant delays if the ALTER TABLE operation takes a long time.
2. DML Operations:

INSERT/UPDATE/DELETE Statements: These modify data within existing tables and are generally faster to replicate.

Replication Behavior:

In most setups, DML operations are replicated asynchronously unless configured otherwise. This means that once a transaction is committed on the master, it will eventually be applied to the slaves.
Why Does ALTER TABLE Block Replication?

When you execute an ALTER TABLE command on the master:

Locking and Schema Changes: The operation locks the table(s) involved in the schema change, which prevents any DML operations from being executed until the alteration is complete.

Network Latency: Even after the ALTER TABLE completes, there's a delay due to network latency as the SQL command needs to be sent to all slaves and applied.

Transaction Commit and Acknowledgment: In synchronous replication setups (like GTID mode in MySQL), the master waits for acknowledgment from all replicas before marking the transaction as committed. If any replica takes longer, it can block the entire pipeline until it catches up.

#### Closing Summary

In summary, ALTER TABLE operations are critical because they fundamentally change how data is stored and structured. This makes them slower to apply compared to DML commands. In synchronous replication setups, these operations can cause delays or even block the replication pipeline due to lock contention and network latency.

#### Common Mistakes/Follow-Ups
Assuming DDL Operations are Instant: Developers often overlook that ALTER TABLE operations take time and can lead to unexpected delays.
Misconfiguring Synchronous Replication: Setting up synchronous replication for all critical operations without considering the performance implications can cause significant bottlenecks.
Failing to Plan for Schema Changes: Without proper planning, schema changes on a master can lead to cascading issues in replicas.
Triaging

To mitigate these issues:

Use asynchronous replication where possible and only enable synchronous replication for critical operations.
Monitor the performance of ALTER TABLE operations and plan them during off-peak hours.
Implement failover mechanisms to ensure high availability even if one replica lags behind.

**Comparison:**

| Operation | Time on Master (s) | Time on Replica (s) |
|-----------|--------------------|---------------------|
| DDL       | 30                 | 20                  |
| DML       | 5                  | 5                   |

**Code:**

```sql
-- Example DDL command
ALTER TABLE customers ADD COLUMN address VARCHAR(255);

-- Example DML command
UPDATE customers SET address = 'New Address' WHERE id = 1;
```

**Example input / output:**

```text
Input:
- Master executes `ALTER TABLE customers ADD COLUMN address VARCHAR(255);`

Output (Replica):
1. Receives command
2. Applies schema change
3. Notifies master of success
4. Master waits for all replicas to confirm

Delay Analysis:
- Master lock duration: 30 seconds
- Network latency: 10 seconds
- Total delay: 40 seconds
```

**Data:**

```json
{
  "master": {
    "operation": "ALTER TABLE",
    "duration": "30 seconds"
  },
  "replica": {
    "operation": "RECEIVE AND APPLY SQL COMMANDS",
    "duration": "20 seconds"
  }
}
```

**Flow:**

1. Yes
2. No
3. Master executes ALTER TABLE
4. Locks table and waits forcompletion
5. Replica receives command
6. Is network latency significant?
7. Apply schema change, notifymaster
8. Apply data changes directly
9. Master waits for all replicas toconfirm before committingtransaction

