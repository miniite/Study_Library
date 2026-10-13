---
type: "practice"
difficulty: "Expert"
---

# Your report query aggregates millions of rows and times out. How would you make it faster?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Reading EXPLAIN](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Reading_EXPLAIN.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: A reporting/analytics query that scans and aggregates millions of rows is timing out. The interviewer wants a structured, senior-level diagnostic-and-fix methodology — not one silver-bullet trick. This is a "expertise" level SQL question aimed at Backend/Fullstack SDE-2/3, so the expected answer covers: how you measure, why it's slow, and the ordered toolbox of fixes (indexing, query rewrite, pre-aggregation, partitioning, resource/isolation, and offloading to the right engine).

What they're probing: Do you jump to "add an index" blindly, or do you first read the EXPLAIN plan, understand the access path, and pick the fix that matches the actual bottleneck? Can you distinguish an OLTP tuning problem from a "this is really an OLAP workload on the wrong engine" problem?

Clarifying questions I'd ask up front:

Is this on the primary OLTP DB (Postgres/MySQL) or a warehouse (Snowflake/Redshift/BigQuery)?
Is the report real-time/on-demand or scheduled? Can results be stale by minutes/hours?
What's the timeout — 30s API gateway limit, or the DB statement_timeout?
Is it one heavy query, or many concurrent report runs competing for resources?
How selective is the filter — does the report touch all millions of rows or a small slice?

Plan: measure → interpret the plan → apply the cheapest effective fix first, escalate only if needed.

Step 1 — Measure before you change anything

Run EXPLAIN (ANALYZE, BUFFERS) (Postgres) / EXPLAIN ANALYZE (MySQL 8) / the query profile (Snowflake). I'm looking for:

Seq Scan / full table scan on a large table where a filtered index would help.
Rows estimated vs. actual wildly diverging → stale statistics; the planner chose a bad plan. Fix with ANALYZE/ANALYZE TABLE.
Nested Loop over millions of rows where a Hash Join would be far cheaper.
Sort / Hash spilling to disk (external merge, temp files) → work_mem too small.
The single most expensive node — optimize that, not everything.

This one step avoids the classic mistake of "adding indexes" that never get used.

Step 2 — Indexing (the first real lever)

Aggregations usually filter then group. The high-leverage moves:

Index the WHERE/JOIN columns, most-selective first. For a report filtered by tenant_id and created_at BETWEEN, a composite (tenant_id, created_at) lets the DB range-scan just the relevant slice instead of the whole table.
Covering index including the aggregated column so the query is answered from the index alone (index-only scan): e.g. (tenant_id, created_at) INCLUDE (amount). No heap fetch = huge win.
Partial index if the report only ever looks at a subset: WHERE status = 'COMPLETED'.
Make sure predicates are sargable — WHERE created_at >= '2026-01-01' uses an index; WHERE DATE(created_at) = ... or WHERE amount * 1.1 > x does not.
Step 3 — Rewrite the query
Push filters down / reduce the working set before the join and aggregation. Aggregate in a subquery, then join, so you group millions once, not per-joined-row.
Kill correlated subqueries and per-row function calls; replace with a single JOIN + GROUP BY or a window function.
Avoid SELECT * and unnecessary DISTINCT/ORDER BY in the report that the client doesn't need.
EXISTS instead of IN (subquery) when checking presence.
Step 4 — Stop recomputing: pre-aggregate

If the same expensive aggregate is read far more than the data changes, don't compute it live:

Materialized view refreshed on a schedule (REFRESH MATERIALIZED VIEW CONCURRENTLY). The report then reads a tiny pre-summed table in milliseconds.
Summary/rollup table maintained incrementally (nightly batch or on write) — e.g. daily_sales(tenant_id, day, total). The report reads thousands of rows, not millions.
This trades freshness for latency, which is exactly the right trade for most reports.
Step 5 — Partitioning & scale-out

For genuinely huge tables:

Range partition by time (created_at) so the report only scans the relevant partitions (partition pruning) and old partitions can be archived.
On a warehouse (Snowflake/BigQuery), this becomes clustering keys / partition columns so micro-partitions are pruned.
Step 6 — Resources, isolation & the right engine
Bump work_mem (session-scoped) so sorts/hashes stay in RAM.
Run heavy reports on a read replica, not the primary, so analytics don't contend with OLTP traffic.
If it's fundamentally an OLAP workload, move it to a columnar/MPP engine (Snowflake, Redshift, BigQuery, ClickHouse). A row-store OLTP DB is the wrong tool for scanning millions of rows for aggregates; a columnar store reads only the needed columns and is parallel by design. Recognizing this is the senior-level insight.
Summary — what a strong answer emphasizes

Start by reading the plan, not guessing. Then walk the ladder cheapest-first: statistics → indexing (composite/covering/partial, sargable predicates) → query rewrite → pre-aggregation (materialized views/rollups) → partitioning → resources/replica → and, if it's truly OLAP, move it off the OLTP DB. Always name the trade-off (freshness vs. latency, write cost vs. read cost).

#### Common mistakes
blindly adding indexes without checking they're used; over-indexing (slows writes); non-sargable predicates; solving on the primary and starving OLTP; and treating an analytics workload as an OLTP tuning problem instead of moving it to a columnar warehouse.

**Comparison:**

```
Technique	When it helps	Trade-off / cost
Read EXPLAIN plan + ANALYZE stats	Always, first step	None — diagnosis only
Composite / covering / partial index	Selective filters, index-only scan	Slows writes; disk; over-indexing risk
Sargable predicate rewrite	Function-wrapped columns disable indexes	Requires query change
Query rewrite (filter down, hash join, no correlated subquery)	Bad join order / per-row work	Dev effort
Materialized view / rollup table	Read >> write; staleness OK	Data is stale until refresh
Range partitioning + pruning	Very large time-series tables	Maintenance, partition management
Read replica / bump work_mem	Reports contend with OLTP; disk spills	Extra infra; replica lag
Move to columnar/MPP (Snowflake, Redshift, ClickHouse)	True OLAP scan-and-aggregate	New system; ETL/ingest pipeline
```

**Code:**

```sql
-- 1) Read the actual plan first
EXPLAIN (ANALYZE, BUFFERS)
SELECT tenant_id, SUM(amount) AS total, COUNT(*) AS cnt
FROM orders
WHERE created_at >= '2026-01-01' AND created_at < '2026-02-01'
GROUP BY tenant_id;

-- 2) Refresh planner statistics if estimates are off
ANALYZE orders;

-- 3) Composite, covering index -> index-only scan for the report
CREATE INDEX idx_orders_report
  ON orders (tenant_id, created_at) INCLUDE (amount);

-- 4) Pre-aggregate: incremental daily rollup table read by the report
CREATE TABLE daily_sales (
  tenant_id BIGINT,
  day       DATE,
  total     NUMERIC,
  cnt       BIGINT,
  PRIMARY KEY (tenant_id, day)
);

-- Or a materialized view refreshed on schedule (non-blocking reads)
CREATE MATERIALIZED VIEW mv_daily_sales AS
SELECT tenant_id, date_trunc('day', created_at)::date AS day,
       SUM(amount) AS total, COUNT(*) AS cnt
FROM orders
GROUP BY 1, 2;
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_daily_sales;

-- 5) Range partition huge tables so pruning skips irrelevant data
CREATE TABLE orders (
  id BIGINT, tenant_id BIGINT, amount NUMERIC, created_at TIMESTAMPTZ
) PARTITION BY RANGE (created_at);
```

**Example input / output:**

```text
-- BEFORE (slow): full sequential scan over ~12M rows, timeout at 30s
EXPLAIN ANALYZE -> Seq Scan on orders  (rows=12,000,000)
                   Sort  (external merge, temp files: 900MB)
Total: 41,200 ms  (killed by statement_timeout)

-- AFTER adding covering composite index + sargable range predicate:
EXPLAIN ANALYZE -> Index Only Scan using idx_orders_report
                   (rows scanned=310,000 for the month)
Total: 220 ms

-- AFTER reading from the daily rollup / materialized view:
SELECT tenant_id, SUM(total) FROM daily_sales
WHERE day >= '2026-01-01' GROUP BY tenant_id;
Total: 8 ms   (reads ~3,000 pre-summed rows instead of 12M)
```

**Flow:**

1. Full scan / bad estimate
2. Bad join / per-row work
3. Recomputing sameaggregate
4. Huge time-series table
5. Contends with OLTP
6. No, it's OLAP
7. Yes
8. Report query times out
9. EXPLAIN ANALYZE + checkstats
10. Bottleneck?
11. ANALYZE +composite/covering index,sargable predicates
12. Rewrite: filter down, hash join,window fn
13. Materialized view / rollup table
14. Range partition + pruning
15. Read replica / raise work_mem
16. Fast enough?
17. Move to columnar/MPPwarehouse
18. Done

