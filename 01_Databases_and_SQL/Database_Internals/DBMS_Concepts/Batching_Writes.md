---
id: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Batching_Writes"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Internals/DBMS_Concepts"
tools: []
---

# Batching Writes

**L4 Advanced** · [↑ DBMS Concepts](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Writing many rows per statement/transaction instead of one at a time cuts round trips, log flushes and per-statement overhead — but batches must be sized so they stay short, restartable and kind to locks and the transaction log.

### 1. The big picture
Every write has fixed costs unrelated to the amount of data: a network round trip, parsing/binding, a transaction commit, and — most expensive — a **synchronous flush of the transaction log to disk** (durability). Inserting 1 million rows as 1 million auto-committed statements pays those fixed costs 1 million times. Inserting them as set-based batches pays them hundreds of times.

Batching is therefore the first optimization for loads, ETL, back-fills, purges and archive jobs. But "bigger is better" is wrong: a single 50-million-row transaction holds locks for minutes, bloats the log, risks lock escalation and takes forever to roll back. The craft is choosing the *right* batch size and structure.

### 2. Core concepts and vocabulary
- **Batch** — a group of rows processed together in one statement or transaction. (In T-SQL "batch" also means a group of statements sent together ending at `GO`; context matters.)
- **Round trip** — one client→server→client exchange; ~0.2–1 ms on a LAN, tens of ms across regions.
- **Commit / log flush** — durability requires the log records up to the commit to reach stable storage before commit returns.
- **Group commit** — the engine flushes the log for several concurrent commits at once.
- **Set-based statement** — one `INSERT … SELECT`, `UPDATE … FROM`, `MERGE`, multi-row `VALUES`; the engine handles all rows in one plan.
- **Bulk load** — specialized path (`BULK INSERT`, `bcp`, `INSERT … SELECT` with `TABLOCK`, `COPY`, `SqlBulkCopy`) with minimal logging when conditions allow.
- **Minimal logging** — logging page allocations rather than every row; requires bulk-logged/simple recovery and conditions such as `TABLOCK` and empty or specific target.
- **Lock escalation** — SQL Server promotes many row/page locks to a table lock (threshold ~5,000 locks per statement per object).
- **Write batching in LSM/NoSQL stores** — memtable + commit log buffering; client batch APIs (Cassandra `BATCH`, DynamoDB `BatchWriteItem`).

### 3. How it works, step by step
**Why 1 row/statement is slow:**
1. Client sends statement → network latency.
2. Server parses/compiles (or looks up cached plan).
3. Row is inserted; log record generated.
4. Autocommit → log flush waits for disk (0.1–5 ms).
5. Reply returns.
→ ~1–2 ms × 1M = ~15–30 minutes even with a fast disk.

**Batched alternatives, from simplest:**
1. **Multi-row insert** — `INSERT … VALUES (…),(…)…` up to 1,000 rows per statement in SQL Server.
2. **Explicit transaction around N inserts** — one commit per N rows.
3. **Table-valued parameter / staging table** — send rows once, then `INSERT … SELECT` set-based.
4. **Bulk API** — streaming rows in large blocks with minimal logging.
5. **Chunked set-based DML** — for updates/deletes, repeat `TOP (N)` in a loop with commits between.

### 4. Worked example
```sql
-- Anti-pattern: row-by-row with autocommit (each INSERT = a log flush)
DECLARE @i INT = 1;
WHILE @i <= 100000
BEGIN
    INSERT dbo.Readings(SensorID, ReadAt, Val) VALUES (@i % 100, SYSDATETIME(), RAND());
    SET @i += 1;
END;                                           -- slow: ~100k commits

-- Better 1: one transaction, still row-by-row   (≈ 10-50x faster; one commit)
BEGIN TRAN; /* same loop */ COMMIT;

-- Better 2: set-based
INSERT dbo.Readings(SensorID, ReadAt, Val)
SELECT n % 100, SYSDATETIME(), RAND(CHECKSUM(NEWID()))
FROM dbo.Numbers WHERE n <= 100000;            -- one statement, one plan

-- Batched purge: delete old rows without a giant transaction
DECLARE @rows INT = 1;
WHILE @rows > 0
BEGIN
    DELETE TOP (5000) FROM dbo.Readings
    WHERE ReadAt < DATEADD(DAY, -90, SYSDATETIME());
    SET @rows = @@ROWCOUNT;
    -- optional: WAITFOR DELAY '00:00:01' to let log backup / replicas catch up
END;
```
**What to look at:** log growth (`sys.dm_db_log_space_usage`), `sys.dm_tran_locks` for lock counts per batch, elapsed time per batch, `sys.dm_exec_requests.wait_type` (`WRITELOG` = log flush bound; `LCK_M_*` = contention). The purge loop keeps each transaction under the 5,000-lock escalation threshold, keeps rollback cheap (a failed batch rolls back 5,000 rows, not 50 M) and is **restartable** because each committed batch is permanent progress.

### 5. Under the hood
- **WRITELOG waits.** With autocommit the session sleeps until the log writer flushes. Batching converts N flushes into 1. Faster storage lowers each flush; batching removes most of them.
- **Log record size.** Fully logged inserts write the row image; indexes add their own records. Wide rows with many nonclustered indexes make each row expensive — drop/disable nonclustered indexes during large loads and rebuild afterward when appropriate.
- **Minimal logging conditions (SQL Server).** Database in `SIMPLE`/`BULK_LOGGED`; target is a heap with `TABLOCK`, or a clustered table with `TABLOCK` and (empty, or trace flag 610 / recent versions for populated); data sorted to match the clustering key helps. Columnstore bulk loads of ≥ 102,400 rows go straight to compressed rowgroups, bypassing the delta store.
- **Lock escalation and the log.** Row locks held until commit; large batches escalate to table locks, blocking readers/writers. Smaller batches avoid it; or `ALTER TABLE … SET (LOCK_ESCALATION = DISABLE)` (carefully).
- **Replication / AG / CDC lag.** A giant transaction is applied on replicas as one unit, producing latency spikes; many small batches flow smoothly.
- **Batching in distributed stores.** Cassandra logged batches guarantee atomicity, not speed, and batches spanning partitions put load on the coordinator — use unlogged batches for single-partition writes, not as a bulk loader. DynamoDB `BatchWriteItem` has 25-item limits and partial failures you must retry.

### 6. Performance and design implications
- Typical sweet spot: 1,000–10,000 rows per transaction for OLTP-visible tables; larger (100k–1M) for staging/bulk paths with minimal logging; tune empirically.
- Measure rows/sec per batch size; the curve flattens, then falls when logs/locks hit.
- Keep batches idempotent (`MERGE` or "delete-then-insert for a key range") so retries are safe.
- Process in key order (clustered key) to keep inserts sequential and page splits low.
- Control log growth: frequent log backups (or `SIMPLE` recovery for staging DBs).
- Throttle (small delay between batches) when replicas or other workloads lag.
- Client side: use TVPs, `SqlBulkCopy`, `COPY`, JDBC `addBatch/executeBatch` with `rewriteBatchedStatements`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `INSERT … SELECT … WITH (TABLOCK)` | Enables minimal logging in right conditions | Loading staging/heap tables |
| `BULK INSERT` / `bcp` / `OPENROWSET(BULK)` | File → table bulk path | File loads |
| `SqlBulkCopy.BatchSize` | Rows per commit from client | .NET loaders |
| `DELETE/UPDATE TOP (n)` loop | Chunked changes | Purges, back-fills |
| `ROWS_PER_BATCH`, `BATCHSIZE`, `ORDER` hints | Guide bulk load | Large file loads |
| `ALTER DATABASE … SET RECOVERY BULK_LOGGED` | Allows minimal logging | Planned big loads |
| `COPY … FROM` (PostgreSQL) | Bulk load | Large loads |
| `MERGE` / `INSERT … ON CONFLICT` | Idempotent batch | Upserts |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Large batches (bulk path) | Table must remain available; log/replica capacity limited |
| Small chunked batches | Fixed overhead dominates — too small wastes the benefit |
| Single big transaction | You need all-or-nothing semantics and can afford the rollback risk |
| Many small independent transactions | Consistency across rows matters |
| Disable indexes then rebuild | Table is queried during the load |

### 9. Common mistakes
- Row-by-row `INSERT` in loops → minutes instead of seconds → set-based or bulk.
- One transaction for 50M rows → log fills, lock escalation, hours to roll back → chunk it.
- Batches without ordering/idempotency → duplicate or missed rows after failure → key-range batches and upserts.
- Ignoring triggers/indexes during loads → each batch pays maintenance → plan disabling and reconciling.
- Unlogged assumption with `BULK_LOGGED` → point-in-time restore limitations → know recovery implications.
- Using NoSQL batches as bulk loaders → coordinator overload → batch per partition, use proper bulk tooling.

### 10. 🎯 Interview kit
**30-second answer:** Batching amortizes per-statement and per-commit costs — round trips, parsing, and the log flush — across many rows. I use set-based or bulk loads for large inserts and chunked `TOP (n)` loops for updates/deletes so transactions stay short, avoid lock escalation and are restartable.

**Structure a longer answer:** clarify (volume, availability needs, recovery model) → why one-at-a-time is slow → batch options (multi-row, transaction grouping, TVP, bulk) → sizing and failure handling → close with a measured example.

**Likely follow-ups:**
- *What is the right batch size?* → empirical; start 1k–10k, watch log, locks, replica lag.
- *Why not one huge transaction?* → lock escalation, log growth, rollback time, replica lag.
- *Minimal logging?* → conditions and recovery model implications.
- *How do you make batches restartable?* → key ranges/watermarks, idempotent statements.
- *Batching in Cassandra/DynamoDB?* → atomicity vs performance; per-partition.

**Signals interviewers look for:** you tie batching to commit/log mechanics and balance it against locks and recoverability.

### 11. Check yourself
- [ ] Explain why autocommit inserts are slow in terms of log flushes.
- [ ] Write a chunked delete loop and say what limits the chunk size.
- [ ] State three conditions for minimal logging.
- [ ] Describe how you make a back-fill restartable.
- [ ] Explain why giant transactions hurt replicas.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is a bulk insert / COPY command and why is it much faster than row-by-row inserts when loading a warehouse?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22573_What_is_a_bulk_insert_COPY_command_and_why_is_it_much_faster.md>)
- [Intermediate] [How does batching multiple INSERTs into one multi-row statement or one transaction reduce commit and WAL overhead?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22537_How_does_batching_multiple_INSERTs_into_one_multi-row_statem.md>)
- [Deep] [What is point-in-time recovery and how would you restore a database to 5 minutes before a bad deploy?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22335_What_is_point-in-time_recovery_and_how_would_you_restore_a_d.md>)
- [Deep] [How would you profile whether the bottleneck in a slow endpoint is the query plan, lock waits, or network round trips?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22500_How_would_you_profile_whether_the_bottleneck_in_a_slow_endpo.md>)
- [Deep] [How do you drop old data cheaply with partition detach/drop instead of a giant DELETE, and why is that so much lighter on the database?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22543_How_do_you_drop_old_data_cheaply_with_partition_detach_drop_.md>)
- [Expert] [How does the write-ahead log (WAL) enable both crash recovery and streaming replication?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22333_How_does_the_write-ahead_log_WAL_enable_both_crash_recovery_.md>)
- [Expert] [What wait events (like buffer busy, log file sync, or lock waits) would you look at first when a database is CPU-idle but queries are slow?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22501_What_wait_events_like_buffer_busy_log_file_sync_or_lock_wait.md>)
- [Expert] [When would you use pg_repack or an online rebuild instead of VACUUM FULL to reclaim space without a long exclusive lock?](<../../../PRACTICE/01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Q-22521_When_would_you_use_pg_repack_or_an_online_rebuild_instead_of.md>)

## Related concepts

- [Large Batch Updates](<../../SQL_Language/SQL_Fundamentals/Large_Batch_Updates.md>) *(SQL Fundamentals)*
- [Read Your Writes in MySQL](<../../../TOOLS/MySQL/MySQL_Platform/Read_Your_Writes_in_MySQL.md>) *(MySQL Platform)*
- [Replica Lag and Read Your Writes](<../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Replica_Lag_and_Read_Your_Writes.md>) *(PostgreSQL Platform)*

## References

- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — has emerged as the key technology for orchestrating the deployment and maintenance of containers
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — BatCh exeCution H XE UTION › CHAPTER 8
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Free Space Page › Bulk Changed Map Page
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Index to Your Environment › Fragmentation Operations
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Heap Bloat and Forwarding
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › Log latency: Batch or real time
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Replay
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Using Time Travel › VACUUM Warnings and Considerations
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Operations on Streaming Data › Write the stream to the output location, maintain\
- *Hadoop: The Definitive Guide* — Tom White — Flume › Batching

