---
type: "practice"
difficulty: "Expert"
---

# A checksum mismatch shows drift on one table between master and replica. How do you repair the replica with pt-table-sync safely?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Schema Changes and Failover](<../../../../01_Databases_and_SQL/Database_Operations/Replication/Failback_After_Failover/Schema_Changes_and_Failover.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To repair a replica that shows a checksum mismatch due to drift in one specific table, we can use pt-table-sync from Percona Toolkit as part of a well-structured process designed to minimize data inconsistencies while ensuring safety and integrity.

#### Triaging the Question

The question is asking how to safely reconcile data discrepancies between the master and replica using pt-table-sync, which involves several steps: identifying the drift, synchronizing the table, and validating the results. The key assumptions include that the database schema and structure are consistent between the master and the replica.

#### First-Principles Solution
Identify Drift: Confirm the checksum mismatch to ensure it's not a transient issue.
Prepare for Synchronization:
Use pt-table-checksum to verify the checksums of both the master and replica.
Ensure both servers are in a consistent state before starting the sync process.
Run pt-table-sync:
Execute pt-table-sync --sync-to-master --replicate=your_table --where="column=value" on the replica server.
Post-Synchronization Validation: Verify that the data is now consistent by re-running checksums.
Step-by-Step Solution

Identify Drift:

Run pt-table-checksum to check for any discrepancies between the master and replica tables. Example command:
SH
pt-table-checksum --master-host=master_host --master-user=replicator --master-password=replicator_password --compare-where="your_table.id > 100" --statistics --no-same-set
This will provide a summary of any mismatches.

Prepare for Synchronization:

Ensure both the master and replica are in sync by running pt-table-checksum on the replica to see if it matches the master.
If necessary, apply any pending transactions from the master to the replica using mysqlbinlog.

Run pt-table-sync:

Identify the specific table that needs to be synchronized, and run pt-table-sync. Example command:
SH
pt-table-sync --sync-to-master --replicate=your_table --where="column=value" --ask-pass
This will attempt to synchronize the specified table from the master to the replica.

Post-Synchronization Validation:

Re-run pt-table-checksum on the replica to verify that the checksums now match those of the master.
Confirm data integrity by checking for any remaining mismatches or errors.

#### Closing Summary

Using pt-table-sync effectively repairs data discrepancies between a master and replica, ensuring safe and consistent synchronization. It's crucial to validate both before and after using these tools to ensure no data is lost or corrupted during the process.

#### Common Mistakes
and Follow-Ups
Mistake 1: Ignoring transient issues that can cause false checksum mismatches.
Mistake 2: Not ensuring both master and replica are in a consistent state before starting the sync.

#### Follow-Up
Consider automating regular checks and synchronizations to prevent such issues from arising.

**Comparison:**

| Step | Action |
|---|---|
| 1 | Run pt-table-checksum to identify drift |
| 2 | Prepare both master and replica by ensuring they are in sync |
| 3 | Synchronize specific table with pt-table-sync |
| 4 | Validate synchronization by re-running pt-table-checksum |

**Code:**

```
SH
Copy
# Example commands for pt-table-checksum and pt-table-sync

pt-table-checksum --master-host=master_host --master-user=replicator --master-password=replicator_password --compare-where="your_table.id > 100" --statistics --no-same-set

pt-table-sync --sync-to-master --replicate=your_table --where="column=value" --ask-pass
```

**Example input / output:**

```

Input:

SH
Copy
pt-table-checksum --master-host=master_host --master-user=replicator --master-password=replicator_password --compare-where="your_table.id > 100" --statistics --no-same-set

Output:

CODE
Copy
Checksums do not match for table 'your_schema.your_table'.

Input:

SH
Copy
pt-table-sync --sync-to-master --replicate=your_table --where="column=value" --ask-pass

Output:

CODE
Copy
Replication done.

Input:

SH
Copy
pt-table-checksum --master-host=master_host --master-user=replicator --master-password=replicator_password --compare-where="your_table.id > 100" --statistics --no-same-set

Output:

CODE
Copy
Checksums match for table 'your_schema.your_table'.
```

**Data:**

```json
{
  "master_host": "localhost",
  "replicator_user": "replicator",
  "replicator_password": "password123"
}
```

**Flow:**

1. Identify Drift
2. Prepare for Synchronization
3. RUN pt-table-sync on Replica
4. Post-Sync Validation withpt-table-checksum
5. Verify Consistency

