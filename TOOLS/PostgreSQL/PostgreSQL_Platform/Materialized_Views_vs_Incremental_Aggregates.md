---
id: "TOOLS/PostgreSQL/PostgreSQL_Platform/Materialized_Views_vs_Incremental_Aggregates"
level: 4
type: "concept"
topic: "TOOLS/PostgreSQL/PostgreSQL_Platform"
tools: ["PostgreSQL"]
---

# Materialized Views vs Incremental Aggregates

**L4 Advanced** · [↑ PostgreSQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A materialized view stores a query's result and is refreshed wholesale on demand; an incremental aggregate (rollup table maintained by your own writes/jobs) updates only the changed slice — choose by freshness needs, data volume and how expensive a full recompute is.

### 1. The big picture
Dashboards and APIs repeat the same expensive aggregations (`SUM` by day by product over billions of rows). Computing them live is too slow; precomputing is the answer. PostgreSQL gives you one built-in tool — the **materialized view** (MV) — which is simple but has no native incremental refresh. When a full refresh becomes too slow or too stale, you build an **incremental aggregate**: a small summary table updated by a trigger, a scheduled upsert of only recent partitions, or a stream. Extensions like TimescaleDB (continuous aggregates) or `pg_ivm` provide automatic incremental maintenance.

### 2. Core concepts and vocabulary
- **View** — stored query, no data; always fresh, always recomputed.
- **Materialized view** — query result physically stored; refreshed with `REFRESH MATERIALIZED VIEW`.
- **`REFRESH … CONCURRENTLY`** — rebuilds in the background and applies a diff so readers aren't blocked; requires a unique index.
- **Rollup / summary table** — hand-maintained aggregate table keyed by the grouping (day, product).
- **Incremental maintenance** — processing only new/changed source rows.
- **Watermark** — high-water mark (timestamp/ID) of what has been aggregated.
- **Late-arriving data** — rows with old event time arriving after the watermark.
- **Idempotent upsert** — `INSERT … ON CONFLICT DO UPDATE` that can be rerun safely.

### 3. How it works, step by step
**Materialized view**
1. `CREATE MATERIALIZED VIEW mv AS SELECT …` runs the query and stores rows (with indexes you add).
2. Reads hit stored data — fast, but stale until refreshed.
3. `REFRESH MATERIALIZED VIEW mv` truncates and rebuilds under `ACCESS EXCLUSIVE` (blocks readers).
4. `REFRESH … CONCURRENTLY` builds a new copy, diffs against the old, applies inserts/deletes — readers continue; needs a unique index and is slower/more IO.

**Incremental aggregate**
1. Create `daily_sales(day, product_id, qty, revenue, PRIMARY KEY (day, product_id))`.
2. A job (cron/`pg_cron`/worker) selects source rows beyond the watermark, aggregates them, and upserts.
3. Advance the watermark in the same transaction.
4. Re-aggregate a trailing window (e.g. last 2 days) to absorb late data.

### 4. Worked example
```sql
-- Materialized view
CREATE MATERIALIZED VIEW mv_daily_sales AS
SELECT order_date::date AS day, product_id,
       sum(qty) AS qty, sum(qty * price) AS revenue
FROM order_items GROUP BY 1, 2;
CREATE UNIQUE INDEX ON mv_daily_sales (day, product_id);
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_daily_sales;

-- Incremental aggregate with idempotent upsert
CREATE TABLE daily_sales (
  day date, product_id bigint, qty bigint, revenue numeric,
  PRIMARY KEY (day, product_id));
CREATE TABLE agg_watermark (name text PRIMARY KEY, upto timestamptz);

BEGIN;
WITH w AS (SELECT upto FROM agg_watermark WHERE name = 'daily_sales' FOR UPDATE),
     src AS (
       SELECT order_ts::date AS day, product_id, sum(qty) AS qty, sum(qty*price) AS revenue
       FROM order_items, w
       WHERE order_ts >= w.upto - interval '2 days'      -- reprocess a trailing window
         AND order_ts <  date_trunc('day', now())        -- only complete days
       GROUP BY 1, 2)
INSERT INTO daily_sales
SELECT * FROM src
ON CONFLICT (day, product_id)
DO UPDATE SET qty = EXCLUDED.qty, revenue = EXCLUDED.revenue;   -- replace, not add -> idempotent
UPDATE agg_watermark SET upto = date_trunc('day', now()) WHERE name = 'daily_sales';
COMMIT;
```
Note the upsert **replaces** the recomputed day rather than adding to it; adding would double count on rerun. The reprocessed window must be recomputed from source, not incremented.

### 5. Under the hood   *(optional at this level)*
- An MV is a regular heap relation with a `pg_rewrite` rule recording its defining query; `REFRESH` (non-concurrent) swaps in a new file like `TRUNCATE`+insert. `CONCURRENTLY` creates a temp table, full-outer-joins old and new on the unique index, and applies deletes/inserts, which can leave dead tuples (needs vacuum).
- Neither plain nor concurrent refresh is incremental: cost ≈ the original query every time. `pg_ivm` and TimescaleDB continuous aggregates track deltas.
- Trigger-based aggregates (`AFTER INSERT … FOR EACH ROW UPDATE summary SET n = n + 1`) are exact and real-time but create a hot row per group → lock contention and per-row overhead; batch via statement-level triggers with transition tables or use periodic jobs.
- `UPDATE`/`DELETE` of source rows need reverse deltas (subtract) — or recompute the affected group.

### 6. Performance and design implications
- Index the MV for the access pattern (it's just a table); remember `CONCURRENTLY` needs a unique index without a `WHERE`.
- Refresh duration grows with source size → fine for daily; unacceptable for minute-level freshness on huge data.
- Partition the source by time and re-aggregate only affected partitions.
- Define freshness explicitly (SLA): "up to 15 minutes stale"; expose `refreshed_at`.
- Counter hot-spots: shard counters (`INSERT` deltas then periodically sum) to avoid single-row contention.
- Monitor refresh time and failures; a failed refresh keeps the old data (non-concurrent refresh is transactional).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE MATERIALIZED VIEW … [WITH NO DATA]` | define and populate | summaries |
| `REFRESH MATERIALIZED VIEW [CONCURRENTLY]` | rebuild | scheduled refresh |
| `pg_cron` / external scheduler | run refresh/upserts | automation |
| `INSERT … ON CONFLICT DO UPDATE` | idempotent rollup | incremental tables |
| Statement-level trigger + `REFERENCING NEW TABLE` | batch delta maintenance | near-real-time counters |
| `pg_ivm` / Timescale continuous aggregates | automatic incremental maintenance | when available |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Materialized view | full recompute takes longer than refresh interval |
| `REFRESH CONCURRENTLY` | no unique key, or you can afford a short lock |
| Incremental rollup table | logic must handle updates/deletes/late data — complexity |
| Trigger maintenance | write-heavy hot groups |
| Plain view / live query | data is small or freshness must be perfect |

### 9. Common mistakes
- Plain `REFRESH` during business hours → readers blocked → use `CONCURRENTLY` (with unique index).
- Rollup that adds on rerun → double counting → idempotent replace with recomputed window.
- Ignoring late-arriving or corrected data → permanently wrong totals → trailing reprocessing window or periodic full rebuild/reconcile.
- Watermark advanced outside the transaction → gaps or duplicates → same transaction.
- Treating an MV as always fresh → stale dashboards → show `refreshed_at`.

### 10. 🎯 Interview kit
**30-second answer:** "A materialized view caches a query and is refreshed in full — simple, stale between refreshes, no native incremental. For big data or tight freshness I maintain a rollup table incrementally with an idempotent upsert and a watermark, reprocessing a trailing window for late data."
**Structure a longer answer:** the problem → MV mechanics and limits → incremental design (watermark, idempotence, late data, deletes) → freshness/cost trade-offs → tools.
**Likely follow-ups:** *Why does `CONCURRENTLY` need a unique index?* → to diff rows. *Trigger vs batch?* → contention vs latency. *Handling deletes?* → subtract or recompute group. *Reconciliation?* → periodic full recompute and compare.
**Signals interviewers look for:** idempotence, late data, and explicit freshness requirements.

### 11. Check yourself
- [ ] Write an MV with `CONCURRENTLY` support and explain every requirement.
- [ ] Design a rollup that survives reruns and late events.
- [ ] Choose between MV, trigger and batch rollup for 500 writes/s and 1-minute freshness.
- [ ] Explain why a non-concurrent refresh blocks readers.
<!-- explainer:end -->

## Implementations (tools)

- [PostgreSQL](<../_index.md>)

## Practice questions

- [Intermediate] [You need to know whether a specific index is worth its maintenance cost; how do you combine pg_stat_user_indexes idx_scan counts with the write volume on the table to make a drop decision?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36250_You_need_to_know_whether_a_specific_index_is_worth_its_maint.md>)
- [Intermediate] [On the frontend you paginate a feed with OFFSET and it degrades as users scroll deep; how do you migrate to keyset pagination while keeping stable sort and handling ties?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35945_On_the_frontend_you_paginate_a_feed_with_OFFSET_and_it_degra.md>)
- [Intermediate] [How would you diagnose why a specific INSERT is slow when the table itself is small, considering trigger overhead, foreign-key checks, and index maintenance, and how do you attribute time to each?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36068_How_would_you_diagnose_why_a_specific_INSERT_is_slow_when_th.md>)
- [Deep] [After upgrading the OS, string sorting and unique index behavior changed and reindex warnings appeared; explain how a glibc collation version change can corrupt text indexes and how you'd remediate with REINDEX.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36275_After_upgrading_the_OS_string_sorting_and_unique_index_behav.md>)
- [Deep] [You need to run a data-fix that emits one summary log per statement not per row; how do statement-level triggers with transition tables (REFERENCING NEW TABLE) let you process the whole affected set at once?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36341_You_need_to_run_a_data-fix_that_emits_one_summary_log_per_st.md>)
- [Deep] [How does the LIMIT and OFFSET pagination work and why is OFFSET slow for large tables?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36362_How_does_the_LIMIT_and_OFFSET_pagination_work_and_why_is_OFF.md>)
- [Deep] [A COPY-based bulk import is slow and inflates WAL; walk through disabling indexes/constraints, using UNLOGGED tables, and the durability trade-offs of each speedup.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35991_A_COPY-based_bulk_import_is_slow_and_inflates_WAL_walk_throu.md>)
- [Deep] [A trigger that writes to an audit table on every UPDATE is slowing down a hot path; how would you measure the trigger's cost and decide between async logging, logical decoding, or accepting the trigger overhead?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36081_A_trigger_that_writes_to_an_audit_table_on_every_UPDATE_is_s.md>)

## Related concepts

- [Index-only Scans](<../../../01_Databases_and_SQL/Query_Performance/Indexing/Index-only_Scans.md>) *(Indexing)*
- [Large Values and TOAST](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Large_Values_and_TOAST.md>) *(SQL Fundamentals)*
- [Statistics and Sargability](<../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Statistics_and_Sargability.md>) *(Execution Plans)*

## References

- *Hadoop: The Definitive Guide* — Tom White — HBase › Counters
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Forwarded Fetch

