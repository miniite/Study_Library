---
id: "TOOLS/SQL_Server/05_Concurrency/Optimized_Locking"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Optimized Locking (2025)

**L4 Advanced** · [↑ Concurrency, Locking and Blocking](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Optimized locking (SQL Server 2025, Azure SQL) cuts lock memory, blocking and escalation in two ways. Writers hold one **transaction-ID (TID) lock** instead of keeping row/page locks to the end, and rows are checked against the latest committed version **before** being locked (lock after qualification). It's the modern answer to problems people used to "solve" with `NOLOCK`.

### 1. The big picture
Classic locking makes a writer keep an X lock on every modified row until commit. A 10,000-row update therefore holds 10,000 key locks (or escalates to a table lock), and any other writer that merely *looks at* those rows while searching for its own targets gets blocked. Over the years people reached for shortcuts: `NOLOCK` / READ UNCOMMITTED, giant `ROWLOCK` hints, disabling escalation. Most shortcuts traded correctness for speed.

Optimized locking changes the engine itself. It builds on two features covered earlier:
- **Accelerated Database Recovery (ADR)**: every row carries the ID of the transaction that last modified it (TID / xactid), and old versions are kept in the persistent version store.
- **RCSI**: readers use versions, so they don't need S locks.

With those in place, the engine no longer needs to *remember* every row lock. The row's own TID already says "transaction 4711 changed me."

This lesson closes the concurrency block. It ties together *Lock Modes*, *Lock Escalation*, *Isolation and Row Versioning* and *Deadlocks*, and it reviews the anti-patterns it replaces.

### 2. Core concepts and vocabulary
- **TID (transaction ID) / xactid**: the ID of the last transaction that modified a row, stored in the row when ADR or row versioning is on.
- **TID locking**: each writing transaction holds an **X lock on its own TID** (resource type `XACT` in `sys.dm_tran_locks`). Row and page X locks are taken only briefly while each row is modified, then released. Another session that wants to change a row modified by an uncommitted transaction waits on that transaction's TID lock (S on the XACT resource).
- **LAQ (Lock After Qualification)**: under RCSI, an UPDATE/DELETE first evaluates its WHERE predicate against the **latest committed version** of each row *without* taking a U lock. It only locks rows that qualify. If a row changed in the meantime, it is re-evaluated.
- **Prerequisites**: ADR on (required), RCSI on (required for LAQ), database option `OPTIMIZED_LOCKING = ON`. It's on by default in Azure SQL Database, and opt-in for SQL Server 2025.
- **Dirty read**: reading uncommitted data that may be rolled back. The main correctness hazard of `NOLOCK`.

### 3. How it works, step by step
**Classic locking: `UPDATE T SET c = c + 1 WHERE status = 'new'` (10,000 qualifying rows)**
1. IX on table, IX on pages.
2. For each row read during the scan: U lock, evaluate predicate, convert to X if it qualifies (release U if not).
3. Keep 10,000 X key locks until commit → escalation at ~5,000 → table X lock.
4. Another writer scanning the same table blocks on the first U/X it meets, even for rows it won't update.

**Optimized locking: same statement**
1. Transaction gets its TID and takes **X on XACT(TID)**. That's held until commit.
2. IX on the table (still needed so DDL/table locks remain correct).
3. For each row: read the latest committed version and evaluate `status = 'new'` *without locking* (LAQ).
4. Qualifying row → take X on the key, modify it (stamping it with our TID), **release the key lock immediately**.
5. At commit, release the TID lock. Anyone who was waiting on our TID continues.
6. Result: at most a handful of row locks exist at any moment → no escalation, small lock memory. Other writers only wait if they actually try to modify one of *our* uncommitted rows.

| Aspect | Classic | Optimized locking |
|---|---|---|
| Locks held to end of transaction | every modified row/page | one XACT (TID) lock |
| Lock escalation from updates | common for big statements | rare |
| Writer scanning past rows another writer touched | blocks on U/X | evaluates committed version; no block unless the row qualifies |
| Prerequisites | none | ADR (+ RCSI for LAQ) |
| Lost-update protection | yes | yes; LAQ re-checks rows that changed |

### 4. Worked example
```sql
-- Enable (test database, SQL Server 2025)
ALTER DATABASE Demo SET ACCELERATED_DATABASE_RECOVERY = ON;
ALTER DATABASE Demo SET READ_COMMITTED_SNAPSHOT ON WITH ROLLBACK IMMEDIATE;
ALTER DATABASE Demo SET OPTIMIZED_LOCKING = ON;
SELECT name, is_accelerated_database_recovery_on, is_read_committed_snapshot_on,
       is_optimized_locking_on            -- DATABASEPROPERTYEX(name,'IsOptimizedLockingOn') on some builds
FROM sys.databases WHERE name = 'Demo';

-- Session 1: big update, keep the transaction open
BEGIN TRAN;
UPDATE dbo.Orders SET Priority = 1 WHERE Status = 'new';   -- 20,000 rows

SELECT resource_type, request_mode, COUNT(*) AS cnt
FROM sys.dm_tran_locks WHERE request_session_id = @@SPID GROUP BY resource_type, request_mode;
-- classic:   KEY X 20000 (or OBJECT X after escalation)
-- optimized: XACT X 1, OBJECT IX 1, PAGE/KEY ~0

-- Session 2: update *different* rows (Status = 'shipped')
UPDATE dbo.Orders SET Priority = 2 WHERE Status = 'shipped';
-- classic: often blocked (escalated table lock or U lock collisions while scanning)
-- optimized: completes; LAQ skips rows that don't qualify without locking them

-- Session 3: update one of session 1's rows
UPDATE dbo.Orders SET Priority = 3 WHERE OrderID = 42;   -- waits on XACT (TID) lock: LCK_M_S on XACT
```

**What optimized locking replaces: the `NOLOCK` habit**

![A NOLOCK reader sees an address change that is later rolled back (dirty read)](<../../../../_assets/100_SQL_Server_Mistakes_and_How_to_Avoid_-_Peter_A_Carter/CH05_F01_Carter.png>)

In the scenario above, an administrator starts updating a customer's delivery address. A salesperson's query with `WITH (NOLOCK)` reads the new, *uncommitted* street. Then the update is rolled back. The order ships to an address that never officially existed. `NOLOCK` can also skip rows or read them twice when pages split during an allocation-order scan, and it can fail with error 601. The correct ways to stop readers being blocked are RCSI (readers use committed versions) and, for writers, optimized locking.

### 5. Under the hood
- **Where the TID lives.** With ADR, each modified row carries version info (TID plus a pointer into the persistent version store). Checking "is the transaction that last changed this row still active?" is a lookup against the TID, which is why the per-row lock no longer needs to be held.
- **Waiting on a TID.** A session that needs to modify a row whose TID belongs to an active transaction requests S on that `XACT` resource. It waits until that transaction ends, so deadlock detection still works (the cycle runs through XACT resources).
- **LAQ conditions.** It applies to UPDATE/DELETE (and the update part of MERGE) under RCSI. It's skipped when the statement uses `UPDLOCK`/`HOLDLOCK`/`REPEATABLEREAD`/`SERIALIZABLE` hints, isolation levels above READ COMMITTED, or certain plan shapes. Then classic U-lock qualification is used, but TID locking still applies.
- **Re-qualification.** If a row's version changes between the lock-free check and the lock (another transaction committed in between), the engine re-evaluates the predicate on the new version. This keeps updates correct (no lost updates) without holding U locks during the scan.
- **Interplay with other features.** Readers under RCSI are unaffected (they never locked). Escalation is still possible for statements that genuinely need table locks, or under higher isolation. DDL still needs Sch-M.
- **Related high-concurrency settings.** `OPTIMIZE_FOR_SEQUENTIAL_KEY` (2019+) addresses *latch* convoys on the last page of an ever-increasing index. Optimized locking addresses *lock* footprint. They solve different problems and can be combined.

### 6. Performance and design implications
- **Lock memory and escalation** drop sharply for large updates. Batch jobs interfere less with OLTP.
- **Writer/writer blocking** drops when writers touch different rows of the same table.
- **Costs:** ADR's persistent version store grows the user database and adds cleanup work. RCSI adds version generation on updates. Measure PVS size (`sys.dm_tran_persistent_version_store_stats`).
- **Behavioral change:** code that relied on U-lock blocking during qualification (e.g. two jobs "claiming" rows by updating a status) still works correctly because of re-qualification, but timing and wait types change. Retest concurrency-sensitive code.
- **Don't** keep using `NOLOCK` "for speed" once RCSI/optimized locking is on. It still gives dirty reads.
- **Data loads:** bulk loads with `TABLOCK` into a heap or minimally logged inserts remain the fastest pattern for staging. Optimized locking is about concurrent DML, not bulk throughput.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ALTER DATABASE d SET ACCELERATED_DATABASE_RECOVERY = ON` | enables ADR + persistent version store | prerequisite |
| `ALTER DATABASE d SET READ_COMMITTED_SNAPSHOT ON` | RCSI; prerequisite for LAQ | non-blocking readers |
| `ALTER DATABASE d SET OPTIMIZED_LOCKING = ON` | enables TID locking + LAQ (SQL 2025) | high-concurrency write workloads |
| `sys.dm_tran_locks` (`resource_type = 'XACT'`) | shows TID locks and waiters | verify it's active |
| `sys.dm_tran_persistent_version_store_stats` | PVS size and cleanup | monitor ADR cost |
| XE `lock_after_qual_stmt_abort` | LAQ fallback/abort events | troubleshooting |
| `READPAST`, `UPDLOCK` hints | queue patterns; note UPDLOCK disables LAQ for the statement | work queues |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Optimized locking: many concurrent writers, big updates causing escalation, SQL 2025/Azure | older versions; storage can't absorb PVS growth |
| RCSI alone: reader/writer blocking only | writer/writer blocking is the real issue |
| `NOLOCK`: never for business data | any query whose result someone acts on |
| Batching DML: still useful for log size and duration | thinking it's the only way to avoid escalation on 2025 |
| `OPTIMIZE_FOR_SEQUENTIAL_KEY`: last-page latch convoys | lock problems (wrong tool) |

### 9. Common mistakes
- Enabling `OPTIMIZED_LOCKING` without ADR → it can't be enabled → turn on ADR first.
- Expecting LAQ without RCSI → only TID locking applies → enable RCSI too.
- Adding `UPDLOCK`/`HOLDLOCK` everywhere out of habit → LAQ turned off for those statements → remove hints that aren't needed.
- Keeping `NOLOCK` in reports "because it was always there" → dirty reads, like the rolled-back address change → remove it; RCSI makes it unnecessary.
- Using `READ UNCOMMITTED` at session level for ETL reads → inconsistent extracts → read from a snapshot or RCSI.
- Ignoring PVS growth after enabling ADR → data file growth → monitor and size storage.

### 10. 🎯 Interview kit
**30-second answer:** Optimized locking, in SQL Server 2025 and Azure SQL, has two parts. With TID locking, a writing transaction holds one exclusive lock on its transaction ID, and row locks are released right after each row is modified, so lock memory and escalation mostly disappear. With lock after qualification, under RCSI an UPDATE or DELETE evaluates its predicate on the latest committed version without locking, and only locks rows that qualify, so writers stop blocking each other on rows they don't actually change. It needs ADR, and RCSI for LAQ. It's the right fix for many problems people used to paper over with NOLOCK, which allows dirty reads and missing or duplicate rows.
**Structure a longer answer:** classic locking costs → prerequisites (ADR/TID, RCSI) → TID locking mechanics → LAQ mechanics and re-qualification → what you see in DMVs → limits and costs → contrast with NOLOCK and sequential-key settings.
**Likely follow-ups:** *Does optimized locking change reader behavior?* → no; RCSI already made readers lock-free. · *How does a writer wait for another's uncommitted row?* → on the XACT (TID) lock. · *Can lost updates happen?* → no; rows are re-qualified if they changed. · *Why is NOLOCK dangerous?* → dirty reads, skipped/duplicated rows, error 601.
**Signals interviewers look for:** knowing it's new in 2025 and the prerequisites, explaining TID vs row locks, LAQ, and rejecting NOLOCK with concrete reasons.

### 11. Check yourself
- [ ] List the three database settings needed for full optimized locking.
- [ ] Compare `sys.dm_tran_locks` output for a 20,000-row update with and without optimized locking.
- [ ] Explain lock after qualification and how re-qualification prevents lost updates.
- [ ] Describe the dirty-read scenario in the figure and the correct alternative.
- [ ] State which problem `OPTIMIZE_FOR_SEQUENTIAL_KEY` solves and why it's different.
<!-- explainer:end -->

## References

- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 16 Blocking And Blocked Processes
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Using Hints for Transaction-Level Control
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › Enhanced Security and Compliance
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Optimization › 10.10 #63 Using Read Uncommitted
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Indexes › 11.8.3 Understanding OPTIMIZEFORSEQUENTIALKEY
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — T-SQL development › 5.2 #15 Using NOLOCK as a performance tweak
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 15 Reduce Query Resource Use
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — SSIS development › 6.2 #23 Not optimizing data loads
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Future Enhancements and Trends

