---
id: "TOOLS/SQL_Server/05_Concurrency/Isolation_and_Row_Versioning"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Isolation Levels and Row Versioning

**L4 Advanced** · [↑ Concurrency, Locking and Blocking](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Isolation levels decide which anomalies a transaction can see (dirty reads, non-repeatable reads, phantoms) and how it avoids them: by holding locks longer (pessimistic) or by reading committed row versions from the version store (optimistic RCSI and SNAPSHOT).

### 1. The big picture
The SQL standard defines isolation by the **anomalies** it forbids. SQL Server implements the four standard levels with locking, and adds two **row-versioning** options that give readers a consistent picture without blocking writers. Choosing a level is one of the most important concurrency decisions: it affects correctness, blocking, deadlocks and tempdb load. This lesson uses the lock vocabulary from *Lock Modes and Granularity* and explains the main fix for reader/writer blocking used in *Blocking Analysis*.

### 2. Core concepts and vocabulary
- **Dirty read**: reading another transaction's *uncommitted* change, which may later roll back.
- **Non-repeatable read**: reading the same row twice in one transaction and getting different committed values.
- **Phantom**: re-running a range query and getting new rows that another transaction inserted.
- **Lost update**: two transactions read a value, both compute a new one, and the second write overwrites the first.
- **Row version**: a copy of a row as it was before a modification, tagged with the transaction sequence number (XSN) of the transaction that created it.
- **Version store**: where versions live. It's in tempdb, or in the user database's *persistent version store (PVS)* when Accelerated Database Recovery (ADR) is on.
- **RCSI (Read Committed Snapshot Isolation)**: a database option that makes READ COMMITTED use **statement-level** snapshots.
- **SI (Snapshot Isolation)**: a separate level (`SET TRANSACTION ISOLATION LEVEL SNAPSHOT`) with **transaction-level** snapshots. It must be allowed with `ALLOW_SNAPSHOT_ISOLATION ON`.
- **Update conflict**: under SI, if you update a row another transaction changed after your snapshot started, you get error **3960** and your transaction rolls back.

### 3. How it works, step by step
**The levels and what they allow**

| Level | Dirty read | Non-repeatable | Phantom | How readers behave |
|---|---|---|---|---|
| READ UNCOMMITTED (`NOLOCK`) | possible | possible | possible | no S locks; reads in-flight data |
| READ COMMITTED (locking, default) | no | possible | possible | S lock per row, released right after reading |
| READ COMMITTED with RCSI | no | possible | possible | no S locks; reads last committed version **as of statement start** |
| REPEATABLE READ | no | no | possible | S locks held to end of transaction |
| SNAPSHOT | no | no | no* | no S locks; reads versions **as of transaction start** |
| SERIALIZABLE | no | no | no | S + key-range locks held to end of transaction |

\*SNAPSHOT shows no phantoms in reads, but it is not truly serializable: **write skew** is possible (two transactions each read a value and update *different* rows based on it).

All levels take **X locks for writes** and hold them to the end of the transaction. Writers always block writers on the same row.

**How row versioning works**
1. With RCSI or SI enabled, every UPDATE/DELETE copies the *old* row image to the version store before changing it. Each modified row gets a 14-byte versioning tag (XSN + pointer to the previous version).
2. Versions form a **chain** from newest to oldest.
3. A reader under RCSI notes the current transaction sequence number when its *statement* starts. Under SI, it notes it when its *transaction* first touches data.
4. For each row, the reader walks the chain until it finds the newest version committed before its snapshot point. It takes no S lock (only Sch-S).
5. A background cleanup thread removes versions older than the oldest active snapshot transaction (about once a minute).

**RCSI vs SI**

| | RCSI | SI |
|---|---|---|
| Enabled by | `ALTER DATABASE … SET READ_COMMITTED_SNAPSHOT ON` | `ALLOW_SNAPSHOT_ISOLATION ON` + `SET TRANSACTION ISOLATION LEVEL SNAPSHOT` |
| Code changes | none: the default level changes behavior | sessions must opt in |
| Consistency point | each statement | whole transaction |
| Update conflicts | no; writers block/wait as usual | yes, error 3960 |
| Typical use | OLTP blocking relief | consistent multi-query reports, read-only transactions |

### 4. Worked example
```sql
-- Setup (test DB)
ALTER DATABASE Demo SET ALLOW_SNAPSHOT_ISOLATION ON;
CREATE TABLE dbo.Acct (id int PRIMARY KEY, bal money);
INSERT dbo.Acct VALUES (1, 100), (2, 100);

-- Session 1
BEGIN TRAN;
UPDATE dbo.Acct SET bal = 50 WHERE id = 1;   -- X lock on id 1; old version (100) stored

-- Session 2, default locking READ COMMITTED
SELECT bal FROM dbo.Acct WHERE id = 1;       -- BLOCKS (LCK_M_S)

-- Session 2, SNAPSHOT
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;
BEGIN TRAN;
SELECT bal FROM dbo.Acct WHERE id = 1;       -- returns 100 immediately (version)

-- Session 1
COMMIT;

-- Session 2 (still in the snapshot transaction)
SELECT bal FROM dbo.Acct WHERE id = 1;       -- still 100: transaction-level consistency
UPDATE dbo.Acct SET bal = bal + 10 WHERE id = 1;  -- error 3960: update conflict
```
With RCSI instead (`SET READ_COMMITTED_SNAPSHOT ON`), session 2's second SELECT would return **50**, because each statement sees the latest committed data. Its UPDATE would succeed.

Monitoring:
```sql
SELECT * FROM sys.dm_tran_version_store_space_usage;     -- version store size per DB
SELECT * FROM sys.dm_tran_active_snapshot_database_transactions
ORDER BY elapsed_time_seconds DESC;                      -- long snapshot readers pin versions
SELECT name, is_read_committed_snapshot_on, snapshot_isolation_state_desc
FROM sys.databases;
```

### 5. Under the hood
- **Row overhead.** The 14-byte tag is added when a row is modified while versioning is on. It can cause page splits right after you enable RCSI on a full table.
- **Version generation cost.** Every update and delete also writes to tempdb (or PVS). Inserts create no prior version, but they do get the tag. Triggers use versions too (`inserted`/`deleted` tables), as do online index builds and MARS, even without RCSI.
- **Long-running readers** keep old versions alive, so tempdb grows and version chains get longer. Readers then walk longer chains, which costs CPU.
- **ADR / PVS.** With Accelerated Database Recovery, versions are stored in the user database. That gives instant rollback and fast recovery, but adds database growth.
- **Locking hints still apply.** `READCOMMITTEDLOCK` forces locking reads under RCSI. `UPDLOCK` still takes U locks. A foreign-key check runs with locking read committed even under RCSI.
- **Behavior change risk with RCSI.** Code that reads a row to decide something and then updates (e.g. "check stock then decrement") was protected *by blocking* under locking read committed. Under RCSI it may read a stale committed value. Use `UPDLOCK` or a single atomic `UPDATE ... WHERE stock >= @qty`.

### 6. Performance and design implications
- RCSI often removes most `LCK_M_S` waits in OLTP systems. It's the default in Azure SQL Database.
- Budget tempdb (or PVS) and I/O for versions. Monitor version store size and the longest running snapshot transaction.
- SERIALIZABLE and REPEATABLE READ increase blocking and deadlocks. Use them for short, specific transactions only.
- READ UNCOMMITTED / `NOLOCK` is *not* a performance feature: it gives wrong results (missing or duplicated rows during page splits, uncommitted data). RCSI gives non-blocking reads correctly.
- Prevent lost updates explicitly: atomic `UPDATE ... SET x = x + 1`, `UPDLOCK`, rowversion columns for optimistic checks in the app.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SET TRANSACTION ISOLATION LEVEL <level>` | session isolation level | per workload |
| `ALTER DATABASE d SET READ_COMMITTED_SNAPSHOT ON WITH ROLLBACK IMMEDIATE` | enable RCSI (needs brief exclusive access) | relieve reader/writer blocking |
| `ALTER DATABASE d SET ALLOW_SNAPSHOT_ISOLATION ON` | allow SNAPSHOT level | reports needing one consistent view |
| `ALTER DATABASE d SET ACCELERATED_DATABASE_RECOVERY = ON` | ADR + persistent version store | fast recovery; optimized locking prerequisite |
| Table hints `READCOMMITTEDLOCK`, `SERIALIZABLE`/`HOLDLOCK`, `REPEATABLEREAD`, `NOLOCK` | per-table override | targeted cases |
| `sys.dm_tran_version_store_space_usage`, `sys.dm_tran_active_snapshot_database_transactions` | version-store monitoring | after enabling RCSI/SI |
| `rowversion` column | auto-incrementing per-row version for optimistic concurrency | app-level conflict detection |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| **RCSI**: OLTP with readers blocked by writers; no code changes possible | logic relies on blocking reads to serialize; tempdb is weak |
| **SNAPSHOT**: reports needing several consistent queries | write-heavy transactions (3960 conflicts must be retried) |
| **REPEATABLE READ**: must re-read the same rows unchanged | long transactions (S locks held) |
| **SERIALIZABLE**: strict invariants on ranges ("no two bookings overlap") | high concurrency → range locks, deadlocks |
| **READ UNCOMMITTED**: rough monitoring queries where errors don't matter | anything business-facing |

### 9. Common mistakes
- `NOLOCK` on every query → incorrect totals, duplicate rows, error 601 → switch to RCSI.
- Enabling RCSI without testing → "check-then-act" code races → add `UPDLOCK` or atomic updates.
- Confusing RCSI and SI → expecting transaction-level consistency from RCSI → use SNAPSHOT for that.
- Forgetting retry logic for error 3960 under SNAPSHOT → user-facing failures → catch and retry.
- Leaving a snapshot transaction open for hours → tempdb fills → monitor and kill long-running snapshot transactions.
- Assuming SERIALIZABLE is "safe by default" (e.g. `TransactionScope`) → deadlocks → set READ COMMITTED explicitly.

### 10. 🎯 Interview kit
**30-second answer:** Isolation levels trade consistency against concurrency. READ UNCOMMITTED allows dirty reads. READ COMMITTED prevents them. REPEATABLE READ also prevents changed rows on re-read. SERIALIZABLE prevents phantoms with key-range locks. SQL Server can also use row versioning: with RCSI, read committed reads the last committed version as of each statement, so readers don't block writers. SNAPSHOT gives a transaction-wide snapshot but can fail with update conflicts. Versioning costs tempdb space (or PVS with ADR) and 14 bytes per row. For OLTP, RCSI is usually the best default. NOLOCK is not.
**Structure a longer answer:** anomalies → levels table → locking vs versioning mechanics → RCSI vs SI → costs → behavior-change risks → a scenario (reports blocking order entry, solved by RCSI).
**Likely follow-ups:** *Does RCSI stop writers blocking writers?* → No. · *What's write skew?* → two snapshot transactions update different rows based on the same read, violating an invariant. · *Where are versions stored?* → tempdb, or the PVS in the user DB with ADR. · *Why can NOLOCK return a row twice?* → an allocation-order scan races with page splits.
**Signals interviewers look for:** the anomaly table, RCSI vs SI clearly separated, awareness of version store costs and the check-then-act risk.

### 11. Check yourself
- [ ] Reproduce a non-repeatable read under READ COMMITTED and prevent it with REPEATABLE READ.
- [ ] Show that a SNAPSHOT reader sees the old value while a writer holds X.
- [ ] Trigger error 3960 and write a retry loop.
- [ ] Explain why enabling RCSI can break a "check stock then update" procedure.
- [ ] Query the version store size and find the oldest active snapshot transaction.
<!-- explainer:end -->

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Read Committed Snapshot Isolation (RCSI)
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Transactions and concurrency
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Snapshot Isolation and Row Versioning
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Differences between SI and RCSI
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Enabling Snapshot and RCSI
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 16 Blocking And Blocked Processes
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Trade-offs and Performance Implications
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Snapshot Isolation (SI)
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Managing Transactions › Thinking About Performance
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Optimization › 10.12 #65 Not considering optimistic isolation levels

