---
type: "practice"
difficulty: "Deep"
---

# How do you keep the master and replicas schema-compatible during an online schema change so replication doesn't break?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Schema Changes and Failover](<../../../../01_Databases_and_SQL/Database_Operations/Replication/Failback_After_Failover/Schema_Changes_and_Failover.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a database's replica remains schema-compatible with its master during an online schema change, we need to implement a controlled process that minimizes disruption and ensures data consistency. This involves carefully planning the change, managing transactions across nodes, and leveraging advanced replication features. The core idea is to temporarily pause or isolate writes on the replicas while changes are being applied to the master.

First, let's break down the key components of this question:

Understanding the Problem: We need to handle schema changes in a way that doesn't disrupt ongoing transactions on the replica.

#### Assumptions
The database supports some form of transactional consistency.
Replicas are kept up-to-date through asynchronous or semi-synchronous replication.

#### Solution
Approach: We'll use techniques like temporary isolation, schema versioning, and transaction coordination.
Step-by-Step Solution

Preparation Phase:

Identify the changes needed in the schema (e.g., adding a column, altering data types).
Prepare any necessary scripts or tools to apply these changes.

Isolation of Replicas:

Temporarily pause write operations on replicas during the change. This can be done by setting them into read-only mode or using a feature like replica_isolation if available in your database system.

Schema Change on Master:

Apply the schema changes to the master database.
Ensure that any necessary transactions are completed before proceeding with the next steps.

Synchronization Phase:

After the master's schema has been updated, resume writes on replicas if they were paused.
Allow replica catch-up by resuming replication from a consistent state (e.g., last known good checkpoint).

Post-Change Validation:

Verify that all replicas are in sync and no data corruption occurred during the change process.
Detailed Workflow

Pause Replicas Temporarily:

SQL
ALTER DATABASE replica_name SET read_only = ON;

Apply Schema Changes on Master:

SQL
-- Example: Adding a column to a table
ALTER TABLE my_table ADD COLUMN new_column VARCHAR(255);

Resume Replicas and Synchronize:

SQL
ALTER DATABASE replica_name SET read_only = OFF;
-- Wait for replicas to catch up with the master

Validation Check:

SQL
-- Run integrity checks on all nodes
REPAIR TABLE my_table;

#### Closing Summary
and Common Mistakes

In summary, keeping schemas compatible during online schema changes requires careful planning and execution. The key steps include isolating replicas, applying the change on the master, allowing for synchronization, and validating the new state.

#### Common mistakes
to avoid:

Failing to pause writes on replicas, leading to incomplete or corrupt data.
Not verifying the consistency of all nodes after the change.
Ignoring transactional dependencies that might still be executing during the schema change window.

**Code:**

```sql
-- Example SQL commands for isolation and synchronization
ALTER DATABASE replica_name SET read_only = ON;
ALTER TABLE my_table ADD COLUMN new_column VARCHAR(255);
ALTER DATABASE replica_name SET read_only = OFF;
REPAIR TABLE my_table;
```

**Flow:**

1. Identify Schema Changes
2. Prepare Scripts/Tools
3. Pause Replicas
4. Apply Schema Changes onMaster
5. Resume Replicas andSynchronize
6. Validate New State

