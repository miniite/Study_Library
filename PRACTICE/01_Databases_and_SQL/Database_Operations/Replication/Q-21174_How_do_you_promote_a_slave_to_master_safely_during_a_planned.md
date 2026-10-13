---
type: "practice"
difficulty: "Deep"
---

# How do you promote a slave to master safely during a planned master maintenance window without losing writes?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Failback After Failover](<../../../../01_Databases_and_SQL/Database_Operations/Replication/Failback_After_Failover/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database replication, especially in a Master-Slave setup, promoting a slave to become the new master involves several steps to ensure data consistency and avoid data loss, particularly during a planned maintenance window when write operations are still active. This process requires careful planning and execution.

#### Triage

The question is probing how to safely transition a slave database to master while ensuring that no writes are lost during this operation. Key assumptions include:

The current master will temporarily pause write operations.
The slave needs to catch up with the latest changes before being promoted.
There may be some downtime or disruption during the promotion process.

#### First Principles Solution
Pause Write Operations on Current Master: Temporarily stop all write operations on the existing master database.
Catch-Up the Slave: Ensure that the slave has caught up with the master in terms of data replication.
Promote Slave to Master:
Replicate the state of the current master to the slave.
Switch any active connections from the old master to the new master.
Resume Write Operations on New Master: Once the transition is complete, resume write operations on the new master.
Detailed Steps

Pause Write Operations:

Use transaction controls or database-specific commands (e.g., SET GLOBAL READ_ONLY = 1 for MySQL) to prevent writes.

Catch-Up the Slave:

Ensure that the slave is fully up-to-date with the master’s data.
In many systems, this can be done by setting MASTER_DELAY on the slave to a high value and waiting until it catches up.

Promote Slave to Master:

Take a snapshot of the current state of the master database (e.g., using tools like mysqldump).
Apply this snapshot to the slave.
Configure the slave as the new master by modifying its configuration files or through management tools.

Switch Connections:

Update any applications and infrastructure configurations to point to the new master address.

Resume Write Operations:

Once the transition is complete, resume write operations on the new master.

#### Closing Summary

To safely promote a slave to master during planned maintenance, it’s crucial to pause writes on the current master, ensure the slave catches up, replicate the state of the master to the slave, switch connections, and then resume writes. This process minimizes downtime and ensures data integrity.

#### Common Mistakes/Follow-Ups
Common Mistake: Failing to fully catch up the slave before promotion can lead to data loss.

#### Follow-Up
Discussing how to handle read operations during the transition and ensuring zero-downtime failover mechanisms are in place for production environments.

**Code:**

```sql
-- Example SQL commands to pause writes on master (MySQL)
SET GLOBAL READ_ONLY = 1;

-- Example command to catch up slave
CHANGE MASTER TO ...
START SLAVE;
```

**Example input / output:**

```text
Input:
- Current Master: DB_MASTER_01
- Slave: DB_SLAVE_01

Output:
1. Paused write operations on DB_MASTER_01.
2. Ensured DB_SLAVE_01 caught up with the latest data.
3. Promoted DB_SLAVE_01 to new master, applied snapshot from DB_MASTER_01.
4. Switched connections and resumed writes on the new master.
```

**Flow:**

1. Yes
2. No
3. Pause Write Operations
4. Is Slave Up-to-Date?
5. Replicate Master State to Slave
6. Wait for Slave Catch-Up
7. Promote Slave to New Master
8. Switch Connections andResume Write Operations

