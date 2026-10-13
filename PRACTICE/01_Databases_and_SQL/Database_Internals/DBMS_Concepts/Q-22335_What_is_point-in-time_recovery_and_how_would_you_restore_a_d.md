---
type: "practice"
difficulty: "Deep"
---

# What is point-in-time recovery and how would you restore a database to 5 minutes before a bad deploy?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1 · [↑ Batching Writes](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Batching_Writes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Point-in-time recovery (PITR) allows restoring a database to a previous state, typically by using backup files or logs from an earlier timestamp. In this scenario, we need to restore the database to 5 minutes before a bad deployment. To achieve this, we'll explore how PITR works and implement it step-by-step.

Firstly, I would like to clarify that the question assumes a setup with a database system that supports PITR. Common systems include PostgreSQL, MySQL, and others that use transaction logs (WAL) for recovery purposes. The core idea here is to leverage these logs to roll back the database to a specified point in time.

#### Assumptions

The database has been configured for PITR.
Transaction logs are available within 5 minutes of the bad deploy.
A backup exists up to at least that time frame.

Let's reason through this from first principles:

Identify the Starting Point: Determine the exact timestamp before the bad deploy. For example, if the bad deploy happened at 2023-11-28T14:30:00Z, we need to restore the database state to around 2023-11-28T14:25:00Z.

Locate the Last Full Backup: If a full backup exists within our timeline, it will serve as the starting point for PITR. For this example, let's assume we have a full backup at 2023-11-28T14:00:00Z.

Identify Log Files: The database system maintains transaction logs (WAL) that record every change made to the database since the last backup. These logs are incremental and need to be applied in chronological order.

Roll Forward from the Backup: Apply all log files up to 5 minutes before the bad deploy time, effectively rolling forward the state of the database from the backup to a point just before the bad deploy.

Roll Back to the Desired Point: Once we have rolled forward as far as possible with the logs (up to 2023-11-28T14:25:00Z), no further log files are needed, and we can finalize the restoration by stopping here.

In practice, this involves executing a series of SQL commands or using a database management tool to perform these operations. Here's an example of how this might look in PostgreSQL:

SQL
-- Start the recovery process from the full backup
RECOVER FROM '2023-11-28T14:00:00Z' TO '2023-11-28T14:25:00Z';

-- Apply transaction logs incrementally
SELECT pg_start_backup('recovery', true);
-- Copy the WAL files up to 2023-11-28T14:25:00Z from the backup location
SELECT pg_stop_backup();

This sequence of commands will ensure that the database state is restored as close as possible to the desired point in time.

#### Closing Summary
Point-in-time recovery involves leveraging a full backup and transaction logs to restore a database to a specific point in time. The process requires identifying the appropriate starting point, applying the necessary log files, and stopping at the desired timestamp. This method allows for precise state restoration without losing any changes made since the last known good state.

#### Common Mistakes/Follow-ups

Forgetting to locate or apply all relevant logs.
Starting recovery from an incorrect backup point.
Not ensuring data integrity checks before finalizing the restore.
Misunderstanding the impact of incomplete log files on the restoration process.

**Comparison:**

| Timestamp       | Action            |
|----------------|-------------------|
| 2023-11-28T14:00 | Full Backup       |
| 2023-11-28T14:25 | Target Restoration|

**Code:**

```sql
RECOVER FROM '2023-11-28T14:00:00Z' TO '2023-11-28T14:25:00Z';
SELECT pg_start_backup('recovery', true);
-- Copy WAL files manually or through a tool
SELECT pg_stop_backup();
```

**Example input / output:**

```text
Input:
RECOVER FROM '2023-11-28T14:00:00Z' TO '2023-11-28T14:25:00Z';
SELECT pg_start_backup('recovery', true);
-- Copy WAL files
SELECT pg_stop_backup();

Output:
Database state restored to 2023-11-28T14:25:00Z.
```

**Data:**

```json
{
  "backup_timestamp": "2023-11-28T14:00:00Z",
  "target_timestamp": "2023-11-28T14:25:00Z"
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Full Backup Available?
5. Use Full Backup
6. Apply Logs Incrementally
7. '2023-11-28T14:25:00Z' TargetReached
8. Finalize Restoration
9. Use Latest Full Backup
10. Apply Logs from Backup toTarget Timestamp
11. '2023-11-28T14:25:00Z' TargetReached

