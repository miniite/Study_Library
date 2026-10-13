---
type: "practice"
difficulty: "Intermediate"
---

# A user reports stale reads right after a write on your primary-replica MySQL setup. Explain the chain from redo/binlog flush to replica apply lag, and how read-your-writes could be violated even with sync=1 on the primary.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ LSM Write Path WAL and Compaction](<../../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/LSM_Write_Path_WAL_and_Compaction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of stale reads right after a write in a MySQL setup involving primary-replica synchronization, we need to trace the steps from when the write is committed on the primary node all the way through to its replication and application on the replica nodes. This involves understanding how MySQL handles transaction commits, binlog flushing, and eventual consistency between primary and replicas.

The key concepts here are:

Transaction Commit and Redo Log Flushing: When a write operation occurs, it's logged in the InnoDB redo log before being flushed to disk.
Binlog Generation and Replication: The transaction is also written into the binary logs on the primary node.
Replica Synchronization: The replica nodes fetch these binlogs from the primary and apply them independently.

With sync=1 set on the primary, we assume that the transaction is immediately flushed to disk before returning control to the application. However, despite this setting, read-your-writes can still be violated due to the asynchronous nature of replication itself.

Let's break down the scenario step-by-step:

#### Triage

The question asks us to explain how a write operation on the primary node could be visible immediately (due to sync=1), yet a subsequent read on the replica might return stale data. The core issue is understanding the delay between the transaction being committed and its replication on the replica.

#### First Principles Solution

Transaction Commit: When a write occurs, it's first logged in InnoDB’s redo logs for durability. The sync=1 setting ensures that this log entry is flushed to disk immediately.

Binary Log Generation: Simultaneously, the transaction details are written into the binary logs on the primary node.

Replica Synchronization:

A replica periodically polls the primary for new binlogs via wsrep_sst or rsync.
Once a binlog is fetched and applied, it’s only then that the changes from the transaction are visible on the replica.

Latency and Asynchronous Nature: There can be significant latency between when the transaction is committed and when its effects reach the replicas due to network delays, processing time, or other factors.

Read-Your-Writes Violation: Given sync=1, a read operation right after the write on the replica would still see old data because the binlog hasn’t yet been applied. This could lead to read-your-writes violations since reads are not guaranteed to see recent writes until they are fully replicated.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, sync=1 ensures that a write is durable locally but doesn't guarantee immediate visibility on replicas due to the asynchronous nature of replication. Therefore, read operations shortly after a write might still return stale data unless explicitly handled with mechanisms like read-after-write queries or application-level retries.

#### Common mistakes
include assuming sync=1 guarantees zero latency in replication, overlooking network latencies, and not recognizing that even with high-availability setups, read-your-writes can still be violated due to the inherent delay in replicated environments.

#### Follow-ups
might include discussing how different MySQL configurations (e.g., synchronous vs asynchronous replication) affect this scenario or strategies like using gtid_mode for ensuring transactional consistency.

**Comparison:**

```
Binlog Position	Primary State	Replica State
12000	Old user records	Old user records
12345	New user record	Incomplete state
Time	Primary Operation	Replica Operation
2023-10-01T10:00	Insert into users; sync=1	Not yet applied
2023-10-01T10:05	Binlog fetched and applied	New user record visible
```

**Code:**

```sql
-- Example of a write operation with sync=1
BEGIN;
INSERT INTO users (name, email) VALUES ('John Doe', 'john.doe@example.com');
COMMIT; -- This ensures the transaction is flushed to disk immediately due to sync=1

-- Replica synchronization process
mysqlbinlog --start-datetime='2023-10-01 00:00:00' --stop-datetime='2023-10-01 01:00:00' /var/lib/mysql/binlogs/* | mysql -h replica-host
```

**Example input / output:**

```text
Primary Write Operation:
- Inserted a new user record into the database.
- Sync=1 ensured the transaction was flushed to disk immediately.

Replica Application Lag Example:
- Primary binlog position: 12345 (latest applied)
- Replica binlog position: 12000 (applied before the latest write)

Read Operation on Replica Right After Write:
- On primary: SELECT * FROM users WHERE id = 1; -> Returns new user record.
- On replica: SELECT * FROM users WHERE id = 1; -> Returns old data or incomplete state.
```

**Data:**

```json
{
  "primary_binlog_pos": 12345,
  "replica_applied_pos": 12000
}
```

**Flow:**

1. Write Operation
2. Transaction Logs to Disk
3. Binary Log Generation
4. Replica Fetches Binlog
5. Binlog Applied on Replica
6. Read Operation on Replica
7. Stale Data Possible

