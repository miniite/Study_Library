---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/MVCC_Bloat_and_Vacuum"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: []
---

# MVCC Bloat and Vacuum

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** MVCC databases never overwrite a row in place — updates and deletes leave old row versions behind — so *vacuum* (or garbage collection) must reclaim them, and when it falls behind you get bloat, slow scans, slow counts and, eventually, transaction-ID wraparound risk.

### 1. The big picture
Multi-Version Concurrency Control lets readers and writers work without blocking each other: every transaction sees a consistent snapshot, and an `UPDATE` creates a *new version* of the row while the old one stays visible to older snapshots. That is great for concurrency, but old versions are garbage the moment no snapshot can see them. Something has to clean up.

In PostgreSQL that is `VACUUM`/autovacuum; in SQL Server (row-versioning) it is background cleanup of the version store in `tempdb`; in InnoDB it is the *purge* thread over undo logs. Understanding this explains many production symptoms: "row count unchanged but table doubled", "`COUNT(*)` takes 30 s", "autovacuum runs forever", "disk keeps growing".

### 2. Core concepts and vocabulary
- **MVCC** — keep multiple versions of rows so readers see snapshots and don't block writers.
- **Tuple / row version** — one physical copy of a row. An update = mark old version dead + insert new version.
- **`xmin` / `xmax`** (PostgreSQL) — transaction IDs that created/deleted a tuple; visibility is computed from them and the snapshot.
- **Dead tuple** — a version no running transaction can see anymore; reclaimable.
- **Bloat** — table/index space occupied by dead tuples and unused gaps.
- **VACUUM** — marks dead-tuple space reusable and updates the visibility map and free-space map; does not normally shrink the file.
- **VACUUM FULL** — rewrites the table compactly; exclusive lock.
- **Autovacuum** — background workers that vacuum/analyze tables past thresholds.
- **HOT update** — heap-only tuple update; no index entry changes when no indexed column changes and there's room on the page.
- **Visibility map** — per-page bits for "all tuples visible", enabling index-only scans and skipping pages.
- **Freeze / wraparound** — 32-bit transaction IDs are reused; old tuples must be *frozen* to avoid becoming "invisible".
- **Long-running transaction** — pins the oldest snapshot, preventing cleanup of anything newer.

### 3. How it works, step by step
1. `UPDATE t SET x = 5 WHERE id = 1`: PG writes a new tuple version in the heap (ideally the same page, as HOT), sets `xmax` on the old one, and (if indexed columns changed) adds new index entries.
2. Concurrent snapshots keep reading the old version until their transaction ends.
3. After the last such snapshot ends, the old version is **dead**.
4. Autovacuum triggers when `dead_tuples > autovacuum_vacuum_threshold + scale_factor × reltuples` (defaults: 50 + 20 % of the table).
5. VACUUM scans the heap (using the visibility map to skip all-visible pages), removes dead index entries, marks heap space reusable in the free-space map, and freezes old tuples.
6. New inserts/updates reuse that space. The file size stays, so total size doesn't drop — but growth stops.

### 4. Worked example
```sql
-- How much dead stuff, and when was it vacuumed?
SELECT relname, n_live_tup, n_dead_tup,
       round(100.0*n_dead_tup/greatest(n_live_tup+n_dead_tup,1),1) AS dead_pct,
       last_autovacuum, last_autoanalyze
FROM   pg_stat_user_tables
ORDER  BY n_dead_tup DESC LIMIT 10;

-- Long transactions that block cleanup
SELECT pid, now()-xact_start AS age, state, left(query,60)
FROM   pg_stat_activity
WHERE  xact_start IS NOT NULL ORDER BY xact_start LIMIT 5;

-- Per-table tuning for a high-churn queue table
ALTER TABLE job_queue SET (
  autovacuum_vacuum_scale_factor = 0.01,
  autovacuum_vacuum_threshold    = 1000,
  autovacuum_vacuum_cost_limit   = 2000,
  fillfactor = 80);              -- leave room for HOT updates

-- Reclaim space without a long lock (extension)
-- pg_repack -t job_queue mydb
```
Reading: `dead_pct` above ~20 % on a hot table means autovacuum is behind. The oldest `xact_start` explains *why* (an idle-in-transaction session from a BI tool, an abandoned replication slot, or a prepared transaction holds the horizon back). After a mass `UPDATE`, sequential scans slow even though the row count is unchanged: the table physically contains old + new versions, so the scan reads roughly twice the pages.

### 5. Under the hood
- **Why `COUNT(*)` is slow in MVCC.** There's no single "row count": each row's visibility depends on the snapshot, so the engine must visit tuples (or at least visibility-map-covered pages via an index-only scan) to count what *this* transaction can see. Options: exact (slow, correct), estimate from `pg_class.reltuples` (fast, approximate), or maintained counter (trigger/summary table — exact but a write hotspot; use per-shard counters or periodic aggregation).
- **Index bloat** is separate: dead index entries are removed by vacuum, but B-tree pages emptied by deletes aren't always merged. `REINDEX CONCURRENTLY` fixes it.
- **Freeze and wraparound.** XIDs are 32-bit (~2 billion usable). `autovacuum_freeze_max_age` (default 200 M) forces an aggressive anti-wraparound vacuum. If it can't finish, PG eventually refuses writes to protect data — a notorious outage.
- **The horizon.** Cleanup can only remove versions older than the oldest snapshot across all backends, replication slots (`backend_xmin`, `catalog_xmin`), hot-standby feedback, and prepared transactions.
- **Other engines.** InnoDB keeps the newest version in the clustered index and old versions in undo logs; a long transaction makes the history list grow and purge lag. SQL Server's version store in tempdb (or the PVS with ADR) grows with long snapshot transactions; check `sys.dm_tran_version_store_space_usage` and `sys.dm_tran_active_snapshot_database_transactions`. Oracle uses undo segments (`ORA-01555` snapshot too old).
- **Optimistic concurrency** (application-level) complements MVCC: add a `version` column, `UPDATE … WHERE id=? AND version=?`, check rows affected; on 0 rows return HTTP 409 and have the client re-read and retry.

### 6. Performance and design implications
- Avoid unnecessary updates: `UPDATE t SET x = x` still creates a new version; add `WHERE x IS DISTINCT FROM new`.
- Keep transactions short; set `idle_in_transaction_session_timeout` and `statement_timeout`.
- Put frequently updated columns in a separate narrow table (counters, last_seen) so index and row bloat don't hit the main table.
- Lower `fillfactor` for update-heavy tables to enable HOT updates.
- Partition and drop old partitions instead of mass `DELETE` — no dead tuples, instant.
- Monitor: `pg_stat_user_tables`, `pgstattuple`/`pgstatindex`, `pg_stat_progress_vacuum`, wraparound age `age(datfrozenxid)`.
- Autovacuum throttling (`autovacuum_vacuum_cost_delay`, cost limit) trades I/O impact for speed; on big, hot tables raise the limit and add workers.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `VACUUM (VERBOSE, ANALYZE) t` | Reclaim + refresh stats | Manual cleanup after bulk changes |
| `VACUUM FULL t` | Rewrite compact; exclusive lock | Offline windows only |
| `pg_repack` | Online rewrite | Bloat in production |
| `REINDEX CONCURRENTLY` | Rebuild bloated index | Index bloat |
| `autovacuum_vacuum_scale_factor` | % trigger | Lower for large hot tables |
| `idle_in_transaction_session_timeout` | Kill idle txns | Protect the horizon |
| `fillfactor` | Page free space | HOT updates |
| `pgstattuple` | Measure bloat | Before/after |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Plain `VACUUM` (autovacuum) | Table already hugely bloated (won't shrink file) |
| `pg_repack` | No spare disk (needs ~2× table size) |
| `VACUUM FULL` | Table is hot — takes an exclusive lock |
| Partition drop for retention | Queries span all partitions without pruning |
| Estimated counts | Exact count is business-critical |
| Counter table/trigger | High write concurrency on one counter row |

### 9. Common mistakes
- Leaving sessions idle in transaction → vacuum can't remove anything → set timeouts, fix connection pooling.
- Mass `UPDATE`/`DELETE` in one go → huge dead-tuple spike → batch + vacuum between batches.
- Disabling autovacuum to "save I/O" → bloat then wraparound emergency → tune, never disable.
- Expecting `VACUUM` to shrink files → disk still full → `pg_repack`/`VACUUM FULL` or partition drops.
- `COUNT(*)` on dashboards of huge tables → seconds per call → estimates or maintained summaries.

### 10. 🎯 Interview kit
**30-second answer:** MVCC keeps old row versions so readers don't block writers. Updates and deletes leave dead versions; vacuum reclaims space once no snapshot needs them. If it lags — often due to long transactions — you get bloat, slow scans and eventually wraparound risk. Tune autovacuum per table, keep transactions short, and use pg_repack or partitioning for cleanup.
**Structure a longer answer:** snapshots and versions → what dead tuples are → vacuum mechanism → why it falls behind → diagnosis (stats, oldest xact) → remediation → prevention.
**Likely follow-ups:**
- *Why is COUNT(*) slow?* → must check visibility per row; use estimates or counter tables.
- *Scans slowed after a big update?* → table holds both versions; vacuum/repack.
- *Autovacuum can't keep up?* → per-table scale factor/threshold, cost limit, more workers, find horizon blockers.
- *MVCC vs locking?* → MVCC trades storage and cleanup for non-blocking reads.
**Signals interviewers look for:** names the concrete views/settings, understands horizon and long transactions, knows `VACUUM` vs `VACUUM FULL`.

### 11. Check yourself
- [ ] Describe what happens physically on an `UPDATE` in PostgreSQL.
- [ ] List four things that can hold back the cleanup horizon.
- [ ] Write the query that shows dead-tuple percentage per table.
- [ ] Explain why `VACUUM` doesn't shrink the file but stops growth.
- [ ] Implement optimistic concurrency with a version column, including the retry loop.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [Walk me through implementing optimistic concurrency control with a version column end-to-end, including how the frontend surfaces a 409 conflict and how the retry loop avoids lost updates.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40302_Walk_me_through_implementing_optimistic_concurrency_control_.md>)
- [Deep] [Explain why COUNT(*) on a 200M-row MVCC table takes 30 seconds, and the trade-offs of exact versus estimated versus trigger-maintained counters.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39921_Explain_why_COUNT_on_a_200M-row_MVCC_table_takes_30_seconds_.md>)
- [Deep] [You're asked why a SELECT COUNT(*) on a 200M-row table takes 30 seconds. Explain why counts are expensive in MVCC databases and the trade-offs of exact vs. estimated vs. maintained counters.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40303_You_re_asked_why_a_SELECT_COUNT_on_a_200M-row_table_takes_30.md>)
- [Expert] [Explain MVCC bloat: after a large UPDATE, sequential scans slowed even though row count is unchanged. How do you confirm dead-tuple bloat and remediate without locking the table?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39892_Explain_MVCC_bloat_after_a_large_UPDATE_sequential_scans_slo.md>)
- [Expert] [Autovacuum keeps falling behind on a high-churn table. How would you tune per-table autovacuum settings, and when is pg_repack or a scheduled aggressive VACUUM the right call?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39893_Autovacuum_keeps_falling_behind_on_a_high-churn_table_How_wo.md>)
- [Expert] [What is MVCC, and how does it let readers not block writers in databases like PostgreSQL?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40262_What_is_MVCC_and_how_does_it_let_readers_not_block_writers_i.md>)
- [Expert] [What is table bloat / dead tuples in an MVCC database and why does VACUUM matter?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40263_What_is_table_bloat_dead_tuples_in_an_MVCC_database_and_why_.md>)
- [Expert] [You observe that autovacuum keeps falling behind on a high-churn table, causing bloat and slow scans. How do you tune per-table autovacuum settings, and when is a scheduled aggressive VACUUM or pg_repack the right call?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40416_You_observe_that_autovacuum_keeps_falling_behind_on_a_high-c.md>)

## Related concepts

- [MVCC](<../../Transactions_and_Concurrency/Locking/MVCC.md>) *(Locking)*
- [Batching Writes](<../../Database_Internals/DBMS_Concepts/Batching_Writes.md>) *(DBMS Concepts)*
- [B-tree Concurrency and Latches](<../../Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/B-tree_Concurrency_and_Latches.md>) *(Storage Engines)*

