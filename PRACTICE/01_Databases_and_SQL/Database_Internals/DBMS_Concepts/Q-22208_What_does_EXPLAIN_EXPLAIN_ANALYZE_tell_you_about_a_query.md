---
type: "practice"
difficulty: "Deep"
---

# What does EXPLAIN / EXPLAIN ANALYZE tell you about a query?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1 · [↑ Views and Materialized Views](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Views_and_Materialized_Views.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: "What does EXPLAIN / EXPLAIN ANALYZE tell you about a query?" The interviewer wants to know whether I can read a query plan — the single most important skill for database performance work. They're probing: do I understand that the database has a query planner/optimizer that chooses how to execute my SQL; do I know the difference between EXPLAIN (estimate, no execution) and EXPLAIN ANALYZE (real run, real timings); and can I name and interpret the concrete things a plan reveals — access methods, join strategies, row estimates, costs, and where the time actually goes.

Clarifying question: Which engine? Postgres and MySQL both have EXPLAIN but different output. I'll answer primarily for Postgres (richest output) and note MySQL differences.

Plan: define what EXPLAIN is, contrast the two modes, then enumerate exactly what the plan tells me and how I act on each signal, with a worked example.

What EXPLAIN is

SQL is declarative — I say what rows I want, not how to get them. The query optimizer turns that into an execution plan: a tree of physical operators (scans, joins, sorts, aggregates) it believes is cheapest. EXPLAIN shows me that plan. Critically, EXPLAIN alone does not run the query — it prints the planner's estimated costs and row counts based on table statistics. It's cheap and safe (even for a DELETE).

EXPLAIN ANALYZE actually executes the query, then annotates each node with real measurements: actual time, actual rows, and loop counts. This lets me compare estimate vs. reality, which is where most performance bugs hide. Caveat: because it runs the query, it has side effects for writes — I wrap those in a transaction and ROLLBACK, or use it only on SELECTs.

What the plan tells me — node by node

1. Access method (how each table is read).

Seq Scan / Full Table Scan — read every row. Fine for small tables or when most rows match; a red flag on a big table with a selective filter.
Index Scan — walk an index, then fetch matching rows from the heap/table.
Index Only Scan (Postgres) — answer entirely from the index (covering index), no heap fetch → fastest.
Bitmap Index Scan / Bitmap Heap Scan — for medium-selectivity filters or combining multiple indexes. This immediately tells me whether my indexes are being used.

2. Join strategy.

Nested Loop — for each outer row, probe the inner side. Great when the inner side is tiny/indexed; terrible over two large inputs.
Hash Join — build a hash table on one side, probe with the other. Good for large, unsorted inputs with an equality join.
Merge Join — both inputs sorted on the join key, then merged. Good when inputs are already sorted (e.g., from indexes). Seeing a Nested Loop over millions of rows is a classic sign of a bad estimate.

3. Row estimates vs. actuals. Each node shows rows= (estimate) and, under ANALYZE, actual rows=. A large gap is the number-one diagnostic signal — it means the planner has a wrong picture of the data (stale statistics or skew), which cascades into wrong join and access choices. Fix: ANALYZE the table / raise the statistics target.

4. Cost. cost=startup..total is the planner's abstract cost estimate (in arbitrary units, roughly page reads + CPU). startup = cost before the first row; total = cost to return all rows. It's for comparing plans, not a time in ms. LIMIT cares about startup cost; a full aggregation cares about total.

5. Actual time & loops (ANALYZE only). actual time=start..end is per-loop, in ms; if loops=N, multiply to get the node's true contribution. I find the node whose actual time × loops dominates — that's what to fix.

6. Extra hints.

BUFFERS (Postgres, EXPLAIN (ANALYZE, BUFFERS)) — shared hit vs. read tells me cache vs. disk I/O; a "fast" plan that reads from disk cold behaves differently under load.
Sort Method: external merge Disk: xkB — the sort spilled to disk because work_mem was too small.
Rows Removed by Filter — how many rows were scanned then thrown away (wasted work).
Heap Fetches on an Index Only Scan — non-zero means the visibility map is stale (vacuum).
Parallel workers, Rows Removed by Join Filter, Hash Batches (multiple batches = hash spilled).

MySQL differences: classic EXPLAIN gives a tabular output — type (ALL=full scan, ref, range, const, eq_ref), key (index chosen), rows (estimated), Extra (Using index=covering, Using filesort, Using temporary). MySQL 8 adds EXPLAIN ANALYZE (real timings) and EXPLAIN FORMAT=JSON for a detailed cost breakdown. type=ALL with high rows and Using filesort/Using temporary are the smells.

How I act on it (the point of reading a plan)
Seq scan where I expected an index → add/fix an index or make the predicate sargable.
Estimate ≠ actual by orders of magnitude → ANALYZE, raise stats target; suspect skew.
Sort spilling to disk → raise work_mem or add an index that provides the order.
Nested loop over huge inputs → usually a downstream symptom of bad estimates; fix stats/indexes so a hash/merge join is chosen.
High Rows Removed by Filter → the index isn't selective enough; add a better composite index.
Summary — what a strong answer emphasizes

The crisp version: EXPLAIN shows the planner's chosen execution plan and estimated cost/rows without running the query; EXPLAIN ANALYZE runs it and adds real timings and row counts. A strong candidate lists the concrete things the plan reveals — access method (seq vs. index vs. index-only), join strategy (nested loop / hash / merge), estimated vs. actual rows (the key mismatch signal for stale stats), cost (an abstract comparison unit, not milliseconds), and actual time × loops (where time really goes) — plus BUFFERS for cache/disk and "spilled to disk" for work_mem. The senior signal is treating the estimate-vs-actual gap as the primary clue and knowing that cost is unitless while actual time is real ms.

#### Common mistakes
thinking EXPLAIN runs the query (it doesn't; ANALYZE does); reading cost as milliseconds; ignoring loops and misjudging a node's true cost; forgetting ANALYZE has side effects on writes. Follow-up probes: "estimate says 5, actual 2M — what do you do?" (analyze/stats/skew); "how do you know if it read from disk or cache?" (BUFFERS); "what does Using filesort mean in MySQL and how do you kill it?" (add an index matching the ORDER BY).

**Comparison:**

```
Aspect	EXPLAIN	EXPLAIN ANALYZE
Runs the query?	No	Yes (real execution)
Rows shown	Estimated only	Estimated and actual
Timings	Abstract cost units	Real actual time (ms) + loops
Safe on writes?	Yes	No — side effects; use BEGIN…ROLLBACK
Best for	Quick "which plan?" check	Finding real bottlenecks / est-vs-actual gaps
Plan signal	What it means	Action
Seq/Full scan on big table	Index missing or unusable	Add/fix index; sargable predicate
est rows ≠ actual rows	Stale statistics / skew	ANALYZE; raise stats target
Nested Loop over huge inputs	Bad estimate chose wrong join	Fix stats/indexes → hash/merge join
Sort Method: external merge Disk	work_mem too small	Raise work_mem or add ordering index
Buffers: read >> hit	Cold / uncached — disk I/O bound	Cache/warm; reduce pages touched
High Rows Removed by Filter	Weak index selectivity	Better composite index
Index Only Scan, Heap Fetches=0	Optimal (covering)	Keep it
```

**Code:**

```sql
-- Postgres: plan only (estimate, does NOT run the query)
EXPLAIN
SELECT o.id, o.total, c.name
FROM orders o JOIN customers c ON c.id = o.customer_id
WHERE o.status = 'PAID' AND o.created_at >= now() - interval '7 days';

-- Postgres: run it, with real timings + cache/disk I/O
EXPLAIN (ANALYZE, BUFFERS, VERBOSE)
SELECT o.id, o.total, c.name
FROM orders o JOIN customers c ON c.id = o.customer_id
WHERE o.status = 'PAID' AND o.created_at >= now() - interval '7 days';

-- For a write, run ANALYZE safely inside a rolled-back transaction:
BEGIN;
EXPLAIN (ANALYZE) DELETE FROM sessions WHERE expires_at < now();
ROLLBACK;

-- MySQL 8
EXPLAIN SELECT ...;                 -- classic tabular estimate
EXPLAIN ANALYZE SELECT ...;         -- real execution timings
EXPLAIN FORMAT=JSON SELECT ...;     -- detailed cost tree
```

**Example input / output:**

```

Reading a Postgres EXPLAIN (ANALYZE, BUFFERS) output:

CODE
Copy
Nested Loop  (cost=0.85..410.2 rows=1 width=45)
             (actual time=0.05..612.7 rows=95,000 loops=1)     <-- est 1, actual 95k = BAD stats
  Buffers: shared hit=12000 read=48000                          <-- 48k pages from DISK
  ->  Seq Scan on orders o  (cost=0.00..380.0 rows=1 width=20)
        (actual time=0.02..120.4 rows=95,000 loops=1)
        Filter: (status = 'PAID' AND created_at >= ...)
        Rows Removed by Filter: 905,000                         <-- scanned & threw away 905k
  ->  Index Scan using customers_pkey on customers c
        (actual time=0.004..0.004 rows=1 loops=95,000)          <-- probed 95,000 times
Planning Time: 0.3 ms
Execution Time: 640.1 ms

How I read it: estimate said 1 row, reality is 95k (stale stats) → planner picked a Nested Loop that now probes customers 95,000 times; the orders filter is a Seq Scan removing 905k rows. Fixes: ANALYZE orders; (so a Hash Join is chosen) + a composite index on orders(status, created_at).

MySQL EXPLAIN smell:

CODE
Copy
id  select_type  table  type  key   rows     Extra
1   SIMPLE       orders ALL   NULL  1000000  Using where; Using filesort

type=ALL (full scan) + key=NULL (no index) + Using filesort → add an index on the filter/sort columns.
```

**Flow:**

1. SQL query
2. Optimizer builds candidateplans
3. EXPLAIN: prints chosen planESTIMATED cost + rows, norun
4. EXPLAIN ANALYZE: runs itADDS actual time + actualrows + loops
5. Read the plan tree
6. Access method?Seq / Index / Index-Only /Bitmap
7. Join strategy?Nested Loop / Hash / Merge
8. est rows vs actual rows?gap = stale stats/skew
9. cost vs actual time?cost = unitless, time = ms xloops
10. BUFFERS: cache hit vs diskread
11. Decide fix: index / stats /work_mem / rewrite

