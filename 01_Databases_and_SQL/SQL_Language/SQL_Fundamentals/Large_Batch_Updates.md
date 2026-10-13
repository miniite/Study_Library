---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Large_Batch_Updates"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: []
---

# Large Batch Updates

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Big `INSERT`/`UPDATE`/`DELETE`/`MERGE` jobs fail on log growth, lock escalation and blocking — so process them in bounded batches (or minimally-logged bulk paths), commit between batches, make each batch restartable, and match batch size to the target engine.

### 1. The big picture
Changing millions of rows in one statement looks simplest and is usually worst. A single huge transaction must keep its entire undo/redo history until it commits, can hold locks on a large part of the table, blocks readers and writers, bloats the transaction log (or WAL), and — if it fails near the end — rolls back for as long as it ran. Conversely, millions of *tiny* auto-committed statements flood the log with commit records and run painfully slowly.

The craft is finding the middle: batches big enough to amortize overhead, small enough to bound locks, log and rollback time. The same principle shows up in columnar/lakehouse systems, but with the opposite failure mode: many small writes create many small files.

### 2. Core concepts and vocabulary
- **Transaction log (WAL)** — sequential record of changes used for recovery and rollback; every row change is logged before data pages are flushed.
- **Log flush / commit overhead** — each auto-commit forces a synchronous log write; many commits ⇒ many flushes.
- **Lock escalation** — when one statement holds too many row/page locks (≈5,000 in SQL Server), the engine converts them to a single table lock.
- **Batching** — splitting a big change into chunks, each its own short transaction.
- **Minimal logging / bulk load** — operations (e.g., `INSERT … SELECT` with `TABLOCK` into an empty/heap target under simple or bulk-logged recovery, `BULK INSERT`, `COPY`) that log page allocations rather than every row.
- **Idempotent / restartable** — rerunning the job after a failure yields the same end state without double-applying.
- **Upsert / `MERGE`** — insert-or-update from a source set in one statement.
- **Small-file problem** — in columnar/lake stores, many tiny writes produce many tiny files.

### 3. How it works, step by step
**Why one-row-at-a-time autocommit is slow.** Each statement is its own transaction: parse, plan, lock, write log record, **flush log to disk on commit**. 10,000 single-row inserts means 10,000 log flushes. Fritchey's book example: a loop of 10,000 inserts took ~39 s and pushed a ~8 MB log to ~72 MB; wrapping the loop in one explicit transaction removed the per-row flush overhead.

**Why one gigantic statement is also bad.** One `UPDATE` of 50 M rows holds locks until the end, the log can't be truncated until it commits, and a failure triggers a full-length rollback.

**The batch loop:**
1. Pick a stable, indexed way to select the next chunk (clustered key range, or `TOP (N)` with a `WHERE` that excludes already-processed rows).
2. `BEGIN TRAN` → modify ≤ N rows (commonly 1,000–50,000; keep under the lock-escalation threshold if blocking matters) → `COMMIT`.
3. Between batches allow log backup/checkpoint to truncate log; optionally `WAITFOR DELAY` to yield to other work.
4. Stop when `@@ROWCOUNT = 0`.
5. Record progress (last key) so a restart continues rather than starts over.

### 4. Worked example
```sql
-- Archive-then-delete 200M rows in restartable batches (SQL Server)
DECLARE @batch int = 10000, @rows int = 1;

WHILE @rows > 0
BEGIN
    BEGIN TRAN;

    DELETE TOP (@batch) FROM dbo.EventLog
    WHERE  created_at < '2025-01-01';      -- needs index on created_at

    SET @rows = @@ROWCOUNT;
    COMMIT;

    IF @rows > 0 WAITFOR DELAY '00:00:00.2';   -- breathe: let log backup & readers in
END;

-- Bulk update by key range (deterministic, restartable)
DECLARE @lo bigint = (SELECT MIN(id) FROM dbo.Orders), @hi bigint, @max bigint = (SELECT MAX(id) FROM dbo.Orders);
WHILE @lo <= @max
BEGIN
    SET @hi = @lo + 49999;
    UPDATE dbo.Orders SET status = 'ARCHIVED'
    WHERE  id BETWEEN @lo AND @hi AND status = 'CLOSED';
    SET @lo = @hi + 1;
END;
```
What to watch: `DBCC SQLPERF(LOGSPACE)` or `sys.dm_db_log_space_usage` while it runs; `sys.dm_tran_locks` for escalation to `OBJECT` locks; the query plan shows an index seek on the batch predicate, not a scan per iteration (a scan per batch makes total time quadratic).

`MERGE`-based batch upsert (Ultimate T-SQL): index join keys on both sides, update statistics beforehand, batch with `TOP`, and be alert to deadlocks between concurrent merges — in practice many teams prefer separate `UPDATE` + `INSERT` statements.

### 5. Under the hood
- **Log truncation rules.** In FULL recovery the log can't be reused until a log backup; in SIMPLE it truncates at checkpoint. A single long transaction pins the log from its first record (the *oldest active LSN*), so the log grows even in SIMPLE. Batching is what lets truncation keep up.
- **Rollback cost** is roughly the same as the work already done (single-threaded), so a failed 3-hour update can take hours to undo — and the table stays locked meanwhile.
- **Lock escalation** isn't a bug: holding millions of lock structures costs memory. Batching under ~5,000 locks per statement avoids a table lock; alternatively disable escalation per table or partition-switch.
- **Index maintenance multiplies write cost**: each nonclustered index must be updated for each changed row. For huge loads, disabling/dropping non-essential indexes and rebuilding after can be faster; for updates, avoid touching indexed columns unnecessarily.
- **Replication and AGs**: huge transactions delay log shipping/secondary redo; batching smooths lag.
- **Columnar stores**: inserts below ~102,400 rows land in a delta store (SQL Server columnstore); load ≥102,400 (ideally 1,048,576) rows per batch for compressed row groups. Collect trickle inserts into a staging table first. In BigQuery/Redshift/Snowflake, row-at-a-time DML is expensive; use staged loads and set-based `MERGE`/`COPY`. Lake formats (Iceberg/Delta) need periodic compaction to fix small files.
- **Postgres**: a big `UPDATE` rewrites every touched row (MVCC) and creates dead tuples — batch plus vacuum, and consider HOT updates by leaving fillfactor room.

### 6. Performance and design implications
- Optimal batch size is empirical: benchmark 1k / 10k / 100k and watch log throughput, lock waits and replication lag; stop growing when returns flatten.
- Always drive batches from an index; avoid `OFFSET`-style paging, which rescans.
- Make jobs idempotent (`WHERE status <> target`) so reruns skip finished rows.
- Run big maintenance in off-peak windows, with `LOCK_TIMEOUT` and deadlock-priority low so it yields to OLTP.
- For one-shot "replace everything" operations, consider **partition switching** or build-new-table-and-rename: metadata-only and instantly reversible.
- Triggers fire per statement/row: a per-row `AFTER INSERT` trigger turns bulk loads into row-by-row work; disable or make set-based.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `TOP (n)` / `DELETE TOP` | Bound rows per statement | Batch loops |
| `@@ROWCOUNT` | Loop-exit signal | After each batch |
| `INSERT … WITH (TABLOCK)` | Enables minimal logging on eligible targets | Large loads |
| `BULK INSERT` / `bcp` / `COPY` | Bulk path | File loads |
| `DBCC SQLPERF(LOGSPACE)` | Log fullness | Watch long jobs |
| `ALTER TABLE … SET (LOCK_ESCALATION = …)` | Control escalation | Hot tables |
| `ALTER TABLE … SWITCH PARTITION` | Metadata-only move | Archive/purge |
| `INSERT OVERWRITE` (Spark) | Replace partition | Idempotent backfills |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Batched loop with commits | The whole change must be atomic for correctness |
| One big statement | Table is idle/offline and log space is ample |
| Minimal-logged bulk load | You need point-in-time recovery of that load (FULL recovery limits) |
| Partition switch | No suitable partition key |
| Set-based `MERGE` | Concurrent writers and deadlock-prone key patterns |

### 9. Common mistakes
- One 100M-row `UPDATE` in business hours → blocking and log explosion → batch it.
- Row-by-row cursor/loop with autocommit → hours of runtime → set-based or explicit transaction per batch.
- Batch query scans the table each loop → quadratic runtime → index the filter.
- No restart point → failure means starting over → persist progress or make rows self-marking.
- Forgetting log backups mid-job in FULL recovery → log fills the disk → schedule frequent log backups.
- Trickle-inserting into columnstore/warehouse → tiny files/delta stores → stage and load in large batches.

### 10. 🎯 Interview kit
**30-second answer:** Don't update huge sets in one transaction or one row at a time. Batch in thousands to tens of thousands of rows, driven by an indexed key, committing each batch so locks and the log stay bounded, with an idempotent restart point. Use bulk/minimal-logging paths or partition switch for the largest jobs.
**Structure a longer answer:** state the failure modes (log growth, locks, rollback, replication lag) → batching design (key, size, commit, delay) → idempotency and monitoring → engine-specific notes (columnar, lakehouse).
**Likely follow-ups:**
- *How do you choose batch size?* → benchmark; keep below lock escalation and within log/replication budget.
- *How to make a backfill idempotent?* → partition overwrite or `MERGE` on a deterministic key; safe to rerun.
- *Why is a per-row trigger a bottleneck in bulk loads?* → fires per row; use set-based logic.
- *Hot table 20k inserts/s?* → append-only design, partitioning, batch commit, avoid hot-spot keys.
**Signals interviewers look for:** concrete knobs, awareness of log truncation, idempotency, monitoring, platform differences.

### 11. Check yourself
- [ ] Explain why 10,000 autocommit inserts are slower than one explicit transaction.
- [ ] Write a batched delete loop with an exit condition and a restart strategy.
- [ ] Explain lock escalation and how batching avoids it.
- [ ] Explain why a long transaction prevents log truncation even in SIMPLE recovery.
- [ ] Describe the load strategy for a columnstore table and why.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is a transaction, and why would you wrap inserting a document plus its chunks in one transaction during ingestion?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40781_What_is_a_transaction_and_why_would_you_wrap_inserting_a_doc.md>)
- [Intermediate] [What does a MERGE statement do, and why is it central to an ELT upsert step in a warehouse pipeline?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39981_What_does_a_MERGE_statement_do_and_why_is_it_central_to_an_E.md>)
- [Intermediate] [Your write path does an INSERT then a follow-up SELECT of the generated row, but under READ COMMITTED a concurrent process sometimes doesn't see it. How do RETURNING clauses and same-transaction reads eliminate this round-trip and race?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40432_Your_write_path_does_an_INSERT_then_a_follow-up_SELECT_of_th.md>)
- [Deep] [A stored proc is fast for one parameter and slow for another. Explain parameter sniffing in SQL Server and how OPTIMIZE FOR UNKNOWN, RECOMPILE, or forced plans stabilize it.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39928_A_stored_proc_is_fast_for_one_parameter_and_slow_for_another.md>)
- [Deep] [Why is idempotency important for an ELT SQL step, and how would you make an INSERT step idempotent?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39993_Why_is_idempotency_important_for_an_ELT_SQL_step_and_how_wou.md>)
- [Deep] [Your MERGE-based upsert into a wide table is slow because it rewrites unchanged rows. How would you add a change-hash guard so only genuinely changed rows are updated, cutting write amplification?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40173_Your_MERGE-based_upsert_into_a_wide_table_is_slow_because_it.md>)
- [Deep] [A ledger stores debits and credits, and you must produce a statement with a running balance that resets each account. How do you partition and order the window, and why must you break ties deterministically to keep balances stable across reruns?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40415_A_ledger_stores_debits_and_credits_and_you_must_produce_a_st.md>)
- [Deep] [Explain how you'd store batch-inference job results in SQL keyed by (batch_id, input_row_id) so a partially-failed batch can be resumed without re-running completed rows.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40885_Explain_how_you_d_store_batch-inference_job_results_in_SQL_k.md>)

## Related concepts

- [Batching Writes](<../../Database_Internals/DBMS_Concepts/Batching_Writes.md>) *(DBMS Concepts)*
- [Clustered Indexes and Indexed Views](<../../../TOOLS/SQL_Server/SQL_Server_Platform/Clustered_Indexes_and_Indexed_Views.md>) *(SQL Server Platform)*
- [MERGE Pitfalls and Error Handling](<../../../TOOLS/SQL_Server/T-SQL/MERGE_Pitfalls_and_Error_Handling.md>) *(T-SQL)*

## References

- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 8 Query Recompilation
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 12 Dealing With Index Fragmentation
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 15 Reduce Query Resource Use
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › Performance Considerations with MERGE
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Inserts, Updates, and Batch Size
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Bulk loading data › Bulk Loading into Columnstore Indexes
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Bulk loading data › Trickle Insert vs. Staged Insert
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Delete anD UpDate OperatiOns UP ATE OPERATI › Delete Operations Delete Operations
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Insert Operations
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Update Operations
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Streaming In and Out of Your Delta Lake › Python

