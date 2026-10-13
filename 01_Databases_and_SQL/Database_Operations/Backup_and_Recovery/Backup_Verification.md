---
id: "01_Databases_and_SQL/Database_Operations/Backup_and_Recovery/Backup_Verification"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Operations/Backup_and_Recovery"
tools: []
---

# Backup Verification

**L4 Advanced** · [↑ Backup and Recovery](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A backup is only an asset once you have *proven* it can be read, is internally intact, and restores to a database that is logically correct. "The job succeeded" proves none of that.

### 1. The big picture
Backups exist to serve recovery, and recovery has requirements (how much data you may lose, how long you may take). A backup file that has never been verified is a hope, not a control. The classic failure story is a DBA who discovers during an emergency that the nightly job had been failing for three weeks and the newest usable copy is three weeks old.

Verification sits at the end of the backup lifecycle: take → verify → store → periodically re-verify → restore-test. It answers three different questions that are often confused:

1. **Is the file complete and readable?** (media-level check)
2. **Is the data inside it physically intact?** (checksums, page-level integrity)
3. **Does it restore into a system that is logically correct and meets the recovery-time target?** (restore test + validation queries)

Each question needs a different tool, and passing one does not imply passing the next. This lesson sits under Backup and Recovery, next to retention/restore-at-scale and consistent-backup topics.

### 2. Core concepts and vocabulary
- **Backup set** — the unit written by one backup operation; it can be verified as a whole.
- **Page checksum** — a value computed from the content of an 8 KB page and stored in its header when the page is written to disk; recomputed on read to detect bit rot or torn writes.
- **Backup checksum** — a backup-wide value computed while the backup streams out; it lets a later verify or restore confirm the file was not damaged in storage or transit.
- **Verify-only** — reading the whole backup set and checking structure/checksums without restoring it.
- **Restore test** — actually restoring into a scratch environment, then running integrity checks and business validation.
- **Consistency check** — an engine-level scan of the restored database for physical and logical corruption (e.g. `DBCC CHECKDB` in SQL Server, `RMAN VALIDATE` in Oracle).
- **Silent failure** — a job that stops producing backups, or produces useless ones, without raising an alert.
- **Logical drift** — data that is wrong but structurally valid (a buggy job slowly corrupting one column). Checksums cannot see it, because the bad value was written *correctly*.
- **RTO / RPO** — recovery time / recovery point objective; the restore test is what proves the real RTO.

### 3. How it works, step by step
A solid verification pipeline has layers:

1. **Detect job-level failure.** Alert on missing or failed backups, not just on errors. Track "time since last successful backup per database" and alert when it exceeds the schedule plus a margin. A job that never started raises no error, so absence must be the signal.
2. **Make the backup self-checking.** Enable page checksums on the database (`PAGE_VERIFY CHECKSUM`) and take the backup with the checksum option. The engine validates every page's checksum as it reads it for the backup and stores a backup-wide checksum.
3. **Verify the file.** Run the verify-only command. It reads the entire set, validates the checksum if one exists, checks structure, and confirms the target has enough free space for a restore. It is cheap compared to a restore but it is *not* proof of restorability.
4. **Re-verify stored copies.** Backups age: media degrades, copies are moved, keys rotate. A backup that sat in cold storage for a year should be re-checked (hash compare against the value recorded at creation, then verify-only, then sample restore) before you rely on it.
5. **Restore for real, on a schedule.** Restore the latest full (+ differential + logs) into an isolated environment, rolling forward to a chosen point. Time it, because this measures real RTO.
6. **Check the restored database.** Run the engine's consistency check, then application-level validation: row counts against known figures, referential-integrity queries, control totals, spot queries from the business, and contract checks that downstream consumers rely on.
7. **Record evidence.** Log the date, backup used, duration, and results. Auditors and postmortems need proof, and the log is how you notice the RTO creeping upward.

A table of what each check proves:

| Check | Proves | Does *not* prove |
|---|---|---|
| Job exit code = success | The command ran | Contents are usable |
| Backup checksum + verify-only | File complete and not bit-damaged | Restores in practice; data is logically right |
| Restore to scratch | The chain is complete and restorable; real RTO | Business correctness |
| Consistency check on restore | No physical/structural corruption | Logical drift in valid data |
| Business validation queries | Data matches invariants | Anything you didn't query |

### 4. Worked example
```sql
-- 1. Make pages self-verifying and take a checksummed, compressed backup
ALTER DATABASE Marketing SET PAGE_VERIFY CHECKSUM;

BACKUP DATABASE Marketing
TO DISK = N'\\backupsrv\sql\Marketing_full.bak'
WITH CHECKSUM, COMPRESSION, STOP_ON_ERROR, INIT;

-- 2. Verify the file without restoring it
RESTORE VERIFYONLY
FROM DISK = N'\\backupsrv\sql\Marketing_full.bak'
WITH CHECKSUM;

-- 3. Real test: restore under another name, then check it
RESTORE DATABASE Marketing_verify
FROM DISK = N'\\backupsrv\sql\Marketing_full.bak'
WITH MOVE 'Marketing'     TO 'T:\verify\Marketing.mdf',
     MOVE 'Marketing_log' TO 'T:\verify\Marketing.ldf',
     CHECKSUM, STATS = 10;

DBCC CHECKDB (Marketing_verify) WITH NO_INFOMSGS, ALL_ERRORMSGS;
```
What happens: step 1 makes SQL Server compute a checksum for each page as it is read from the data file for the backup; a damaged page stops the backup (`STOP_ON_ERROR`) instead of being silently written into the file. Step 2 re-reads the backup and confirms the stored checksum; it prints "The backup set on file 1 is valid." On failure you get a checksum/media error and should treat that file as unusable. Step 3 proves the chain restores; the elapsed time is your measured RTO for this database. `DBCC CHECKDB` returning no messages means the restored copy is physically and structurally sound.

Then run business checks, for example:
```sql
SELECT COUNT(*) FROM Marketing_verify.dbo.Campaign;      -- vs. expected
SELECT COUNT(*) FROM Marketing_verify.dbo.Lead l
LEFT JOIN Marketing_verify.dbo.Campaign c ON c.CampaignID = l.CampaignID
WHERE c.CampaignID IS NULL;                              -- orphans: must be 0
```

For a lakehouse table the shape is the same: keep a version-controlled "known good version" pin, run a restore-to-version command, run validation queries, and write the measured recovery seconds to an audit table to prove the objective was met.

### 5. Under the hood   *(optional at this level)*
- Page checksum is computed at write time and checked at read time, so it detects corruption introduced *by the storage stack after the engine wrote the page*. If the engine writes a bad page (bug, memory error before the checksum), the checksum is "valid" for bad content. This is why a checksum-clean backup can still restore into a corrupt database.
- A backup checksum covers the backup stream; it protects the file in storage and transit, not the logical content.
- Verify-only reads the media; it does not rebuild the database, so it cannot detect a missing log in the chain you did not include, nor problems in recovery itself.
- Open transactions during a backup: the engine includes enough of the log to bring the restored data to a transactionally consistent state at the end of the backup; recovery on restore rolls back what was uncommitted. So a hot backup is consistent after restore *by design*, as long as the engine's own backup mechanism (not a bare file copy) is used.

### 6. Performance and design implications
- Checksummed backups cost CPU and lengthen backup time, noticeably on large databases. Accept it; the alternative is trusting unverified bytes.
- Restore tests cost storage and IO. Run them on a test server, ideally on a rolling sample of databases so that each is covered within a defined period; prioritise tier-zero systems more often.
- Verify jobs against cold/object storage can incur retrieval fees and take hours; plan hash-based pre-checks first.
- Measure: backup duration, backup size trend, restore duration, `msdb` backup history (or your backup catalog) for missing days, and time-since-last-good.
- Detecting logical drift needs *domain* checks: per-column distribution/hash baselines, row-count deltas between successive backups, reconciliation against a source of truth.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ALTER DATABASE … SET PAGE_VERIFY CHECKSUM` | Writes/validates a checksum per page | Always on for modern databases |
| `BACKUP … WITH CHECKSUM` | Validates pages and stores a backup-wide checksum | Every backup |
| `RESTORE VERIFYONLY … WITH CHECKSUM` | Validates the backup file without restoring | After each backup, and before relying on an old copy |
| `RESTORE … WITH MOVE` to a scratch name | Real restore test | On a schedule, and after any backup-process change |
| `DBCC CHECKDB` / `RMAN VALIDATE` | Consistency check of a database/backup | On the restored copy; regularly on production |
| `RESTORE HEADERONLY` / `FILELISTONLY` | Read metadata of the set | Cheap sanity check of cold-storage files |
| `msdb.dbo.backupset` | Backup history | Alerting on gaps |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Verify-only after every backup | You treat it as proof of restorability |
| Full restore tests on a rolling schedule | Storage/time budget doesn't exist; then prioritise by tier |
| Checksums on every backup | Backup window is razor-thin; then lengthen the window rather than drop checks |
| Automated validation in CI/CD or a scheduler | Nobody owns the failures; alerts must have an owner |

### 9. Common mistakes
- Trusting the job's success status → silent gaps and zero-byte files go unnoticed → alert on "no good backup in N hours".
- Backups without checksum → corrupt pages are written into the backup → enable `WITH CHECKSUM` and page verify.
- Verifying only the newest file → the older ones in the chain may be bad → verify the whole restore chain.
- Never restoring → discovered RTO is 9 hours against a 1-hour target → schedule restore drills and record timings.
- Validating bytes but not meaning → restore "works" but application reports inconsistent data → add business and referential checks.
- Ignoring hardware/network as a cause of corruption → repeated corruption → investigate storage and transfer paths too.
- Not acting after one incident → recurrence → add preventive controls (monitoring, second copy, drills).

### 10. 🎯 Interview kit
**30-second answer:** "I treat a backup as unproven until verified in layers: monitoring that a backup exists, checksums plus verify-only to prove the file is intact, and scheduled restores into a scratch environment followed by a consistency check and business validation queries. The restore drill also measures the real RTO."

**Structure a longer answer:** clarify (engine, RPO/RTO, tier) → layered checks (exists, intact, restores, correct) → tooling (checksum, verify-only, CHECKDB/RMAN, validation SQL) → trade-offs (cost vs coverage, sampling) → a scenario (three corrupt backups).

**Likely follow-ups:**
- *What does a checksum prove vs a restore?* → Checksum: bytes not damaged since written. Restore: chain complete, usable, timed. Neither proves logical correctness.
- *Last three backups are corrupt, what now?* → Look for older sets, log backups, standby/replica, storage snapshots; reconstruct from source systems; then root-cause the cause (storage, transfer, bug) and add verification + a second copy.
- *How do you test DR without hurting production?* → Isolated restore environment, synthetic failure, scheduled game days.
- *How do you catch slow logical drift?* → Column-level baselines, row-count/hash deltas between backups, reconciliation, data contracts.
- *Backup in cold storage for a year?* → Compare recorded hash, header/metadata check, verify-only, sample or full restore.

**Signals interviewers look for:** distinguishing integrity from restorability from correctness; measuring RTO; ownership of alerts; treating verification as automated and evidenced, not heroic.

### 11. Check yourself
- [ ] Explain why a successful job exit code is not evidence of recoverability.
- [ ] Describe what `PAGE_VERIFY CHECKSUM` and `BACKUP … WITH CHECKSUM` each protect, and what neither can detect.
- [ ] Write the three statements that back up, verify, and restore-test a database.
- [ ] Design an alert for a silently missing backup.
- [ ] Explain how you would detect a bug that has been corrupting one column for weeks.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [What is the purpose of a backup checksum, and how does it differ from validating a restore?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Backup_and_Recovery/Q-04133_What_is_the_purpose_of_a_backup_checksum_and_how_does_it_dif.md>)
- [Deep] [A backup file passes checksum validation but the restore fails with corruption errors. How do you diagnose this?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Backup_and_Recovery/Q-04114_A_backup_file_passes_checksum_validation_but_the_restore_fai.md>)
- [Deep] [How do you validate that your standby database is actually caught up and safe to promote during a disaster?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Backup_and_Recovery/Q-04131_How_do_you_validate_that_your_standby_database_is_actually_c.md>)
- [Deep] [What checks would you automate to detect backup drift, such as tables silently excluded from the backup?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Backup_and_Recovery/Q-04152_What_checks_would_you_automate_to_detect_backup_drift_such_a.md>)
- [Deep] [How do you recover when a restore succeeds technically but the application reports data inconsistencies afterward?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Backup_and_Recovery/Q-04164_How_do_you_recover_when_a_restore_succeeds_technically_but_t.md>)
- [Deep] [Explain how backup validation jobs like RMAN VALIDATE or DBCC CHECKDB fit into a recovery-readiness process.](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Backup_and_Recovery/Q-04181_Explain_how_backup_validation_jobs_like_RMAN_VALIDATE_or_DBC.md>)
- [Deep] [How would you detect and recover from a scenario where replication silently stopped and your standby is days stale?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Backup_and_Recovery/Q-04185_How_would_you_detect_and_recover_from_a_scenario_where_repli.md>)
- [Expert] [How would you architect backup validation to detect slow, silent logical drift, such as a bug that has been subtly corrupting one column for weeks, before it poisons every recovery point?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Backup_and_Recovery/Q-04065_How_would_you_architect_backup_validation_to_detect_slow_sil.md>)

## Related concepts

- [HA Failover Design](<../High_Availability/HA_Failover_Design.md>) *(High Availability)*
- [Read Your Writes with Replicas](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Read_Your_Writes_with_Replicas.md>) *(Cloud Fundamentals)*
- [Verifying Live Migrations](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Verifying_Live_Migrations.md>) *(Distributed Systems Core)*

## References

- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Google DiRT Disaster Recovery Testing › Data integrity
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Instance and database management › 9.8 #50 Not regularly checking for corruption
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Backups › 12.4 #81 Not testing backups
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Backups › 12.4.1 Checking that backups completed successfully
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Backups › 12.4.2 Verifying backup integrity
- *Think Distributed Systems* — Dominik Tornow — Failure tolerance › Be vigilant

