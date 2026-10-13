---
id: "TOOLS/SQL_Server/01_Architecture_and_Storage/Transaction_Log_and_Recovery"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Transaction Log and Recovery

**L4 Advanced** · [↑ Architecture and Storage](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** The transaction log is a sequential record of every change; write-ahead logging makes commits durable, and crash recovery replays (redo) and reverses (undo) log records so the database comes back consistent.

### 1. The big picture
Durability — the "D" in ACID — means once a transaction commits, its changes survive a crash that happens one millisecond later. SQL Server can't afford to write every changed 8 KB data page to random places on disk at commit time, so it writes a compact, **sequential** description of the change to the transaction log and flushes only that. Data pages are written later in bulk. If the server crashes, the log is the source of truth.

The log also powers rollback, savepoints, log backups and point-in-time restore, replication, availability groups, change data capture and more. Its size and health (VLF count, what is preventing truncation) are a daily DBA concern, and its write latency is commit latency for every application.

### 2. Core concepts and vocabulary
- **Log record** — a description of one change (e.g. "insert row X in page 1:312", "modify bytes"), with enough information to redo and undo it.
- **LSN (Log Sequence Number)** — an ever-increasing ID of each log record. Every data page header stores the LSN of the last change applied to it.
- **Write-ahead logging (WAL)** — log records describing a change must be on disk before the changed data page is written, and a commit is acknowledged only after its log records are hardened.
- **Log buffer / log flush** — log records are collected in memory (up to 60 KB log blocks) and written to disk on commit or when the buffer fills.
- **Checkpoint** — writes dirty pages to data files and records a checkpoint in the log, limiting how much log recovery must redo.
- **VLF (Virtual Log File)** — the log file is internally divided into VLFs; they are the unit of reuse (truncation).
- **Active log / MinLSN** — the part of the log still needed (open transactions, unreplicated changes, not-yet-backed-up records in FULL recovery).
- **Log truncation** — marking inactive VLFs as reusable (does not shrink the file).
- **Recovery model** — SIMPLE (log truncated at checkpoint), FULL (truncated only by log backups; point-in-time restore possible), BULK_LOGGED (like FULL but bulk operations minimally logged).
- **Minimal logging** — logging only page allocations instead of every row (e.g. `TRUNCATE`, bulk inserts with `TABLOCK`, index builds under SIMPLE/BULK_LOGGED).
- **Savepoint** — a named point inside a transaction to roll back to without undoing the whole transaction.

### 3. How it works, step by step
**Committing a change**
1. `BEGIN TRAN; INSERT INTO Orders … ;` — the engine modifies the data page in the buffer pool (now *dirty*) and stamps it with the new LSN.
2. A log record for the insert (and a begin-transaction record) goes to the log buffer.
3. `COMMIT` — a commit record is added and the log buffer is **flushed** to the log file. The session waits (`WRITELOG`) until the disk confirms.
4. The client is told "committed". The data page may still exist only in memory.
5. Later, a **checkpoint** (automatic, roughly every minute of recovery work — or indirect checkpoints targeting `TARGET_RECOVERY_TIME`, 60 seconds by default since 2016) or the **lazy writer** writes the dirty page. WAL guarantees its log records are already on disk.

**Crash recovery** (on restart, per database):
1. **Analysis** — scan the log from the last checkpoint to find which transactions were active and which pages may be dirty.
2. **Redo** — replay logged changes whose LSN is newer than the page's LSN, bringing pages forward to the moment of the crash (committed *and* uncommitted work).
3. **Undo** — roll back transactions that never committed, using the log records in reverse.
4. The database comes online. With **Accelerated Database Recovery** (2019+), undo is nearly instant because old row versions are kept in the database's persistent version store.

**Log reuse (circular log)**
1. The log is written VLF by VLF. When the end of the file is reached, writing wraps to the start — *if* the first VLFs are inactive.
2. VLFs become inactive (truncatable) when nothing needs them: no open transaction, checkpoint done, and in FULL/BULK_LOGGED a log backup has taken them.
3. If nothing can be reused, the file **grows** (by its autogrowth setting); if it can't grow, you get error 9002 "log is full". `sys.databases.log_reuse_wait_desc` tells you why.

**Savepoints:**
```sql
BEGIN TRAN;
UPDATE Accounts SET Balance = Balance - 100 WHERE AccountID = 1;
SAVE TRANSACTION BeforeCredit;
UPDATE Accounts SET Balance = Balance + 100 WHERE AccountID = 999; -- wrong account
ROLLBACK TRANSACTION BeforeCredit;   -- undo only the credit
UPDATE Accounts SET Balance = Balance + 100 WHERE AccountID = 2;
COMMIT;
```
The rollback to a savepoint uses log records to undo work after the savepoint; locks taken after it are kept until the transaction ends.

### 4. Worked example
```sql
-- 1. Why can't the log be reused?
SELECT name, recovery_model_desc, log_reuse_wait_desc FROM sys.databases;

-- 2. How full is the log?
SELECT total_log_size_in_bytes / 1048576 AS size_mb,
       used_log_space_in_percent
FROM sys.dm_db_log_space_usage;

-- 3. VLF count vs a sensible target (≈ 2–4 VLFs per GB in this rule of thumb)
SELECT li.database_id, li.ActualVLFs,
       mf.size / 128 / 1024.0 AS SizeInGB,
       mf.size / 128 / 1024.0 * 2 AS MinIdealVLFs,
       mf.size / 128 / 1024.0 * 4 AS MaxIdealVLFs
FROM (SELECT database_id, COUNT(*) AS ActualVLFs
      FROM sys.dm_db_log_info(DB_ID()) GROUP BY database_id) li
JOIN sys.master_files mf ON mf.database_id = li.database_id AND mf.type = 1;

-- 4. Logged vs minimally logged delete of everything
BEGIN TRAN; DELETE FROM dbo.BigTable;  SELECT database_transaction_log_bytes_used FROM sys.dm_tran_database_transactions WHERE database_id = DB_ID(); ROLLBACK;
BEGIN TRAN; TRUNCATE TABLE dbo.BigTable; SELECT database_transaction_log_bytes_used FROM sys.dm_tran_database_transactions WHERE database_id = DB_ID(); ROLLBACK;
```
Interpretation:
- `log_reuse_wait_desc` = `LOG_BACKUP` means the database is in FULL recovery and no log backup is running — the classic runaway log. `ACTIVE_TRANSACTION` means a long open transaction; `REPLICATION`, `AVAILABILITY_REPLICA` point to those features.
- Thousands of VLFs (from tiny autogrowth steps) slow down restores, recovery and availability-group operations. Fix: back up (FULL) or checkpoint (SIMPLE) to clear the log, `DBCC SHRINKFILE` it to small, then grow it in planned large steps and set a sensible growth increment.
- `DELETE` logs every row (and every index row), so log bytes are huge; `TRUNCATE` logs only page deallocations — tiny — yet both can be rolled back inside a transaction.

### 5. Under the hood
- **Log block structure.** Log records are packed into variable-size log blocks (512 bytes to 60 KB) aligned to sectors. A commit flushes the current block; many concurrent commits share one flush (group commit).
- **Page LSN and redo.** Redo compares each log record's LSN with the page's LSN; if the page already has the change, it's skipped. That makes redo idempotent.
- **Undo needs before-images.** For updates the log stores enough to reverse the change; that's why deletes are expensive to log — the whole row is in the record.
- **Compensation log records (CLRs)** are written during rollback, so a crash during rollback doesn't undo the undo.
- **VLF creation.** Each growth adds VLFs (SQL Server 2014+ rules: small growths add 1 VLF if the growth is < 1/8 of the current size; otherwise 4, 8 or 16 VLFs depending on growth size). Many small growths → many VLFs.
- **Zeroing.** Log files cannot use instant file initialization (except growths up to 64 MB from SQL Server 2022), so large log growths and log creation on restore pause while the file is zeroed — another reason not to use multiple log files.
- **`TRUNCATE` vs `DELETE` beyond logging.** `TRUNCATE` resets identity to its seed; it's not allowed when a foreign key references the table (even if empty or disabled); and since 2016 it can target specific partitions: `TRUNCATE TABLE dbo.T1 WITH (PARTITIONS (1, 3, 5, 7 TO 10));`.
- **Delayed durability** (`DELAYED_DURABILITY = ALLOWED/FORCED`) lets commits return before the log flush — faster, but a crash can lose recently committed transactions.

### 6. Performance and design implications
- **Commit latency = log write latency.** Keep the log on low-latency storage; watch `WRITELOG` waits and `sys.dm_io_virtual_file_stats` write stall for the log file (aim for low single-digit ms).
- **Batch work** — committing every row in a loop forces a flush per row; batching thousands of rows per transaction cuts flushes dramatically (but don't make one gigantic transaction that blocks log reuse).
- **Size the log up front** and grow in large fixed increments (e.g. 8 GB) to keep VLF counts reasonable.
- **Back up the log** regularly in FULL recovery — that is both your point-in-time recovery and what truncates the log.
- **Long-running transactions** pin the log (and, with snapshot isolation, the version store).
- **Data warehouse loads** benefit from minimal logging (bulk insert with `TABLOCK`, SIMPLE or BULK_LOGGED), not from extra log files.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sys.databases.log_reuse_wait_desc` | Why the log can't truncate | Log growing / full |
| `sys.dm_db_log_space_usage` | Size and percent used | Monitoring |
| `sys.dm_db_log_info(db_id)` | One row per VLF | VLF fragmentation check |
| `BACKUP LOG db TO …` | Backs up and truncates log (FULL) | Routine, every few minutes |
| `DBCC SHRINKFILE (logname, size)` | Shrinks log file | Fixing VLF sprawl, after emergencies |
| `ALTER DATABASE … SET RECOVERY SIMPLE/FULL` | Recovery model | Match to restore requirements |
| `ALTER DATABASE … SET TARGET_RECOVERY_TIME = 60 SECONDS` | Indirect checkpoints | Predictable recovery time |
| `ALTER DATABASE … SET ACCELERATED_DATABASE_RECOVERY = ON` | Fast undo, quick log truncation | Long transactions, fast failover |
| `SAVE TRANSACTION name` | Savepoint | Partial rollback |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| FULL recovery — need point-in-time restore | No log backups scheduled → log grows forever |
| SIMPLE — dev/test, rebuildable staging DBs | Production data you can't lose between full backups |
| BULK_LOGGED during large loads | Point-in-time restore needed inside the bulk window |
| `TRUNCATE` to empty tables | Referenced by FKs; need to keep identity values |
| Delayed durability for loss-tolerant high-rate logging | Financial or order data |

### 9. Common mistakes
- **FULL recovery without log backups** → log fills the disk (9002) → schedule log backups or use SIMPLE.
- **Tiny autogrowth (1 MB or 10%)** → thousands of VLFs, slow restore and failover → pre-size, fixed large growth.
- **Shrinking the log on a schedule** → it regrows (with zeroing pauses and new VLFs) → size it correctly once.
- **Multiple log files for "performance"** → no gain, slower restores → one log file; extra only temporarily in a disk-full emergency.
- **Row-by-row commits in ETL** → `WRITELOG` waits dominate → batch commits.
- **Assuming `TRUNCATE` can't be rolled back** → it can, inside an explicit transaction (but not once committed — protect production connections).

### 10. 🎯 Interview kit
**30-second answer:** SQL Server uses write-ahead logging: every change is recorded in the sequential transaction log, and commit waits only for the log flush. Dirty data pages are written later by checkpoint and lazy writer. After a crash, recovery runs analysis, redo of logged changes, and undo of uncommitted transactions. The log is reused in VLF units once nothing needs them — in FULL recovery that requires log backups — which is why `log_reuse_wait_desc` is the first thing to check when a log grows.

**Structure a longer answer:** WAL and commit path → checkpoint → recovery phases → log reuse, VLFs and recovery models → performance (log latency, batching, minimal logging) → scenario: log full at 2 a.m. with `LOG_BACKUP` wait.

**Likely follow-ups:**
- *Does a checkpoint truncate the log?* → in SIMPLE yes (marks VLFs reusable); in FULL only log backups do.
- *Why is `TRUNCATE` faster than `DELETE`?* → deallocates pages, logs allocations, not rows.
- *What's ADR?* → persistent version store makes rollback/undo near-instant and allows aggressive log truncation.
- *Why does shrinking the log not fix growth?* → the cause (no backups, open transaction) remains.

**Signals interviewers look for:** precise WAL/recovery mechanics, using `log_reuse_wait_desc` and VLF DMVs, and practical sizing/backups habits.

### 11. Check yourself
- [ ] Explain what is on disk at the moment a `COMMIT` returns.
- [ ] Describe analysis, redo and undo, and why redo is safe to repeat.
- [ ] Diagnose a growing log using `log_reuse_wait_desc` and fix each common cause.
- [ ] Count VLFs for a database and plan how to reduce them.
- [ ] Compare log usage of `DELETE` and `TRUNCATE` on the same table.
<!-- explainer:end -->

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Durability
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 11: Automation with SSIS, SQL Agent, and PowerShell › Automated Recovery and Retry Strategies
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Instance and database management › 9.5 #47 Allowing logs to become fragmented
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 11: Automation with SSIS, SQL Agent, and PowerShell › Checkpoints
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 11: Automation with SSIS, SQL Agent, and PowerShell › Logging Best Practices
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 13: Security, Permissions, and Auditing › Audit Log Management and Retention
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Data ��difi�ati�n
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › SAVE TRANSACTION
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 11: Automation with SSIS, SQL Agent, and PowerShell › Logging and Reporting

