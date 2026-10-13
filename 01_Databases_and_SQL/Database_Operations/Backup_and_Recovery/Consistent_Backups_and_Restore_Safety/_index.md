---
id: "01_Databases_and_SQL/Database_Operations/Backup_and_Recovery/Consistent_Backups_and_Restore_Safety"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Operations/Backup_and_Recovery"
tools: ["AWS", "MySQL", "Oracle"]
---

# Consistent Backups and Restore Safety

**L4 Advanced** · [↑ Backup and Recovery](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A backup is only safe to restore if it captures a transactionally consistent state; snapshots and file copies taken without the engine's cooperation can look fine and still restore into a corrupt database.

### 1. The big picture
Databases keep data in two places at once: durable files and in-memory buffers, with a log that makes sense of the difference. A backup has to capture a state that the engine can recover from. Engine-native backups do that on purpose. Storage or VM snapshots, plain file copies, and replication-lag copies may not, which is why teams are often surprised when a "protected" VM restores into a database that will not start or has damaged data.

This lesson covers: what "consistent" means, backup temperature (hot/warm/cold), crash-consistent vs application-consistent snapshots, how logs enable point-in-time recovery (PITR), recovery models and what they cost, and the safety rules around the restore itself (not restoring over production, authorisation, keeping the chain intact).

### 2. Core concepts and vocabulary
- **Transactionally consistent** — contains every committed change up to a point and none of the uncommitted ones (after recovery runs).
- **Crash-consistent snapshot** — a point-in-time copy of all blocks with write order preserved, equal to what a power cut would leave on disk. The engine must run crash recovery on restore.
- **Application-consistent snapshot** — taken after coordinating with the database (flush/quiesce, VSS-style writers, or a backup mode), so no recovery surprises.
- **Hot / warm / cold backup** — taken while fully online (reads and writes), with writes paused briefly, or with the database shut down.
- **Full / differential / incremental / log backup** — everything; changes since last full; changes since last backup of any kind; the transaction log stream.
- **WAL / transaction log / redo log / binlog / oplog** — the sequential record of changes that supports durability and PITR; the formats and retention differ by engine.
- **Recovery model** (SQL Server) — `SIMPLE`, `FULL`, `BULK_LOGGED`: how much is logged and whether log backups are possible.
- **PITR** — restore a base backup then replay logs to a chosen moment.
- **Physical vs logical backup** — copies of storage blocks/files vs exported schema and rows (SQL dumps). Logical is portable across versions; physical is faster to restore at scale.
- **Media recovery vs crash recovery** — restore from backup after a lost device vs automatic redo/undo after an abnormal stop.

### 3. How it works, step by step
**What the engine does to make an online backup consistent**
1. Backup starts; the engine records the log position (e.g. LSN) at the start.
2. It copies data pages while normal activity continues, so the copy is fuzzy: different pages reflect different moments.
3. It then captures the log generated during the copy.
4. On restore, data pages are written back, then **recovery** rolls forward committed work from that log and rolls back what was uncommitted. The result is consistent as of the end of the backup.

**Why a crash-consistent snapshot can fail**
1. Snapshot freezes the volume at an instant.
2. At that instant a transaction may have updated pages in memory, written the log, and flushed only some pages to the data file.
3. On restore, the engine runs crash recovery using the log. If *data and log are on separate volumes snapshotted at different instants*, or the snapshot didn't preserve write ordering across them, the log and data disagree and recovery can fail or leave damage.
4. Even when it works, a crash-consistent snapshot cannot truncate or back up the log, so PITR is lost.

**Choosing the method**

| Method | Consistency | Downtime | PITR | Notes |
|---|---|---|---|---|
| Engine full + log backups | Transactionally consistent | None | Yes | Gold standard |
| Application-consistent snapshot (quiesced) | Consistent | Seconds of pause | Only with logs | Good for large volumes |
| Crash-consistent snapshot | Recoverable at best | None | No | Great for fast VM rollback, weak as only protection |
| Cold file copy | Consistent | Full outage | No | Simple, small systems |
| Logical dump | Consistent snapshot per transaction/settings | None-ish | No (unless with logs) | Portable, slow to restore |

### 4. Worked example
```sql
-- SQL Server: nightly full + differential, 15-minute log backups
BACKUP DATABASE Sales TO DISK = N'N:\bak\Sales_full.bak'  WITH CHECKSUM, COMPRESSION, INIT;
BACKUP DATABASE Sales TO DISK = N'N:\bak\Sales_diff.bak'  WITH DIFFERENTIAL, CHECKSUM, COMPRESSION;
BACKUP LOG      Sales TO DISK = N'N:\bak\Sales_log_1015.trn' WITH CHECKSUM;

-- Point-in-time restore to just before a bad DELETE at 10:42:30
RESTORE DATABASE Sales_pitr FROM DISK = N'N:\bak\Sales_full.bak'
  WITH NORECOVERY, MOVE 'Sales' TO 'T:\r\Sales.mdf', MOVE 'Sales_log' TO 'T:\r\Sales.ldf';
RESTORE DATABASE Sales_pitr FROM DISK = N'N:\bak\Sales_diff.bak' WITH NORECOVERY;
RESTORE LOG Sales_pitr FROM DISK = N'N:\bak\Sales_log_1030.trn' WITH NORECOVERY;
RESTORE LOG Sales_pitr FROM DISK = N'N:\bak\Sales_log_1045.trn'
  WITH STOPAT = '2026-10-12T10:42:29', RECOVERY;
```
Walk-through: `NORECOVERY` keeps the database in "restoring" so the chain can continue; the final step with `RECOVERY` runs recovery and opens it. `STOPAT` replays log up to just before the mistake. Restoring under a *new name* (`Sales_pitr`) lets you extract the lost rows and copy them back, leaving production running. A missing log file breaks the chain: you can only restore to the point before the gap.

In MySQL the same idea uses a full backup plus binary logs replayed with `mysqlbinlog --stop-datetime`; a missing or corrupt binlog means you stop at the last good file and lose everything after.

### 5. Under the hood   *(optional at this level)*
- **WAL rule:** a changed page may not be written to disk before its log record. That is what makes crash recovery possible and why log + data must be recovered together.
- **Recovery phases:** analysis, redo (repeat history), undo (roll back losers). A restored crash-consistent image needs all three.
- **Recovery model effect on logging:** under `FULL` every allocation and change is logged in detail so any point can be restored; `SIMPLE` logs just enough for rollback and crash recovery, and the log is truncated automatically (so no log backups, no PITR). The detail difference can be large; page-level versus extent-level allocation logging is one example, and bulk operations in `BULK_LOGGED` log less still.
- **Cloud managed databases** do automated continuous backups: for example Azure SQL keeps PITR for a configurable window (up to 35 days) and long-term retention up to ten years, with geo-redundant copies. You still own *testing* and *retention policy*.
- **Heterogeneous engines:** WAL, binlog, redo, and oplog differ in consistency and retention semantics, so a "standard PITR" has to be defined per engine, not once.

### 6. Performance and design implications
- Full backups read the whole database; differential/log backups shrink backup size and the data-loss window at the cost of a longer restore chain.
- Frequent log backups are cheap individually but add up in storage and chain length; retention must keep the chain from the last full to the target point.
- `FULL` recovery on a database refreshed once a night by ETL inflates the log and slows loads; if the effective RPO is the nightly load, `SIMPLE` plus a post-load full backup is often the correct choice.
- Backups compete with production for IO and network: schedule them off-peak, throttle, or back up from a replica/secondary.
- Compression and deduplication trade CPU (and slower first backup) for storage; already-compressed data compresses poorly.
- Object storage backups are cheap and durable but restore speed is bounded by network and retrieval; block-level/snapshot restores are faster.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `BACKUP LOG … / RESTORE LOG … WITH STOPAT` | Log backup and PITR | Any `FULL` recovery database |
| `ALTER DATABASE … SET RECOVERY SIMPLE/FULL/BULK_LOGGED` | Chooses logging and PITR capability | Match to RPO, not habit |
| `RESTORE … WITH NORECOVERY / RECOVERY / STANDBY` | Controls whether the chain continues | Multi-step restores |
| `BACKUP … COPY_ONLY` | Ad-hoc backup that doesn't disturb the chain | Dev refresh |
| `pg_basebackup` + WAL archive / `mysqldump --single-transaction` / binlogs | Base + log for PITR in PostgreSQL/MySQL | Open-source engines |
| Storage snapshot with a quiesce hook | Application-consistent snapshot | Large volumes |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Engine-native full + log backups | Database is so large that restore time breaks RTO; add snapshots/replicas |
| Application-consistent snapshots | The quiesce integration isn't tested |
| Crash-consistent snapshots | They are your *only* protection for a database |
| `SIMPLE` recovery | You need to recover to any point in the day |
| `FULL` recovery | Nobody takes log backups; the log grows until the disk fills |
| Logical dumps | You need fast restore of terabytes |

### 9. Common mistakes
- VM-level crash-consistent snapshots as the sole backup → restore reveals corruption, no PITR → use engine backups or app-consistent snapshots, and test.
- `FULL` recovery with no log backups → log fills the volume → schedule log backups or switch to `SIMPLE`.
- Nightly full only, "lost 6 hours of data after a crash" → RPO is 24 h by design → add log backups sized to the RPO.
- Restoring over the wrong database (backing up dev and restoring it onto production) → outage and data loss → restore to a new name, require two-person checks, use distinct credentials/naming.
- Breaking the chain (deleted log, lost full) → can't reach target point → monitor chain completeness and protect the catalog metadata.
- Forgetting logins, jobs, sequences, grants, extensions → data restores but the app doesn't work → back up and test the whole recovery set, not just user tables.
- Copying files of a running database → inconsistent copy → use engine tooling.

### 10. 🎯 Interview kit
**30-second answer:** "A consistent backup is one the engine can recover to a transaction boundary. I use engine full plus log backups for PITR, application-consistent snapshots for big volumes, and never rely on crash-consistent snapshots alone. I choose recovery model from RPO and test restores, including a restore to a new name so production is never overwritten."

**Structure a longer answer:** clarify (engine, RPO/RTO, size) → mechanism (log + data, recovery phases) → options (hot/warm/cold, snapshots, logical) → trade-offs (restore time, IO, cost) → scenario (bad DELETE at 10:42).

**Likely follow-ups:**
- *Hot vs warm vs cold?* → Online / writes briefly paused / database stopped; hot needs the log to be consistent.
- *Crash recovery vs media recovery?* → Automatic redo/undo after an abnormal stop vs restore from backup plus logs after losing storage.
- *What if a log is missing or corrupt?* → Chain breaks; restore only to before the gap; recover the rest from replicas or source systems.
- *Differential vs incremental?* → Since last full vs since last backup; differential restores need fewer files, incremental backups are smaller.
- *WAL's role in durability?* → Log is flushed at commit before data pages, enabling redo after a crash.
- *Back up Redis/in-memory stores?* → Use persistence (snapshots/AOF) and replicas; restore semantics are weaker, so state your RPO.

**Signals interviewers look for:** distinguishing crash- from application-consistency, linking RPO to recovery model, awareness that snapshots do not give PITR, restore-safety habits.

### 11. Check yourself
- [ ] Explain why a crash-consistent snapshot of a database VM can restore badly.
- [ ] Draw the restore sequence for full + differential + logs with `STOPAT`.
- [ ] Decide `SIMPLE` vs `FULL` for a nightly-loaded warehouse and justify it.
- [ ] Describe how you would prevent restoring a backup over production.
- [ ] Explain physical vs logical backups and when each wins.
<!-- explainer:end -->

## Go deeper

- [Backup Retention and Restore at Scale](<Backup_Retention_and_Restore_at_Scale.md>)

## Implementations (tools)

- [AWS](<../../../../TOOLS/AWS/_index.md>)
- [MySQL](<../../../../TOOLS/MySQL/_index.md>)
- [Oracle](<../../../../TOOLS/Oracle/_index.md>)

## Practice questions

- [Basic] [What is the difference between a hot backup, a warm backup, and a cold backup?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Backup_and_Recovery/Q-04029_What_is_the_difference_between_a_hot_backup_a_warm_backup_an.md>)
- [Intermediate] [What is the difference between backing up to object storage versus block storage, and how does it affect restore speed?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Backup_and_Recovery/Q-04154_What_is_the_difference_between_backing_up_to_object_storage_.md>)
- [Deep] [How do you handle backup and recovery for very large tables that are partitioned by date?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Backup_and_Recovery/Q-04110_How_do_you_handle_backup_and_recovery_for_very_large_tables_.md>)
- [Deep] [Explain the role of the write-ahead log (WAL) in crash recovery and how it guarantees durability.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Backup_and_Recovery/Q-04129_Explain_the_role_of_the_write-ahead_log_WAL_in_crash_recover.md>)
- [Deep] [How do you back up and recover in-memory or ephemeral database components like Redis when persistence is enabled?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Backup_and_Recovery/Q-04149_How_do_you_back_up_and_recover_in-memory_or_ephemeral_databa.md>)
- [Deep] [Explain how MySQL binary logs are used for point-in-time recovery and what happens if a log is missing or corrupt.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Backup_and_Recovery/Q-04173_Explain_how_MySQL_binary_logs_are_used_for_point-in-time_rec.md>)
- [Deep] [How would you architect backups for a system where the database and a search index like Elasticsearch must be recovered to a mutually consistent state after a failure?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Backup_and_Recovery/Q-04053_How_would_you_architect_backups_for_a_system_where_the_datab.md>)
- [Expert] [Explain how you would recover a database to a specific transaction rather than a specific timestamp.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Backup_and_Recovery/Q-04150_Explain_how_you_would_recover_a_database_to_a_specific_trans.md>)

## Related concepts

- [Separating Compute and Storage](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Separating_Compute_and_Storage.md>) *(Cloud Fundamentals)*
- [HA Failover Design](<../../High_Availability/HA_Failover_Design.md>) *(High Availability)*
- [Read Your Writes with Replicas](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Read_Your_Writes_with_Replicas.md>) *(Cloud Fundamentals)*
- [Batching Writes](<../../../Database_Internals/DBMS_Concepts/Batching_Writes.md>) *(DBMS Concepts)*
- [ACID Durability and Consistency](<../../../Transactions_and_Concurrency/ACID_Transactions/ACID_Durability_and_Consistency.md>) *(ACID Transactions)*
- [LSM Write Path WAL and Compaction](<../../../Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/LSM_Write_Path_WAL_and_Compaction.md>) *(Storage Engines)*

## References

- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Backups › 12.2 #79 Using database snapshots as a recovery strategy
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Backups › 12.3 #80 Using crash-consistent snapshots as a recovery strategy
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Backups › 12.6 #83 Always using the FULL recovery model on data warehouse and development systems
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Backups › 12.7 #84 Using SIMPLE recovery model for OLTP databases
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Backup and Restore in Azure SQL
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Introducing Database Reliability Engineering › Survival and Safety
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Backup and Recovery › Online versus Offline
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Backup and Recovery › Full, Incremental, and Differential
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Backup and Recovery › Incremental physical backups
- *Database Internals* — Alex Petrov — Introduction and Overview › Durability in Memory-Based Stores
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › Recovery
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › ARIES
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Data durability and availability
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Data retention
- *Think Distributed Systems* — Dominik Tornow — Transactions › 5.3.3 Completeness

