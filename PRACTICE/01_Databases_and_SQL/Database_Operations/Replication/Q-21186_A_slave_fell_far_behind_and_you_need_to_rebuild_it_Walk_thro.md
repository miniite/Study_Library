---
type: "practice"
difficulty: "Intermediate"
---

# A slave fell far behind and you need to rebuild it. Walk through re-cloning it from a consistent backup without stopping the master.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Failback After Failover](<../../../../01_Databases_and_SQL/Database_Operations/Replication/Failback_After_Failover/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When faced with the challenge of rebuilding a database slave that has fallen significantly behind, the goal is to bring it back into sync with minimal disruption to the master. This process involves several steps and requires careful planning to ensure data consistency and minimize downtime.

The key concept here is recloning from a consistent backup without interrupting the master server's operation. The primary challenge is ensuring that the slave can be brought up to date while the master continues processing transactions. Let’s break down the process step by step:

#### Assumptions

We have access to a recent, consistent backup of the master database.
The replication topology includes at least one active master and multiple slaves.
The master is running and accepting writes.

Steps:

Stop Replication on Slave: First, we need to pause all replication activities from the current slave.
Restore Backup: Restore the recent consistent backup onto a new instance or environment that will serve as the new slave.
Reconfigure New Slave: Configure this new slave to start replicating from the master. This involves setting up its replication user, credentials, and ensuring it can connect to the master.
Verify Replication Lag: Monitor the new slave's replication lag until it catches up with the master.

Here’s a detailed walk-through of each step:

Step-by-Step Solution

Stop Replication on Slave:

SQL
STOP SLAVE;

This command stops all ongoing replication activities and allows us to safely perform a backup without data corruption.

Restore Backup:

Use mysql or mysqldump to restore the recent consistent backup onto a new instance.
Ensure that you are using an appropriate point-in-time recovery method if your database supports it, such as InnoDB’s --apply-log and --copy-back options with innobackupex.

Reconfigure New Slave:

Connect to the new slave instance and configure its replication settings.
Set up a replication user on the master that has the necessary privileges.
SQL
GRANT REPLICATION SLAVE ON *.* TO 'repl_user'@'%';
Reset the slave’s binary log coordinates if needed, ensuring it knows where to start replicating from:
SQL
CHANGE MASTER TO MASTER_USER='repl_user', MASTER_HOST='<master_host>', MASTER_PASSWORD='<password>', MASTER_LOG_FILE='<log_file_name>', MASTER_LOG_POS=<log_position>;
Start the slave with START SLAVE; and monitor its progress.

Verify Replication Lag:

Use MySQL’s built-in tools to check the replication lag.
SQL
SHOW SLAVE STATUS\G;
Look for the Seconds_Behind_Master field, which indicates how long the slave is behind the master.
Ensure that this value is zero or very close to it. If there’s a significant lag, you might need to manually apply missing transactions.

#### Closing Summary

By following these steps, we can effectively rebuild a slave database from a consistent backup without stopping the master server's operation. This process minimizes downtime and ensures data consistency between the master and new slave.

#### Common Mistakes/Follow-Ups
Mistake 1: Failing to stop replication on the current slave before performing the backup.
Mistake 2: Not setting up a robust monitoring mechanism during the reconfiguration phase.

#### Follow-Up
Ensuring that the master’s transaction logs are not truncated too early, which can lead to missing transactions on the new slave.

**Code:**

```

STOP SLAVE;

CODE
Copy
```sql
GRANT REPLICATION SLAVE ON *.* TO 'repl_user'@'%';
CHANGE MASTER TO MASTER_USER='repl_user', MASTER_HOST='<master_host>', MASTER_PASSWORD='<password>', MASTER_LOG_FILE='<log_file_name>', MASTER_LOG_POS=<log_position>;
START SLAVE;
```

**Flow:**

1. Stop Replication on Slave
2. Backup Database
3. Restore Backup to NewInstance
4. Reconfigure New Slave
5. Verify Replication Lag
6. Monitor Progress, EnsureSynchronization

