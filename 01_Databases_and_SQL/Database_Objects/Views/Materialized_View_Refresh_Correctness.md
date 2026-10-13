---
id: "01_Databases_and_SQL/Database_Objects/Views/Materialized_View_Refresh_Correctness"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Objects/Views"
tools: []
---

# Materialized View Refresh Correctness

**L4 Advanced** · [↑ Views](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A materialized view is only as trustworthy as its refresh: choose full vs incremental and synchronous vs deferred deliberately, make refreshes atomic and non-blocking, and detect drift with explicit reconciliation, because a refresh that is slow, partial or silently wrong costs more than the query it replaced.

### 1. The big picture
Once you accept a materialized view (MV) you inherit three operational questions: *how* does it get refreshed, *what do readers see while that happens*, and *how do I know it is correct?* Most MV incidents are not "the MV was slow" but "the refresh blocked reports", "the MV was empty at 9 a.m.", "the totals were wrong after a delete", or "the logs grew forever".

This lesson is about refresh semantics, locking, atomicity, consistency across several MVs, drift detection, and capacity planning of refresh windows. It assumes the trade-off lesson (when to use an MV at all).

### 2. Core concepts and vocabulary
- **Complete (full) refresh** — truncate/replace and rerun the defining query. Simple, always right, cost grows with data.
- **Incremental (fast) refresh** — apply only changed rows. Cheap, but restricted by SQL shape and needs change capture.
- **Append-only incremental** — refresh by processing only new rows; valid when history never changes.
- **Merge/upsert refresh** — recompute only the groups touched by new/changed rows and replace them.
- **`REFRESH … CONCURRENTLY` (PostgreSQL)** — builds a new copy, diffs against the old, applies row changes; readers are not blocked; requires a unique index.
- **Refresh log (Oracle MV log)** — change records on base tables used by fast refresh; must be purged by refresh.
- **Atomic swap** — build into a new object, then switch name/synonym/partition in a single metadata operation.
- **Staleness contract** — the documented maximum age of data and how consumers can see it.
- **Drift** — MV content diverges from what the defining query would return now.
- **Late-arriving data** — rows that belong to an already-aggregated period.

### 3. How it works, step by step
**Choosing the refresh method**
1. Estimate refresh cost: query cost × frequency. If a full refresh fits the window comfortably *and* data volume grows slowly, prefer full: least logic, least risk.
2. If data is append-only and periods are closed, use incremental by new partition/date.
3. If rows are updated/deleted, use change capture (logs/CDC/Change Tracking) and merge affected groups, or accept full refresh.
4. Pick the trigger: on commit (write amplification), on a schedule (staleness bounded by period), on event (after the load job).

**Making a refresh safe**
1. Build the new data *beside* the live data (temp table/partition/shadow MV).
2. Validate (row counts, sums, non-empty, key uniqueness).
3. Swap atomically (rename inside a transaction, partition switch, `REFRESH CONCURRENTLY`).
4. Record refresh metadata (start, end, rows, source watermark).
5. Expose the watermark to consumers.

### 4. Worked example
```sql
-- PostgreSQL: non-blocking refresh needs a unique index on all rows
CREATE MATERIALIZED VIEW mv_daily_revenue AS
  SELECT order_date::date AS d, region, SUM(amount) AS revenue
  FROM orders GROUP BY 1,2;
CREATE UNIQUE INDEX ux_mv_daily_revenue ON mv_daily_revenue (d, region);

REFRESH MATERIALIZED VIEW CONCURRENTLY mv_daily_revenue;
-- Without the unique index: "cannot refresh materialized view concurrently".
-- Without CONCURRENTLY: ACCESS EXCLUSIVE lock; reports queue until it finishes.
```
Under the hood, `CONCURRENTLY` computes the new result into a temporary table, does a full outer join against the old rows using the unique index, and issues `INSERT`/`DELETE` only for the differences. Readers see the old snapshot until the refresh transaction commits; then they see the new one — atomic from the reader's perspective.

**Atomic group refresh for a nightly report (SQL Server pattern)**
```sql
-- Build shadow tables, validate, then swap names in one transaction
SELECT ... INTO rpt.DailyRevenue_new  FROM ...;
SELECT ... INTO rpt.DailyByRegion_new FROM ...;
-- validate: row counts > 0, SUM(revenue) matches base within tolerance
BEGIN TRAN;
  EXEC sp_rename 'rpt.DailyRevenue',  'DailyRevenue_old';
  EXEC sp_rename 'rpt.DailyRevenue_new', 'DailyRevenue';
  EXEC sp_rename 'rpt.DailyByRegion', 'DailyByRegion_old';
  EXEC sp_rename 'rpt.DailyByRegion_new', 'DailyByRegion';
COMMIT;
```
Both reports flip together; a reader never sees one new and one old. (Alternatives: a single "snapshot id" column that every MV carries and a pointer table that readers join to; or partition switching.)

**Drift detection**
```sql
-- Compare the MV with the definition for a closed period
SELECT m.d, m.region, m.revenue AS mv, b.revenue AS base
FROM mv_daily_revenue m
FULL JOIN (SELECT order_date::date d, region, SUM(amount) revenue
           FROM orders WHERE order_date < date_trunc('day', now())
           GROUP BY 1,2) b USING (d, region)
WHERE m.revenue IS DISTINCT FROM b.revenue;
```
Run it after bulk loads and on a schedule; any rows returned mean drift → force a complete refresh.

### 5. Under the hood
- **Why fast refresh silently becomes complete (Oracle).** Fast refresh requires an MV log containing all columns referenced, `ROWID`/primary key, and `INCLUDING NEW VALUES` for aggregates; certain constructs (outer joins without conditions, non-deterministic functions, set operators, `DISTINCT` with some aggregates) or *direct-path loads* without logging invalidate the incremental path. Check `DBA_MVIEWS.REFRESH_MODE`/`LAST_REFRESH_TYPE` and use `DBMS_MVIEW.EXPLAIN_MVIEW`.
- **Unbounded MV logs.** Logs are purged only after *all* dependent MVs have refreshed past a change. An abandoned or failing MV pins the log, and every base-table DML pays to write a log row — growing slowdown. Fix: refresh or drop the stale MV, or purge deliberately (`PURGE_LOG`), then complete-refresh.
- **Deletes and wrong aggregates.** MIN/MAX cannot be maintained incrementally after deletes without rescanning; sums need the count (`COUNT(*)` companion). If the MV omits the count/log columns, deletes may leave wrong aggregates — validate with a drift query.
- **Snapshot consistency.** A report reading two MVs at `READ COMMITTED` can read one before and one after a refresh. Isolation levels do not fix this because the refreshes are separate transactions; use an atomic swap, a snapshot-id design, or read both inside one `REPEATABLE READ`/snapshot transaction *and* refresh them in one transaction.
- **Incremental pitfalls with grain.** If the MV is by day but source rows are timestamps and loads arrive mid-day, a refresh must *recompute the whole current day* (delete day, reinsert), not just append — otherwise the day is double counted or incomplete.
- **Refresh failure halfway.** A truncate-and-insert refresh outside one transaction leaves an empty table; wrap in a transaction (PostgreSQL DDL is transactional; SQL Server `TRUNCATE` is too) or use shadow-and-swap.

### 6. Performance and design implications
- Refresh cost scales with the *defining query*; index base tables for it. Check its plan like any ETL query.
- Schedule refreshes away from peak I/O; measure IOPS/CPU before/during/after; consider resource governance (Resource Governor, workload classes).
- Partition large MVs by period and refresh only the open partition.
- Frequency math: if tolerated lag is 15 minutes and a refresh takes 4 minutes, refresh every 10 minutes (refresh duration + safety margin < lag); if the refresh approaches the interval, switch to incremental.
- `ON COMMIT` refresh multiplies write cost; use only for small, simple MVs on low-write tables.
- Handle failures with retries *and* alerts; retries alone can hide chronic failure.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `REFRESH MATERIALIZED VIEW [CONCURRENTLY]` | PostgreSQL refresh | Needs unique index for concurrent |
| `DBMS_MVIEW.REFRESH(mv, 'F'/'C'/'?')` | Oracle fast/complete/force | Scheduled jobs |
| `DBMS_MVIEW.EXPLAIN_MVIEW` | Why fast refresh is/ isn't possible | Diagnosis |
| `CREATE MATERIALIZED VIEW LOG … WITH …` | Oracle change log | Fast refresh prerequisites |
| `sp_rename` / `ALTER SCHEMA … TRANSFER` / partition `SWITCH` | Atomic swap | SQL Server shadow-and-swap |
| `pg_matviews` / `DBA_MVIEWS` | Metadata | Inventory, last refresh |
| Watermark table | Records source position/time | Staleness contract |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Full refresh | Data small or window large; simplicity beats cost; you cannot prove incremental correctness |
| Incremental (append) | History immutable, clear partition/date key |
| Incremental (change capture) | Updates/deletes matter and logging is guaranteed for every load path |
| `CONCURRENTLY` / swap | Readers must never block or see empties; you can afford extra I/O and a unique key |
| `ON COMMIT` | Tiny, cheap MV on low-write table |
| Scheduled | Staleness tolerance > refresh time |

### 9. Common mistakes
- Refresh outside a transaction → empty/partial MV during business hours → build-then-swap.
- Missing unique index → `CONCURRENTLY` fails → add a unique index covering all rows (all columns that identify a row).
- Bulk load bypasses logging → MV silently out of sync → reconcile and complete-refresh; document load paths.
- Staleness not communicated → wrong decisions → expose `last_refreshed_at`.
- Ignoring late-arriving data → closed periods wrong → reprocess a trailing window (e.g. last 3 days).
- Refreshing multiple MVs separately → inconsistent report → atomic group refresh or snapshot id.
- Migrating to incremental without validation → silent errors → run both in parallel, diff, then cut over.

### 10. 🎯 Interview kit
**30-second answer:** Refresh is the correctness boundary of an MV. Full is simple and safe; incremental is cheaper but needs change capture and restricted SQL. To avoid blocking and partial states, build beside and swap atomically, or use `CONCURRENTLY` with a unique index; verify with drift queries and expose last-refresh time.

**Structure a longer answer:** clarify (engine, freshness SLA, update pattern) → choose refresh model → make it atomic and non-blocking → validate/monitor → capacity and scheduling.

**Likely follow-ups:**
- *`CONCURRENTLY` failed — why?* → no unique index covering the rows (and MV must be populated); trade-off: more work and I/O, no reader blocking.
- *Non-concurrent refresh lock semantics?* → takes an exclusive lock for the duration; readers block; schedule off-peak or use concurrent.
- *Logs growing, fast refresh not purging?* → an MV is not refreshing past the log; find stale MVs, fix, complete-refresh, monitor.
- *Refresh failed halfway and left it empty?* → transactional refresh or shadow-and-swap; validation gate.
- *Detect drift after bulk load?* → compare aggregates for closed periods; force rebuild.
- *Consistent snapshot across MVs?* → atomic swap or snapshot id; isolation alone is insufficient.

**Signals interviewers look for:** you separate "fast" from "correct", you know that isolation does not fix multi-MV consistency, and you plan validation and monitoring.

### 11. Check yourself
- [ ] Explain why `REFRESH CONCURRENTLY` needs a unique index and what it costs.
- [ ] Design an atomic refresh of three related MVs.
- [ ] List three reasons a fast refresh falls back to complete.
- [ ] Compute a refresh interval for a 15-minute freshness SLA and 6-minute refresh.
- [ ] Write a drift-detection query and say when you would run it.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [How do you refresh a materialized view and what is the difference between complete and incremental refresh?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Views/Q-43910_How_do_you_refresh_a_materialized_view_and_what_is_the_diffe.md>)
- [Deep] [What is the syntax to refresh a materialized view in PostgreSQL, and what does CONCURRENTLY do?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Views/Q-43975_What_is_the_syntax_to_refresh_a_materialized_view_in_Postgre.md>)
- [Deep] [A materialized view with FAST refresh suddenly started doing COMPLETE refreshes in production; what would you check to find why the incremental path was invalidated?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Views/Q-43847_A_materialized_view_with_FAST_refresh_suddenly_started_doing.md>)
- [Deep] [How do you decide between refreshing a materialized view ON COMMIT versus on a schedule, given the write amplification cost on the base table? Give a load scenario.](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Views/Q-43853_How_do_you_decide_between_refreshing_a_materialized_view_ON_.md>)
- [Deep] [A materialized view join is stale by up to an hour and a dashboard shows wrong totals during that window; how do you communicate and design around this staleness contract?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Views/Q-43859_A_materialized_view_join_is_stale_by_up_to_an_hour_and_a_das.md>)
- [Deep] [How do you decide the refresh frequency of a materialized view when the base data updates continuously but the report tolerates 15-minute lag? Walk through the SLA math.](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Views/Q-43874_How_do_you_decide_the_refresh_frequency_of_a_materialized_vi.md>)
- [Deep] [A view is used in a hot path and you see repeated recompilation; explain how view definition changes invalidate cached plans and how to minimize churn.](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Views/Q-43887_A_view_is_used_in_a_hot_path_and_you_see_repeated_recompilat.md>)
- [Deep] [A materialized view got out of sync with base tables after a bulk load bypassed logging; how do you detect the drift and force a correct rebuild?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Views/Q-43905_A_materialized_view_got_out_of_sync_with_base_tables_after_a.md>)

## Related concepts

- [Materialized View Refresh and Maintenance](<../Materialized_Views/Materialized_View_Refresh_and_Maintenance.md>) *(Materialized Views)*

## References

- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Storage Design Patterns › Consequences
- *Data Pipelines Pocket Reference* — James Densmore — Transforming Data › Modeling Append-Only Data

