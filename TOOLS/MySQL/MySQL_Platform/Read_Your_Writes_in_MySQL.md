---
id: "TOOLS/MySQL/MySQL_Platform/Read_Your_Writes_in_MySQL"
level: 4
type: "concept"
topic: "TOOLS/MySQL/MySQL_Platform"
tools: ["MySQL"]
---

# Read Your Writes in MySQL

**L4 Advanced** · [↑ MySQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** After a successful `COMMIT` the data is durable on the primary, but a replica, a cache or a second service may not show it yet — "read your writes" means designing the read path (primary reads, replica position waits, versioned caches, stable pagination) so a user never sees their own change disappear.

### 1. The big picture
A single MySQL server gives you strong guarantees: committed data is durable (the redo log and binlog are flushed) and later transactions see it. Scale-out breaks the illusion. Reads sent to asynchronous **replicas** may lag the primary by milliseconds to minutes; **caches** may hold the pre-write value; a **load balancer** may send the next request to a different node; a **TIMESTAMP** written under one session time zone is read under another. The user-visible symptoms are always the same: "I saved, the page still shows the old value", counters that jump back, rows that appear twice across pages.

This lesson brings together the pieces: what Durability and MVCC actually promise, the replica-lag strategies, cache write paths, schemas that make concurrent reads and writes safe (audit logs, drafts, dedup tables, append-only logs), and the operations discipline (backup/PITR) that decides how much you can lose.

### 2. Core concepts and vocabulary
- **Durability (the D in ACID)** — once `COMMIT` returns, the change survives a crash (given `innodb_flush_log_at_trx_commit=1`, `sync_binlog=1`).
- **Read-after-write (read-your-writes) consistency** — a client always sees its own previous writes.
- **Replica lag** — time/transactions by which a replica trails the source.
- **GTID** — global transaction identifier (`server_uuid:n`) letting you ask "has this replica applied transaction X?".
- **Sticky reads** — route a user's reads to the primary (or the same replica) for a short window after a write.
- **MVCC** — multi-version concurrency control: readers see a consistent snapshot built from undo records instead of waiting for writers.
- **Undo log** — old row versions kept to roll back and to serve snapshot reads.
- **Change buffer** — InnoDB structure that caches changes to non-unique secondary index pages not in the buffer pool.
- **Outbox / write-through / cache-aside** — patterns for keeping caches and other systems aligned with the DB.
- **RPO / RTO** — recovery point objective (data you can lose) / recovery time objective (time to recover).
- **PITR** — point-in-time recovery: restore a backup then replay binlogs to a chosen moment.

### 3. How it works, step by step
**Committing on the primary**
1. Changes are applied to buffer-pool pages and recorded in the **redo log** (and undo for rollback).
2. At `COMMIT`: redo flushed (fsync) → binlog written/fsynced (two-phase commit between engine and binlog) → transaction visible to others.
3. Replicas fetch the binlog (I/O thread into the relay log), then the applier thread(s) replay it. Visible on the replica only after apply.

**Read-your-writes strategies (choose per feature)**
1. **Read from the primary for N seconds after a write** by that user (session flag / cookie / token timestamp). Simple; costs primary capacity.
2. **Pass the write's GTID and wait on the replica:** after the write, capture `@@session.gtid_executed` (or enable `session_track_gtids=OWN_GTID`); on the read, run `SELECT WAIT_FOR_EXECUTED_GTID_SET('uuid:1-12345', 1)` on the replica — returns 0 when caught up, 1 on timeout → fall back to the primary.
3. **Version token / monotonic version column:** return the new `version` to the client; reads must satisfy `version >= token` else retry/primary.
4. **Replica lag-aware routing:** exclude replicas whose lag > threshold from the pool.
5. **Synchronous-ish replication** (semi-sync ensures receipt, not apply; Group Replication's `group_replication_consistency = BEFORE/AFTER`) when you can pay the latency.
6. **Optimistic UI + reconcile:** show the user's own change locally while the confirmation propagates.

### 4. Worked example
```sql
-- Primary: write and capture position
START TRANSACTION;
UPDATE profile SET display_name = 'Asha N.' WHERE user_id = 42;
COMMIT;
SELECT @@global.gtid_executed;   -- or read it from the OK packet with session_track_gtids

-- Replica: read-after-write with a bounded wait
SELECT WAIT_FOR_EXECUTED_GTID_SET('3E11FA47-71CA-11E1-9E33-C80AA9429562:1-77', 0.5) AS timed_out;
-- 0 → replica has applied it, safe to read; 1 → timed out, read from primary
SELECT display_name FROM profile WHERE user_id = 42;

-- Diagnose lag
SHOW REPLICA STATUS\G     -- Seconds_Behind_Source, Replica_SQL_Running_State
SELECT * FROM performance_schema.replication_applier_status_by_worker\G
```
Common production causes for lag: a large single transaction (applied serially), no parallel applier (`replica_parallel_workers` / `replica_parallel_type=LOGICAL_CLOCK`), missing primary key on a row-based-replicated table (every row update scans the table), long DDL, an I/O-saturated replica serving heavy reads. Mitigate without stopping the batch: chunk the batch into small transactions, enable parallel replication, add the missing PK, throttle the batch on lag, or send the dashboards to a dedicated replica.

**Write-path ordering for cache correctness**
```
1. Write to MySQL (commit)
2. Then invalidate (DELETE) the cache key — never "set" a value computed before the commit
3. Readers: cache miss → read primary or caught-up replica → populate with version tag
```
If step 2 fails after commit, the cache is stale until TTL; use a short TTL plus an **outbox table** (written in the same transaction as the data) and a relay that performs the invalidation at-least-once. This ensures the cache is never *ahead* of the database, only behind and eventually corrected.

**Audit log**
```sql
CREATE TABLE audit_log (
  id          BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  entity_type VARCHAR(40)  NOT NULL,
  entity_id   BIGINT UNSIGNED NOT NULL,
  actor_id    BIGINT UNSIGNED NOT NULL,
  action      ENUM('INSERT','UPDATE','DELETE') NOT NULL,
  old_values  JSON NULL,
  new_values  JSON NULL,
  occurred_at DATETIME(6) NOT NULL DEFAULT CURRENT_TIMESTAMP(6),
  KEY ix_entity (entity_type, entity_id, occurred_at)
) ENGINE=InnoDB;
```
Write it in the same transaction as the change (application or trigger) so the log can't disagree with the data; make it append-only (no `UPDATE`/`DELETE` grants).

**Time-series log table keys**
```sql
CREATE TABLE sensor_reading (
  id        BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,   -- sequential inserts
  sensor_id INT UNSIGNED NOT NULL,
  ts        DATETIME(3) NOT NULL,
  val       DOUBLE NOT NULL,
  KEY ix_sensor_ts (sensor_id, ts)
);
```
Sequential PK keeps inserts at the right edge of the clustered index; the secondary index serves per-sensor time ranges. If reads are *always* by `(sensor_id, ts)`, clustering on that composite (`PRIMARY KEY (sensor_id, ts, id)`) trades insert locality (writes spread across sensors) for read locality.

**Deduplicating replayed messages:** `UNIQUE (source, message_id)` + `INSERT … ON DUPLICATE KEY UPDATE id = id` (or `INSERT IGNORE` when you accept silent skip), applied in the same transaction as the side effect.

**Safe purge:**
```sql
-- loop until ROW_COUNT() = 0, sleep between batches
DELETE FROM event WHERE created_at < NOW() - INTERVAL 90 DAY ORDER BY id LIMIT 5000;
```
Index `created_at`; small batches keep locks short and replicas current. For tables purged by time, partition by range and `DROP PARTITION` (instant).

**Draft then publish:** keep `article` (published, immutable-ish) and `article_draft` (or a `status` + `published_version_id`); publishing is one short transaction that points readers to the new version, so readers never see a half-edited row.

**Stable paging under concurrent inserts:** keyset on `(created_at, id)` with an upper bound captured on the first request; readers get a consistent window and no duplicates.

### 5. Under the hood
- **MVCC in InnoDB.** Each clustered-index record has hidden `DB_TRX_ID` (last modifying transaction) and `DB_ROLL_PTR` (pointer into the undo log). A **read view** records which transaction IDs were active when the snapshot was taken. A non-locking `SELECT` follows the undo chain until it finds a version visible to its read view. Under `REPEATABLE READ` the read view is created at the first read in the transaction; under `READ COMMITTED` a new one per statement. Readers take no row locks, so they don't block writers and vice versa.
- **Undo and purge.** Old versions are retained until no read view needs them; the purge thread then frees them. A long-open transaction (or an idle connection in `START TRANSACTION`) pins history → undo growth, slower reads walking long chains, and `History list length` rising in `SHOW ENGINE INNODB STATUS`.
- **Durability knobs.** `innodb_flush_log_at_trx_commit`: `1` fsync each commit (durable), `2` write to OS cache each commit and fsync per second (lose up to ~1 s on OS crash), `0` flush per second (lose ~1 s on mysqld crash). `sync_binlog=1` fsyncs the binlog each commit. The doublewrite buffer protects against torn pages.
- **Change buffer.** For non-unique secondary indexes whose leaf page isn't in memory, InnoDB buffers the change and merges it later (on read or by background merge), converting random I/O into sequential. `innodb_change_buffering` controls what's buffered; it helps write-heavy workloads on spinning disks and large indexes, matters less on fast NVMe or when the working set is in memory, and it makes recovery/merges more work — disable it (`none`) if you see persistent merge stalls or when indexes are mostly unique.
- **TIMESTAMP zone behaviour.** Stored as UTC; converted by session `time_zone` on write/read. Two app servers with different OS zones writing the *same literal* produce different instants. Standardize: DB server and connection zone `+00:00` (or set `time_zone` per connection) and convert in the presentation layer; DST changes are a non-issue for UTC instants but break local-time arithmetic.
- **Auto-increment under `ON DUPLICATE KEY UPDATE`.** InnoDB reserves auto-increment values before discovering the duplicate, so `INSERT … ODKU` burns IDs (gaps) — harmless unless the column is a small type.

### 6. Performance and design implications
- **Autosave writes:** coalesce on the client (debounce 2–5 s), send only changed fields, write the latest draft with `INSERT … ON DUPLICATE KEY UPDATE` keyed by `(doc_id, user_id)` — one row, not one row per keystroke; or buffer in Redis and flush.
- **"Mark all as read":** update by a high-water mark (`last_read_at` / `last_read_id` on the user) instead of touching thousands of notification rows; compute unread as `id > last_read_id`.
- **Counters flicker:** make reads monotonic by sourcing the displayed value from one place (primary or a versioned cache) and tolerating only increases client-side.
- **Cross-service transactions:** don't read your own write across a service boundary assuming it's committed; return data from the write call itself, or publish after commit via outbox.
- **Replicas are not a free scaling lever** for read-your-writes-sensitive pages; budget primary capacity for them.
- **Backups:** logical (`mysqldump --single-transaction --source-data`) or physical (Percona XtraBackup / MySQL Enterprise Backup / cloud snapshots) plus **binary logs** shipped off-box for PITR.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `WAIT_FOR_EXECUTED_GTID_SET(set, timeout)` | Block until replica applied GTIDs | Read-after-write on replicas |
| `session_track_gtids=OWN_GTID` | Return the write's GTID to the client | Passing tokens |
| `SHOW REPLICA STATUS`, `performance_schema.replication_*` | Lag and applier state | Lag diagnosis |
| `replica_parallel_workers`, `replica_parallel_type=LOGICAL_CLOCK` | Parallel apply | Reduce lag |
| `innodb_flush_log_at_trx_commit`, `sync_binlog` | Durability vs speed | Policy by data class |
| `innodb_change_buffering` | Control change buffer | Tuning I/O patterns |
| `mysqldump --single-transaction`, XtraBackup, `mysqlbinlog --start-position/--stop-datetime` | Backup and PITR | Recovery runbooks |
| `SET time_zone = '+00:00'` | Session zone | Deterministic TIMESTAMPs |
| `INSERT … ON DUPLICATE KEY UPDATE` | Atomic upsert | Counters, dedup, autosave |
| `semi-sync`, `group_replication_consistency` | Stronger replication guarantees | Lower RPO / consistent reads |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Primary-read window after writes | Write-heavy and primary already hot |
| GTID wait on replica | Clients can't carry tokens; waits add tail latency |
| Cache invalidate-after-commit + outbox | Strict consistency needed (read primary) |
| `ON DUPLICATE KEY UPDATE` upsert | Multiple unique keys; read-then-write is racy and slower |
| Keyset pagination | Random page jumps required |
| `innodb_flush_log_at_trx_commit=2` | Financial or irreplaceable data |
| Partition drop for retention | Queries lacking partition key (all partitions scanned) |

### 9. Common mistakes
- **Reading from a replica right after writing** → stale page → sticky reads/GTID waits.
- **Setting cache before commit** → cache ahead of DB, ghost data on rollback → invalidate after commit.
- **`SELECT` then `INSERT` for upsert** → race, duplicates → unique key + ODKU.
- **Big single-transaction batch** → replica lag spikes → chunk.
- **Leaving transactions open (idle in transaction)** → undo bloat → timeouts, short transactions.
- **Replication on a table without a primary key** (row-based) → each row change scans → add PK.
- **Mismatched time zones between app servers** → inconsistent `TIMESTAMP`s → standardize on UTC.
- **Untested backups** → restore fails during an incident → scheduled restore drills with measured RTO.
- **Assuming `OFFSET` pages stay stable** under inserts → duplicates/skips → keyset.

### 10. 🎯 Interview kit
**30-second answer:** `COMMIT` is durable on the primary, but replicas apply asynchronously. For read-your-writes I route the user's next reads to the primary for a short window, or capture the write's GTID and have the replica `WAIT_FOR_EXECUTED_GTID_SET`, with fallback to the primary on timeout; caches are invalidated after commit using an outbox so they're never ahead of the DB.

**Structure a longer answer:** guarantees on one node (durability, MVCC) → what breaks with replicas/caches → strategy options and trade-offs → implementation details (GTID, tokens, routing) → monitoring lag → recovery planning.

**Likely follow-ups:**
- *How does MVCC let readers avoid blocking writers?* → snapshot reads via undo chains and read views.
- *What are undo logs for?* → rollback and old versions for snapshots; purged when no longer needed.
- *What does Durability guarantee?* → committed changes survive crashes, subject to flush settings.
- *Diagnose 40 s lag during nightly batch?* → find big transactions, parallel apply, missing PKs; chunk and throttle.
- *Design backup/PITR?* → periodic full backup + continuous binlog shipping, restore drills, measured RTO/RPO.
- *ODKU vs read-then-write?* → atomic and race-free vs racy; note auto-increment gaps.

**Signals interviewers look for:** you separate "committed" from "visible everywhere", you pick a strategy with a fallback, and you talk about measurement and recovery drills.

### 11. Check yourself
- [ ] Explain exactly why a user can fail to see their write on a replica.
- [ ] Implement a GTID-based read-after-write flow including the timeout fallback.
- [ ] Describe read views and the undo chain for a `REPEATABLE READ` snapshot.
- [ ] Choose `innodb_flush_log_at_trx_commit` values for three data classes and justify.
- [ ] Design a PITR procedure and how you'd prove RPO/RTO.
- [ ] Convert a 90-day purge into a safe repeating job.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../_index.md>)

## Practice questions

- [Basic] [What does the 'D' for Durability in ACID guarantee after a COMMIT?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33426_What_does_the_D_for_Durability_in_ACID_guarantee_after_a_COM.md>)
- [Intermediate] [A form submits and the API returns 200 but the row sometimes isn't visible on the next page load — how do you debug this write-then-read-replica consistency gap end to end?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33261_A_form_submits_and_the_API_returns_200_but_the_row_sometimes.md>)
- [Deep] [Your endpoint sometimes returns a row that another transaction just deleted — how do you reason about whether READ COMMITTED vs REPEATABLE READ is the right fix?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33444_Your_endpoint_sometimes_returns_a_row_that_another_transacti.md>)
- [Deep] [How do you decide when to move analytics off the OLTP primary onto a replica or warehouse, and what query patterns are the signal to do so?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33550_How_do_you_decide_when_to_move_analytics_off_the_OLTP_primar.md>)
- [Deep] [What is the default isolation level in InnoDB and how does REPEATABLE READ work there?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33225_What_is_the_default_isolation_level_in_InnoDB_and_how_does_R.md>)
- [Deep] [How would you design deduplication for an ingestion pipeline writing events to MySQL where the source occasionally replays messages?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33167_How_would_you_design_deduplication_for_an_ingestion_pipeline.md>)
- [Expert] [How does MVCC in InnoDB let readers avoid blocking writers, and what are undo logs?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33084_How_does_MVCC_in_InnoDB_let_readers_avoid_blocking_writers_a.md>)
- [Expert] [How do you design and test a backup/restore + point-in-time-recovery strategy for MySQL, and how do you validate RTO/RPO before a real incident?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33159_How_do_you_design_and_test_a_backup_restore_point-in-time-re.md>)

## Related concepts

- [Replica Lag and Read Your Writes](<../../PostgreSQL/PostgreSQL_Platform/Replica_Lag_and_Read_Your_Writes.md>) *(PostgreSQL Platform)*
- [Distributed Counters and Consistency](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Distributed_Counters_and_Consistency.md>) *(Distributed Systems Core)*
- [Read Your Writes with Replicas](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Read_Your_Writes_with_Replicas.md>) *(Cloud Fundamentals)*

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Problems with Replication Lag › Reading Your Own Writes
- *Designing Data-Intensive Applications* — Martin Kleppmann — Detecting Concurrent Writes › Capturing the happens-before relationship
- *Data Pipelines Pocket Reference* — James Densmore — Data Ingestion Extracting Data › Binary Log Replication of MySQL Data
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Risk Management › Evaluation
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Risk Management › Implementation
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Managing Data with Delta Lake › Concurrent writes and reads scenario
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Conclusion › T

