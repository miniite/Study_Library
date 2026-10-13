---
id: "TOOLS/MySQL/MySQL_Platform/MySQL_Performance_Diagnosis"
level: 4
type: "concept"
topic: "TOOLS/MySQL/MySQL_Platform"
tools: ["MySQL", "Redis"]
---

# MySQL Performance Diagnosis

**L4 Advanced** · [↑ MySQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Diagnose MySQL performance by measuring first — slow query log and `performance_schema` digests for *which* statements hurt, `EXPLAIN` for *why*, and connection/lock/replication metrics for *system* health — then fix the layer that actually costs time: round trips, indexes, connection counts, or the way the data is read and written.

### 1. The big picture
"The database is slow" is rarely one thing. It can be one bad query, thousands of cheap queries (N+1, polling), too many connections, lock waits, an overloaded buffer pool, replica lag, or a batch job holding resources. A diagnosis workflow keeps you from guessing:

1. **Observe** at system level (CPU, I/O, connections, lock waits, replication lag).
2. **Rank** statements by total cost (slow log / digest).
3. **Explain** the top offenders (`EXPLAIN ANALYZE`, indexes, rows examined vs returned).
4. **Fix** at the right layer (query, index, schema, access pattern, pooling, caching).
5. **Verify** with the same measurement and add an alert so it can't silently return.

This lesson also covers the design patterns that avoid whole classes of problems: queue tables that stay fast as the backlog grows, maintained counters instead of `COUNT(*)`, async exports with streaming, and sensible connection pooling.

*(Book context: Spark/data-algorithm recipes emphasize the same principle — avoid shipping all data to one place and aggregate where the data lives; in MySQL that means push aggregation into SQL, stream large reads, and precompute.)*

### 2. Core concepts and vocabulary
- **Slow query log** — file/table of statements exceeding `long_query_time`; also `log_queries_not_using_indexes`.
- **Digest** — normalized statement text with literals replaced by `?`; `performance_schema.events_statements_summary_by_digest` aggregates counts and times per digest.
- **Rows examined vs rows sent** — engine work vs result size; a large ratio means missing/ineffective index.
- **Buffer pool** — InnoDB's in-memory cache of data and index pages (`innodb_buffer_pool_size`); hit ratio = 1 − (disk reads / read requests).
- **Connection pool** — a set of reusable connections held by the application; limited by `max_connections` on the server.
- **Lock wait** — a session blocked on a row/metadata lock; visible in `sys.innodb_lock_waits`.
- **Replication lag** — delay between primary commit and replica apply (`Seconds_Behind_Source` / `performance_schema.replication_applier_status_by_worker`).
- **Streaming result** — client fetches rows incrementally instead of buffering the full set (`mysql_use_result`, JDBC streaming).
- **Keyset (seek) pagination / queue table / maintained counter** — patterns for scale described below.

### 3. How it works, step by step — a diagnosis loop
1. **Enable evidence.** `SET GLOBAL slow_query_log = ON; SET GLOBAL long_query_time = 0.5;` (or 0 briefly for a sample window), and make sure `performance_schema` (default ON) and the `sys` schema are available.
2. **Rank.** `SELECT * FROM sys.statement_analysis ORDER BY total_latency DESC LIMIT 10;` or `pt-query-digest slow.log` — sort by *total* time (count × avg), not just worst single run.
3. **Classify each top statement:**
   - high rows_examined/rows_sent → index or predicate problem;
   - huge count with tiny latency → N+1 / polling → batch, cache, or redesign;
   - high lock time → transaction scope/order;
   - big sorts/temp tables on disk → index for order or reduce data.
4. **Explain.** `EXPLAIN ANALYZE <stmt>` shows actual rows and time per step.
5. **Change one thing**, re-measure on the same workload.
6. **Guard.** Dashboard + alerts: slow-query rate, p95 latency, connections used %, `Innodb_row_lock_waits`, buffer pool hit ratio, replication lag, deadlock count, disk and redo log pressure.

### 4. Worked example
```sql
-- Top time consumers
SELECT digest_text, count_star, ROUND(sum_timer_wait/1e12,1) AS total_s,
       ROUND(avg_timer_wait/1e9,1) AS avg_ms, sum_rows_examined, sum_rows_sent
FROM   performance_schema.events_statements_summary_by_digest
ORDER  BY sum_timer_wait DESC LIMIT 5;

-- Connections: are we near the ceiling?
SHOW GLOBAL STATUS LIKE 'Threads_connected';
SHOW GLOBAL STATUS LIKE 'Max_used_connections';
SHOW VARIABLES LIKE 'max_connections';
SELECT user, COUNT(*) FROM information_schema.processlist GROUP BY user;
```
**Too many connections.** Typical root cause: `pods × pool_size > max_connections` (200 pods × pool 20 = 4,000). Fixes: shrink per-pod pools (a pool of 5–10 serves far more than its size because most requests spend time outside the DB), add a proxy/connection multiplexer (ProxySQL, RDS Proxy), reduce idle connections with `wait_timeout`, use per-request timeouts so slow queries don't hold connections. Do not just raise `max_connections` — each connection costs memory (per-thread buffers) and more concurrent runners usually slows throughput.

**Bulk insert, 500k rows in an hour → minutes**
```sql
-- Bad: autocommit, 500k round trips, 500k log flushes
INSERT INTO t (a,b) VALUES (1,'x');   -- repeated

-- Good: batches of ~1,000 rows per statement inside one transaction per batch
START TRANSACTION;
INSERT INTO t (a,b) VALUES (1,'x'),(2,'y'), … ,(1000,'z');
COMMIT;

-- Best for files
LOAD DATA LOCAL INFILE '/path/data.csv' INTO TABLE t FIELDS TERMINATED BY ',';
```
Layers: client round trips (batch), commit flushes (`innodb_flush_log_at_trx_commit=1` fsyncs per commit — fewer commits), index maintenance (load before adding secondary indexes, or sort by PK), and constraint checks. Expect orders-of-magnitude, not percent, gains.

**Queue table that stays fast**
```sql
CREATE TABLE email_job (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  status TINYINT NOT NULL,          -- 0 pending, 1 sent, 2 failed
  run_after DATETIME NOT NULL,
  payload JSON NOT NULL,
  KEY ix_pending (status, run_after, id)
);

START TRANSACTION;
SELECT id, payload FROM email_job
WHERE status = 0 AND run_after <= NOW()
ORDER BY run_after, id LIMIT 100
FOR UPDATE SKIP LOCKED;                -- 8.0: workers don't block each other
-- ... mark rows as in-progress / sent ...
COMMIT;
```
Keep the *pending* set small: move finished rows to an archive table or delete in chunks, so `ix_pending` doesn't grow with history. Partition by day if retention is by time.

**Counters.** Instead of `SELECT COUNT(*) FROM likes WHERE post_id = ?` on every read, maintain `post.like_count` in the same transaction as the `likes` insert/delete (`UPDATE post SET like_count = like_count + 1 WHERE id = ?`), reconcile nightly, or shard the counter row for hot items.

**Large export.** Run as an async job: stream rows with a server-side cursor (`useCursorFetch`/`fetchSize` in JDBC, `mysql_use_result` in C APIs, `SSCursor` in Python) in primary-key order with keyset pagination, write to object storage, and let the user download. A single request holding a connection for minutes belongs in a worker, not the web tier.

**Set a query ceiling.** `SELECT /*+ MAX_EXECUTION_TIME(2000) */ …` (read-only `SELECT`s) or `SET SESSION max_execution_time = 2000`; plus driver-level socket/statement timeouts so one runaway query can't exhaust the pool.

### 5. Under the hood
- **Query cache** was removed in 8.0 (invalidated on any table write, scaled badly). Hot-read acceleration now comes from the buffer pool, application/Redis caching, replicas, and covering indexes.
- **Buffer pool.** Pages are 16 KB; LRU with a young/old sublist so scans don't evict hot pages. A hit ratio dropping or `Innodb_buffer_pool_reads` rising signals the working set no longer fits.
- **Streaming internals.** By default the client library buffers the entire result in memory (`mysql_store_result`). Streaming reads rows as the server produces them, holding the connection and server-side resources (including a read view → long transactions keep old undo) until the last row; so stream from a replica or in chunks for very large sets.
- **`INSERT … SELECT`** under `REPEATABLE READ` takes shared next-key locks on the source rows it reads (for statement-based replication safety). Under `READ COMMITTED` with row-based binlog the lock footprint shrinks; or copy in primary-key chunks.
- **`ANALYZE TABLE` and plan flips.** Persistent stats are re-sampled by `ANALYZE` (and auto-recalc after ~10% change). A new sample can shift cardinality estimates and flip a plan. Make it predictable: raise `innodb_stats_persistent_sample_pages` for big/skewed tables, schedule analyze in a maintenance window on a replica first, use histograms on skewed non-indexed columns, pin critical plans with optimizer hints, and alert on plan changes by comparing digest latency before/after.
- **BLOBs in rows.** Large `BLOB`/`TEXT` values live in overflow pages; they bloat the buffer pool, backups and replication, and slow `SELECT *`. Store files in object storage and keep a key/URL, size, and checksum in MySQL.
- **Connection cost.** Each thread has per-session buffers (sort, join, read buffers) allocated on demand; memory ≈ global buffers + connections × per-thread usage.

### 6. Performance and design implications
- **Aggregates:** compute on the fly if data is small and indexed; cache (Redis/app) if expensive but tolerates staleness; store as a maintained column or summary table if read constantly — and define how it is repaired (periodic recompute, idempotent rebuild by key).
- **Incremental materialization** of a nightly 100M-row aggregate: maintain per-day/per-key rollups by processing only rows with `id > last_processed_id` or `updated_at >= watermark`, upsert with `ON DUPLICATE KEY UPDATE`, and handle late/updated source rows by re-aggregating the affected keys.
- **Polling endpoints:** store job status in a small keyed table or cache, add backoff/ETag, or push via SSE/WebSocket instead of one-second polling.
- **Dashboards with six aggregates:** combine aggregates over the same filter in one query (conditional `SUM(CASE …)`), cache what tolerates staleness, precompute what is heavy.
- **Rate limiter:** fixed/sliding window in MySQL works for modest rates with an indexed `(key, window_start)` upsert; at high QPS, per-request writes make the DB the bottleneck — use Redis.
- **Observability set:** QPS and p95/p99 latency, slow queries per minute, `Threads_running`, connection utilization, row lock waits/time, deadlocks, buffer pool hit rate and dirty pages, redo/checkpoint age, replica lag, disk free, and `Aborted_connects`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `slow_query_log`, `long_query_time`, `log_queries_not_using_indexes` | Capture slow statements | Always on with a sane threshold |
| `pt-query-digest`, `mysqldumpslow` | Aggregate slow log | Rank offenders |
| `performance_schema`, `sys.statement_analysis`, `sys.innodb_lock_waits`, `sys.schema_unused_indexes` | Built-in diagnostics | First stop in 5.7/8.0 |
| `EXPLAIN ANALYZE` | Actual plan timing | Verify fix |
| `SHOW PROCESSLIST` / `information_schema.processlist` | Who is running what | Live incident |
| `SHOW ENGINE INNODB STATUS` | Buffer pool, locks, log, deadlocks | Deep dive |
| `max_connections`, `wait_timeout`, `thread_cache_size` | Connection controls | Capacity |
| `MAX_EXECUTION_TIME` hint / `max_execution_time` | Kill long SELECTs | Protect the pool |
| `LOAD DATA`, multi-row `INSERT` | Fast ingestion | Bulk loads |
| `innodb_buffer_pool_size`, `innodb_flush_log_at_trx_commit`, `sync_binlog` | Memory/durability knobs | Tuning with eyes open |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Fix the query/index first | Pure capacity problem (working set > RAM) — add memory/replicas |
| Larger pool/more connections | The DB is already CPU- or lock-bound; more concurrency lowers throughput |
| Relax durability (`flush_log_at_trx_commit=2`) for rebuildable data | Money/ledger data |
| Maintained counter | Concurrent hot-row contention (shard it) or frequent drift |
| Cache aggregates | Users need read-your-writes accuracy |
| Streaming from primary | Long read views block purge; use a replica |

### 9. Common mistakes
- **Tuning from the single slowest query** → ignoring a 2 ms query run 40M times → sort by total time.
- **Raising `max_connections` to "fix" errors** → memory pressure and slower throughput → pool sizing/proxy.
- **Per-pod pools summed beyond the server limit** → connection storms at scale-out → budget connections.
- **`SELECT COUNT(*)` for badges on every page** → table scans → maintained counters.
- **Large `INSERT … SELECT` or `DELETE` in one transaction** → long locks, huge undo, replica lag → chunk by PK.
- **Storing files in BLOB columns** → bloat → object storage.
- **No statement timeouts** → one runaway query starves the pool → timeouts at DB and client.
- **Running `ANALYZE TABLE` blindly on the primary at peak** → plan flips → controlled, tested analyze.
- **Alerting only on CPU** → missing lock waits and lag → multi-signal alerts.

### 10. 🎯 Interview kit
**30-second answer:** I measure first: slow log and `performance_schema` digests ranked by total time, then `EXPLAIN ANALYZE` on the top offenders. Typical fixes are an index, fewer round trips (batching/N+1), connection pool limits, chunking big writes, precomputing aggregates, and streaming big reads — then I alert on the metric so it stays fixed.

**Structure a longer answer:** symptoms → system-level metrics → statement ranking → plan analysis → targeted change → verification → guardrails (alerts, CI plan checks, timeouts).

**Likely follow-ups:**
- *Slow log usage?* → `long_query_time`, digest with `pt-query-digest`, look at rows examined and lock time.
- *`max_connections` vs pool size?* → pool is per app instance; sum of pools must stay below server limit with headroom; use proxy.
- *Why was the query cache removed?* → global invalidation and mutex contention; use app caches and replicas.
- *How do you speed up 500k inserts?* → batch, one transaction per batch, `LOAD DATA`, defer indexes, fewer fsyncs.
- *Keep a queue table fast?* → narrow pending index, `SKIP LOCKED`, delete/archive done rows, partition.
- *Too many connections spike?* → check `Threads_connected`, leaked connections, slow queries holding them, pool sizing, proxy.

**Signals interviewers look for:** a repeatable method, awareness of connection and write-path costs, and designing for growth rather than one-off tuning.

### 11. Check yourself
- [ ] List five dashboard metrics and an alert threshold idea for each.
- [ ] Explain why `pods × pool_size` can exhaust `max_connections`.
- [ ] Rewrite a row-at-a-time load into an efficient batch.
- [ ] Design a pending-jobs table whose fetch query doesn't degrade with backlog.
- [ ] Explain how streaming reads work and their risks.
- [ ] Describe how to make `ANALYZE TABLE` safe in production.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../_index.md>)
- [Redis](<../../Redis/_index.md>)

## Practice questions

- [Intermediate] [A batch INSERT ... SELECT copies a million rows and blocks reads on the source table — how do you reduce its lock footprint and isolation impact?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33468_A_batch_INSERT_SELECT_copies_a_million_rows_and_blocks_reads.md>)
- [Intermediate] [Explain how you'd measure query cost fairly across environments given MySQL 8 dropped the query cache — what replaces it and how do you keep hot reads fast?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33287_Explain_how_you_d_measure_query_cost_fairly_across_environme.md>)
- [Deep] [How would you maintain a fast, accurate counter (like total likes) without running COUNT(*) on every read?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33089_How_would_you_maintain_a_fast_accurate_counter_like_total_li.md>)
- [Deep] [Prod latency spiked at 2am and slow_log shows a query that was fast yesterday now doing a full scan — walk me through how you'd root-cause a plan flip in InnoDB.](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33108_Prod_latency_spiked_at_2am_and_slow_log_shows_a_query_that_w.md>)
- [Deep] [Explain how you'd detect and fix a table that's accumulating InnoDB fragmentation and bloated free space, and when OPTIMIZE TABLE is worth the lock.](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33194_Explain_how_you_d_detect_and_fix_a_table_that_s_accumulating.md>)
- [Deep] [Explain how you'd handle a "thundering herd" of cache misses all issuing the same expensive MySQL query at once, at both the app and DB layer.](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33276_Explain_how_you_d_handle_a_thundering_herd_of_cache_misses_a.md>)
- [Expert] [What is connection pooling at the proxy layer and how does ProxySQL help MySQL scale connection counts?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33096_What_is_connection_pooling_at_the_proxy_layer_and_how_does_P.md>)
- [Expert] [Explain the trade-offs of innodb_flush_log_at_trx_commit and sync_binlog settings when you need both throughput and no data loss on power failure.](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33181_Explain_the_trade-offs_of_innodb_flush_log_at_trx_commit_and.md>)

## Related concepts

- [Batching Writes](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Batching_Writes.md>) *(DBMS Concepts)*
- [B-tree Concurrency and Latches](<../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/B-tree_Concurrency_and_Latches.md>) *(Storage Engines)*
- [Index Tuning in Practice](<../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/_index.md>) *(Indexing)*
- [Clickstream Counting Pipeline](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/_index.md>) *(System Design Core)*
- [Advanced Aggregation](<../../../01_Databases_and_SQL/SQL_Language/Aggregation/Advanced_Aggregation.md>) *(Aggregation)*
- [Partitioning and Partition Pruning](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Partitioning_and_Partition_Pruning.md>) *(DBMS Concepts)*

## References

- *Hadoop: The Definitive Guide* — Tom White — HBase › Successful Service
- *Data Algorithms with Spark* — Mahmoud Parsian — **CHAPTER 7** › CHAPTER 7
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Operational Visibility › Excessive Health Checking

