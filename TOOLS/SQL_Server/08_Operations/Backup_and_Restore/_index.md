---
id: "TOOLS/SQL_Server/08_Operations/Backup_and_Restore"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Backup and Restore

**L4 Advanced** · [↑ Operations and Administration](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A backup strategy is built backwards from the business's RPO and RTO: choose the recovery model, schedule full/differential/log backups so the data-loss window is short enough, store them where you can retrieve them fast and safely, and prove they restore.

### 1. The big picture
Everything else in operations (HA, security, performance) can fail; backups are the last line of defence. A backup is only valuable if it can be restored within the time the business tolerates, to a point that loses no more data than it tolerates. So design starts from two numbers:

- **RPO (Recovery Point Objective)**: the maximum acceptable data loss, measured in time. It dictates how often you take *transaction log* backups.
- **RTO (Recovery Time Objective)**: the maximum acceptable downtime. It dictates *where* backups live, how quickly you can fetch them, and how many files must be restored in sequence.

A typical schedule is a nightly full, a daytime differential, and log backups at the RPO interval. The common failures are not exotic: wrong recovery model, untested backups, an ad-hoc backup that breaks the restore chain, backups that live on the same file share an attacker can encrypt, and HA mistaken for DR.

### 2. Core concepts and vocabulary
- **Full backup** — a complete copy of the data plus enough log to be transactionally consistent. The base of every restore sequence.
- **Differential backup** — everything changed since the last *full* (the "differential base"). Each differential is cumulative.
- **Transaction log backup** — the log records since the previous log backup; enables point-in-time restore and lets the log be truncated (in `FULL`/`BULK_LOGGED`).
- **Recovery model** — `SIMPLE`, `BULK_LOGGED`, `FULL`.
- **Log chain** — the unbroken sequence of log backups from a full backup; a gap breaks point-in-time restore.
- **Differential base** — the most recent full backup that differentials are measured from.
- **`COPY_ONLY`** — a backup option that does not alter the differential base or log chain.
- **Point-in-time restore** — `RESTORE LOG … WITH STOPAT`.
- **Tail-log backup** — a final log backup taken just before a restore, to capture the transactions since the last scheduled log backup.
- **`RESTORE VERIFYONLY` / `CHECKSUM`** — integrity checks of the backup media (not a proof of restorability).
- **Air gap / immutability** — copies the production environment (and an attacker) cannot modify.

### 3. How it works, step by step
**Choosing the recovery model**

| Model | Backups possible | Log truncation | Restore granularity |
|---|---|---|---|
| `SIMPLE` | Full, differential (no log backups) | At checkpoint | To the end of a backup |
| `BULK_LOGGED` | Full, differential, log | After log backup | To the end of a backup (point-in-time not possible within a bulk-logged log backup) |
| `FULL` | Full, differential, log | After log backup | To any point in time |

In `FULL`, more is logged (for example individual page allocations); in `SIMPLE`, the log only needs to support rollback and crash recovery, so far less is written. A nightly-ETL data warehouse has an effective RPO of 24 hours (rerun the load), so `SIMPLE` is often right; an OLTP system is the opposite. Note HA/DR features (availability groups, log shipping) and transactional replication all require `FULL`.

**Designing the schedule**
1. Write down RPO and RTO per database.
2. Set the log backup interval **to the RPO** (RPO 1 hour → hourly log backups; often 5–15 minutes in practice).
3. Add a nightly full and optional differentials to shorten the restore chain (RTO).
4. Choose the storage path: local disk first (fast restore), then offload to tape, object storage or a backup tool.
5. Add integrity: `WITH CHECKSUM`, regular `RESTORE VERIFYONLY`, scheduled *real* test restores and `DBCC CHECKDB`.
6. Add security: encryption, offsite/immutable copy.

**Restoring**
1. Take a tail-log backup if the database is still readable.
2. Restore the latest full `WITH NORECOVERY`.
3. Restore the latest differential `WITH NORECOVERY` (if any).
4. Restore each log backup in order `WITH NORECOVERY`, the last one `WITH STOPAT` (if point-in-time) and `RECOVERY`.

### 4. Worked example
Why you must take a full backup after switching `SIMPLE` → `FULL`:

```sql
CREATE DATABASE PromotionDB;
ALTER DATABASE PromotionDB SET RECOVERY SIMPLE;
BACKUP DATABASE PromotionDB TO DISK = 'D:\Backups\PromotionDB_simple.bak';

ALTER DATABASE PromotionDB SET RECOVERY FULL;   -- "promoted to production"

BACKUP LOG PromotionDB TO DISK = 'D:\Backups\PromotionDB.trn';
-- Msg 4214: BACKUP LOG cannot be performed because there is no current database backup.
```

The database *behaves as SIMPLE* until a new full backup initializes the log chain. If the next scheduled full backup is at midnight and log backups run every 30 minutes from 2 p.m., you get 19 failed log backups and an RPO of up to 10 hours.

And the ad-hoc backup problem:

```sql
-- Schedule: Full 01:00, Diff 06:00 / 12:00 / 18:00, Log hourly
-- At 10:00 someone takes a plain full backup before a change, then deletes it.
-- The 12:00 and 18:00 differentials are now based on the deleted 10:00 full: unrestorable.

BACKUP DATABASE Marketing TO DISK = 'D:\Backups\Marketing_adhoc.bak'
WITH COPY_ONLY, CHECKSUM, COMPRESSION;
```

`COPY_ONLY` leaves the differential base alone. The author even recommends using it for *all* ad-hoc backups.

Finding at-risk databases:

```sql
SELECT d.name, d.recovery_model_desc,
       MAX(CASE WHEN b.type='D' THEN b.backup_finish_date END) AS last_full,
       MAX(CASE WHEN b.type='I' THEN b.backup_finish_date END) AS last_diff,
       MAX(CASE WHEN b.type='L' THEN b.backup_finish_date END) AS last_log
FROM sys.databases d
LEFT JOIN msdb.dbo.backupset b ON b.database_name = d.name
WHERE d.database_id > 4
GROUP BY d.name, d.recovery_model_desc;
```

### 5. Under the hood
- **Full backup** reads allocated pages, then includes the log generated *during* the backup so the restored copy can be rolled forward/back to a consistent point.
- **Differential** uses the **differential change map** (DCM) pages: one bit per extent changed since the last full. This is why it is cheap to determine *what* to copy, and why a new full resets the base.
- **Log backup** copies log records from the last log backup LSN to the current one and then allows VLFs to be marked reusable. If the log isn't backed up in `FULL`, the file just keeps growing. Transaction log backups are small: the log is typically a fraction of data size and, the more often you back it up, the less each backup contains.
- **Bulk-logged** minimally logs certain bulk operations; the log backup then needs the data extents changed by those operations, so point-in-time restore *within* that interval isn't possible.
- **Backup history** lives in `msdb` (`backupset`, `backupmediafamily`); `RESTORE HEADERONLY / FILELISTONLY` read the header of a file.
- **Encryption** needs a database master key and a certificate in `master`; **back up the certificate and private key separately**, or the encrypted backups cannot be restored on another server.
- **Cloud / managed**: Azure SQL Database takes automated backups (full weekly, differential and log frequently), supports point-in-time restore within 7–35 days, geo-redundant storage, and long-term retention up to 10 years; no manual backup jobs.

### 6. Performance and design implications
- **Full backups** are heavy I/O; schedule in quiet windows and use compression (`COMPRESSION`), multiple files/striping (`TO DISK=…, DISK=…`) and tuned `BUFFERCOUNT`/`MAXTRANSFERSIZE` for large databases.
- **Log backups** are light; if they hurt, back up more often, size the server for RPO, or on availability groups back up on a secondary.
- **RTO** is dominated by retrieval and restore speed: backups on tape or in archive tiers can take hours or a day to retrieve; a backup team's queue adds delay. When RTO is tight, take native backups to local disk on a short retention (for example 3 days, cleaned by a scheduled script) and let the enterprise tool copy them off. You then own success monitoring and the extra disk cost.
- **`SIMPLE` for non-critical/ETL databases** reduces log volume and can speed up nightly loads.
- **Restore time grows with chain length**: more differentials reduce the number of log files to apply.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `BACKUP DATABASE … TO DISK` | Full backup | Nightly/baseline |
| `… WITH DIFFERENTIAL` | Changes since last full | Shorten restore chain |
| `BACKUP LOG` | Log backup | At RPO interval; also tail-log |
| `WITH COPY_ONLY` | Doesn't affect base or chain | Ad-hoc / refresh dev |
| `WITH CHECKSUM, COMPRESSION` | Page checksum verification, smaller files | Default for all backups |
| `RESTORE … WITH NORECOVERY / STANDBY / RECOVERY` | Control chain state | Multi-step restores |
| `RESTORE LOG … WITH STOPAT / STOPATMARK` | Point-in-time | "Undo" a bad delete |
| `RESTORE VERIFYONLY` | Checks media readability | Cheap sanity check, not a substitute for restore tests |
| `ALTER DATABASE … SET RECOVERY …` | Change recovery model | Followed by a full backup when moving from SIMPLE |
| `BACKUP CERTIFICATE` | Export the encryption certificate | Immediately after creating backup encryption |
| `sys.sp_cycle_errorlog`, `msdb.dbo.backupset` | Operational history | Compliance and monitoring |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `FULL` + log backups: OLTP, needs PITR, HA/DR, replication | Large ETL/warehouse where RPO = last load and logging cost hurts |
| `SIMPLE`: warehouse, dev/test, rebuildable data | Any database where losing a day's data is unacceptable |
| Differentials: large DB, long chain of logs | Many tiny databases: more files, more complexity |
| Native backups to local disk + offload | Enterprise tool only, with slow retrieval SLAs and no one to verify |
| Immutable/air-gapped cloud copy (time-based retention) | Backups on a writable file share reachable from the SQL host |

### 9. Common mistakes
- **No one considers RPO/RTO** → schedule is arbitrary → derive log interval and storage from the objectives.
- **Never testing restores** → corrupt/incomplete backup found during the outage → check job success, use `CHECKSUM` and `VERIFYONLY`, and restore to a test server regularly.
- **`FULL` model with no log backups** → log fills the disk → schedule log backups or choose `SIMPLE` deliberately.
- **`SIMPLE` for OLTP production** → RPO of a day, no AG/log shipping/replication → use `FULL`.
- **Switching to `FULL` without a full backup** → log backups fail (error 4214) → take a full backup immediately.
- **Log backup directly after full backup only** → looks like PITR but doesn't help in a disk-loss scenario → schedule logs at the RPO interval.
- **Ad-hoc backup without `COPY_ONLY`** → resets the differential base and breaks the usual sequence → use `COPY_ONLY`.
- **Backups on an attackable file share** → ransomware encrypts them too → offload to immutable storage and encrypt; back up the certificate.
- **Relying on HA** → no protection from logical corruption or deletes → keep backups.

### 10. 🎯 Interview kit
**30-second answer:** I derive the strategy from RPO and RTO. Log backups at the RPO interval, a nightly full plus differentials to shorten the restore chain, the recovery model chosen per database (`FULL` for OLTP, `SIMPLE` for rebuildable warehouses), backups stored where retrieval meets RTO and protected against ransomware, and regular restore tests.

**Structure a longer answer:** objectives → recovery model → backup types and schedule → storage and security → verification (checksum, verify, test restore, CHECKDB) → restore procedure (tail-log, full → diff → logs, STOPAT) → close with an incident story.

**Likely follow-ups:**
- *Why can't you take a log backup after switching to FULL?* → The chain isn't initialized until a full backup exists.
- *What does COPY_ONLY do?* → Doesn't reset the differential base or affect the log chain.
- *Differential vs. log?* → Differential: changed extents since the last full, cumulative, speeds restores; log: transaction records, enables PITR.
- *How do you restore to 9:55 a.m.?* → Tail-log, full, latest diff before it, logs up to it, `STOPAT '09:55'`.
- *Is `RESTORE VERIFYONLY` enough?* → No; it checks readability, not that the database restores and is consistent.

**Signals interviewers look for:** you start from objectives, know the three models, explain the chain and differential base, think about retrieval time and ransomware, and insist on tested restores.

### 11. Check yourself
- [ ] Define RPO and RTO and say which backup type each influences.
- [ ] List what each recovery model allows and where log truncation happens.
- [ ] Explain why `BACKUP LOG` fails after `SIMPLE` → `FULL`, and the fix.
- [ ] Explain what an uncontrolled ad-hoc full backup does to the restore sequence.
- [ ] Write the restore commands for a point-in-time restore using full + diff + logs.
- [ ] Describe three controls that protect backups from ransomware.
<!-- explainer:end -->

## References

- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Backups › 12.6 #83 Always using the FULL recovery model on data warehouse and development systems
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Backups › 12.9 #86 Scheduling log backups immediately after a full backup
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Backups › 12.10 #87 Not using COPYONLY backups for ad hoc backups
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Backups › 12.1 #78 Not considering RPO and RTO
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Backup and Restore in Azure SQL
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Backups › 12.8 #85 Not backing up after changing recovery model
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Backups › 12.7 #84 Using SIMPLE recovery model for OLTP databases
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Storage and Backup Management
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Backups › 12.4 #81 Not testing backups
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Backups › 12.11 #88 Forgetting that backups are part of our security posture

