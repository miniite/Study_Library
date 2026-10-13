---
id: "TOOLS/SQL_Server/05_Concurrency/Lock_Escalation"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Lock Escalation

**L4 Advanced** · [↑ Concurrency, Locking and Blocking](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** When a single statement holds too many row or page locks on one object (about 5,000), or lock memory runs short, SQL Server replaces them with one table (or partition) lock. That saves memory, but can suddenly block everyone else.

### 1. The big picture
Fine-grained locks give concurrency but cost memory and CPU. A statement that updates 2 million rows with row locks would need millions of lock structures. **Lock escalation** is the engine's safety valve: it trades concurrency for resources by swapping many fine locks for one coarse lock, and releasing the fine ones.

Escalation is usually invisible. It becomes a problem in mixed workloads: a nightly purge or a large `UPDATE` escalates to a table X lock, and every OLTP query on that table waits until it commits. Understanding escalation explains a whole class of "everything froze for 30 seconds" incidents. This lesson builds on *Lock Modes and Granularity*.

### 2. Core concepts and vocabulary
- **Lock escalation**: converting many row/page locks on one object into a single HoBT (partition) or OBJECT (table) lock.
- **Escalation threshold**: about **5,000 locks** acquired by *one statement* on *one object* (one index/heap reference). The check repeats every ~1,250 additional locks if the first attempt failed.
- **Memory threshold**: when lock memory exceeds about 40% of the engine's non-AWE memory (or the `locks` configuration), escalation is attempted for sessions holding many locks.
- **Escalation target**: the table, or the partition if `LOCK_ESCALATION = AUTO` on a partitioned table.
- **Failed escalation**: if another session holds an incompatible lock on the table (e.g. IX from an update elsewhere), escalation can't happen. The statement continues with fine-grained locks and retries later. Nothing waits for escalation.
- **De-escalation**: does not exist. Once escalated, the table lock stays until the transaction ends.

### 3. How it works, step by step
1. A statement acquires row or page locks as it processes rows.
2. When it reaches ~5,000 locks on a single object reference, the engine attempts escalation.
3. It tries to take the coarse lock: **S** on the table if the fine locks were S, **X** if any were X/IX.
4. **If compatible** with all other sessions' locks on that table: grant the table lock, release all row/page locks (and intent locks on pages) for that object.
5. **If incompatible** (someone else holds e.g. IX): escalation fails silently. The statement keeps going with row locks and tries again after another 1,250 locks.
6. The table lock is held until the transaction commits or rolls back. Other sessions that need any incompatible lock on that table now wait (`LCK_M_IX`, `LCK_M_S`…).

Things that *don't* count toward the per-statement threshold: locks across different tables, and locks across different statements in the same transaction (each statement is checked on its own, but the memory threshold sees them all).

### 4. Worked example
```sql
-- Watch escalation happen
CREATE EVENT SESSION LockEsc ON SERVER
ADD EVENT sqlserver.lock_escalation(ACTION(sqlserver.sql_text, sqlserver.session_id))
ADD TARGET package0.ring_buffer;
ALTER EVENT SESSION LockEsc ON SERVER STATE = START;

BEGIN TRAN;
UPDATE TOP (10000) Sales.SalesOrderDetail SET UnitPriceDiscount = UnitPriceDiscount;
SELECT resource_type, request_mode, COUNT(*) AS cnt
FROM sys.dm_tran_locks WHERE request_session_id = @@SPID
GROUP BY resource_type, request_mode;
-- After escalation: OBJECT X (1 row), no KEY/PAGE rows for that table
ROLLBACK;

-- Escalation counts per index since restart
SELECT OBJECT_NAME(object_id) AS tbl, index_id,
       index_lock_promotion_attempt_count, index_lock_promotion_count
FROM sys.dm_db_index_operational_stats(DB_ID(), NULL, NULL, NULL)
WHERE index_lock_promotion_count > 0;
```
Now avoid it by batching:
```sql
DECLARE @rows int = 1;
WHILE @rows > 0
BEGIN
    DELETE TOP (4000) FROM dbo.AuditLog WHERE LoggedAt < '20240101';
    SET @rows = @@ROWCOUNT;      -- each batch stays under ~5,000 locks and commits quickly
END;
```

### 5. Under the hood
- **Per-reference counting.** A self-join, or a statement touching a table through two indexes, counts locks per *reference*, not per table total.
- **Partition-level escalation.** `ALTER TABLE ... SET (LOCK_ESCALATION = AUTO)` escalates to the HoBT (partition) level on partitioned tables. Work on different partitions can then continue. The downside: two sessions each holding a partition X lock and wanting the other's partition can deadlock.
- **`DISABLE`** prevents most escalations on that table, but not, for example, escalation under SERIALIZABLE scans of a heap, or when the memory threshold forces it.
- **Trace flags** 1211 (disable escalation completely, even under memory pressure: dangerous) and 1224 (disable count-based escalation, keep memory-based) are server-wide. Prefer the table option.
- **Optimized locking** (SQL Server 2025 / Azure SQL) holds a single XACT (transaction ID) lock instead of keeping row X locks for the duration of the transaction. Row locks are released as soon as each row is modified, so the count rarely reaches 5,000, and escalation largely disappears for updates.
- **Columnstore** and **bulk loads with TABLOCK** take table-level or rowgroup-level locks by design. That isn't escalation, but it has the same effect on concurrency.

### 6. Performance and design implications
- **Batch large modifications** (1,000–4,000 rows per transaction). This prevents escalation, keeps the log small and lets other sessions interleave.
- **Index the WHERE clause** of big updates/deletes. Scanning locks more rows, so you reach the threshold sooner.
- **Partition large tables** and use `LOCK_ESCALATION = AUTO` when maintenance targets single partitions.
- **RCSI** removes S locks for readers, so escalation affects writers only. Readers stop waiting on escalated X locks because they read versions.
- **Monitor:** XE `lock_escalation`, `index_lock_promotion_count`, and `LCK_M_*` waits that spike during batch jobs.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ALTER TABLE t SET (LOCK_ESCALATION = TABLE)` | default; escalate to table | most tables |
| `... = AUTO` | escalate to partition on partitioned tables | partition-wise maintenance |
| `... = DISABLE` | prevent most escalations on the table | proven escalation pain; watch lock memory |
| `DELETE TOP (n)` / `UPDATE TOP (n)` loops | batch to stay under the threshold | purges, backfills |
| XE `sqlserver.lock_escalation` | captures each escalation with text | diagnosis |
| `sys.dm_db_index_operational_stats` | escalation attempts/successes per index | trends |
| DBCC TRACEON (1224, -1) | disables count-based escalation server-wide | last resort |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Let escalation happen: batch jobs in quiet windows | jobs run during OLTP hours |
| Batch the DML: almost always for large changes | you need all-or-nothing atomicity (then use staging + switch) |
| `DISABLE`: specific hot tables with proven issues | huge statements → lock memory pressure, error 1204 |
| `AUTO`: partitioned tables with partition-local work | cross-partition transactions (deadlock risk) |
| Optimized locking: SQL 2025/Azure with ADR + RCSI | older versions (not available) |

### 9. Common mistakes
- Running a 5-million-row `UPDATE` at noon → table X lock → app timeouts → batch it, or run it off-hours.
- Disabling escalation globally (TF 1211) → lock memory exhaustion → use per-table settings.
- Expecting `ROWLOCK` to prevent escalation → it doesn't; it only sets the starting granularity.
- Believing escalation goes row → page → table → it goes straight to table (or partition), never to page.
- Ignoring failed escalation attempts → they're harmless, but a high count shows statements are near the threshold.

### 10. 🎯 Interview kit
**30-second answer:** Lock escalation replaces many fine-grained locks with one table lock, or a partition lock if `LOCK_ESCALATION = AUTO`, when one statement holds about 5,000 locks on one object, or when lock memory gets high. It saves memory but can block every other session on that table until the transaction ends. If an incompatible lock exists, escalation just fails and retries later. I avoid it by batching large DML, indexing so statements touch fewer rows, partitioning, and in rare cases disabling it per table. On SQL Server 2025, optimized locking makes it much rarer.
**Structure a longer answer:** why (lock memory) → thresholds → mechanism and failure case → impact → detection → prevention options with trade-offs.
**Likely follow-ups:** *Does it escalate to page level?* → No; to HoBT/table. · *Does `ROWLOCK` stop it?* → No. · *What happens if escalation can't get the table lock?* → it continues with row locks and retries every ~1,250 locks.
**Signals interviewers look for:** knowing the ~5,000 figure, batching as the default fix, awareness of `LOCK_ESCALATION` options and their risks.

### 11. Check yourself
- [ ] Trigger escalation with a 10,000-row update and see the OBJECT X lock.
- [ ] Rewrite a large DELETE as a batched loop and confirm no escalation.
- [ ] Explain what happens when escalation is attempted but another session holds IX.
- [ ] Compare `TABLE`, `AUTO` and `DISABLE` and name a risk of each.
<!-- explainer:end -->

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Lock Escalation
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Minimizing Lock Escalation
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Lock Escalation
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Escalation Thresholds and Lock Monitoring

