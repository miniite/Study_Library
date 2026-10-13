---
type: "practice"
difficulty: "Deep"
---

# What is a full table scan and when is it acceptable versus a red flag?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Reading EXPLAIN](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Reading_EXPLAIN.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Define a full table scan, then show judgment about when it's fine and when it signals a performance problem. The word "deep understanding" plus fintech/e-commerce tiers means the interviewer wants more than "scans mean no index" — they want to know that you understand the query planner's cost model, selectivity, table size, access patterns, and the difference between a scan being the symptom versus the right plan.

What the interviewer is probing: Do you read EXPLAIN/EXPLAIN ANALYZE? Do you understand that the optimizer sometimes chooses a scan on purpose? Can you diagnose a bad scan and fix it without cargo-culting indexes onto everything?

Clarifying assumptions I'll state: Row-store OLTP databases (PostgreSQL/MySQL-InnoDB) unless told otherwise; a columnar/analytics warehouse (Redshift, BigQuery, Snowflake) changes the answer because scans are the normal mode there.

Plan: (1) define the term precisely, (2) explain why the planner might pick it, (3) give the concrete rules for "acceptable" vs "red flag", (4) show how to diagnose and fix, (5) note the analytics exception.

What a full table scan actually is

A full table scan (a.k.a. sequential scan, Seq Scan in PostgreSQL, type: ALL in MySQL EXPLAIN) is when the database reads every row/page of the table and applies the filter to each one, instead of using an index to jump directly to the matching rows. Cost is roughly O(N) in the number of rows, and — more importantly for a disk-bound OLTP system — it does O(pages) sequential I/O.

Contrast with an index scan / index seek: the engine walks a B-tree (log N) to find matching keys and then, if needed, fetches the corresponding rows (a "bookmark lookup" / heap fetch). An index range scan touches only the qualifying slice.

Key nuance: a full table scan is not automatically bad. The optimizer is cost-based. It estimates the cost of each plan and picks the cheapest. Sometimes a sequential scan genuinely is cheapest.

Why the planner sometimes chooses a scan (and is right to)

Low selectivity of the predicate. If a query returns a large fraction of the table (rule of thumb ~5–20%+ depending on the engine and row width), an index adds overhead: for each matching key you do a random heap fetch. Random I/O is far slower than sequential I/O. Past a threshold, "just read the whole thing sequentially" wins. This is why an index on a status column where 60% of rows are ACTIVE won't be used for WHERE status='ACTIVE'.

Small tables. A 500-row lookup/config table fits in a page or two. The planner will scan it because a B-tree descent plus buffer bookkeeping costs more than reading two pages. Adding an index here is pure write-time overhead with zero read benefit.

No usable index for the predicate. WHERE UPPER(email) = 'X@Y.COM', WHERE created_at::date = '2026-01-01', or LIKE '%foo%' can't use a plain B-tree on the raw column, so the engine scans.

Aggregations/reports that legitimately touch everything. SELECT count(*), avg(amount) FROM orders WHERE created_at >= now() - interval '1 day' on a table where "yesterday" is most of the data — you're going to read most pages anyway.

Stale statistics. The planner thinks a predicate is unselective because ANALYZE/ANALYZE TABLE hasn't run. Here the scan is a red flag, but the fix is statistics, not an index.

Acceptable vs red flag — the decision rule

Acceptable when: the table is small; OR the query genuinely needs a large fraction of rows (analytics/report/full export); OR it's a rare/batch/back-office query, not a hot path; OR you're in a columnar warehouse where scans are the intended access pattern.

Red flag when: a scan sits on the hot OLTP path (per-request, high QPS) against a large and growing table, returning a small selective result set. That's the classic "works in dev with 1k rows, melts in prod with 50M rows" bug. Fintech/payments and e-commerce feeds are exactly where this hurts: a SELECT ... FROM transactions WHERE user_id = ? doing a Seq Scan at 5k QPS is a latency cliff and a lock/contention amplifier.

Two extra red-flag signatures worth naming:

Scan under a nested loop join. If the inner side of a join is repeatedly full-scanned per outer row, cost multiplies — often the real cause of a "slow join."
Scan + Sort + Limit where an index could have provided pre-sorted rows and stopped early (an "ORDER BY … LIMIT" that reads and sorts the whole table to return 20 rows).
How to diagnose

Run EXPLAIN (ANALYZE, BUFFERS) (Postgres) / EXPLAIN ANALYZE (MySQL 8). Look for:

Seq Scan / type: ALL on a big table on a hot path.
Estimated vs actual rows diverging wildly → stale stats; run ANALYZE.
Rows Removed by Filter huge → you read millions to keep hundreds → indexable.
High shared read/hit buffer counts on the scan node.
How to fix (in order of preference)
Make the predicate sargable + add the right index. WHERE user_id = ? → B-tree on user_id. For UPPER(email) use a functional/expression index CREATE INDEX ON users (lower(email)) and query with lower(email)=. For created_at::date filter on a range instead: created_at >= d AND created_at < d+1.
Composite / covering indexes so the query is answered from the index alone (index-only scan), avoiding heap fetches: (user_id, created_at DESC) INCLUDE (amount).
Refresh statistics (ANALYZE) if the estimates are wrong — cheaper and often the actual root cause.
Rewrite the query — remove functions on columns, replace OR with UNION/IN, replace LIKE '%x%' with a trigram/full-text index if search is real.
Don't over-index. Every index slows INSERT/UPDATE/DELETE and consumes cache. On a write-heavy fintech ledger, an unused index is a tax on every write and on WAL/replication.
The analytics exception

In a columnar warehouse (Redshift, BigQuery, Snowflake, ClickHouse), full scans are the design. They rely on column pruning, compression, zone maps / block-range indexes and MPP parallelism, not B-trees. There, the lever isn't "add an index," it's partitioning, clustering/sort keys, and reading fewer columns. Judging a warehouse scan by OLTP rules is a mistake.

Summary — what a strong answer emphasizes

A full table scan reads every page and is O(N). It is not inherently bad — the cost-based optimizer legitimately picks it for small tables, unselective predicates, and full-table aggregations. It becomes a red flag when it lands on a large, growing table on a hot, high-QPS OLTP path returning few rows. Diagnose with EXPLAIN ANALYZE (watch "Rows Removed by Filter" and estimate-vs-actual), then fix via a selective/sargable index, covering indexes, or refreshed statistics — while respecting the write cost of indexes and remembering that in columnar warehouses scanning is the intended, healthy mode.

#### Common mistakes
blindly indexing every column; assuming "scan = always bad"; forgetting functions-on-columns kill index usage; ignoring stale statistics; applying OLTP intuition to a warehouse.

**Comparison:**

```
Situation	Full table scan is...	Why / what to do
Small lookup/config table (~hundreds of rows)	Acceptable	Fits in 1–2 pages; index adds only write cost
Predicate returns >~20% of a large table	Acceptable	Sequential I/O beats many random heap fetches
Full-table aggregation / report / export	Acceptable	Query legitimately needs most rows
Columnar warehouse (Redshift/BigQuery/Snowflake)	Acceptable / expected	Scans + column pruning are the design; tune sort/cluster/partition keys
Hot OLTP path, large growing table, selective result	Red flag	Add selective/covering index; classic prod latency cliff
Non-sargable predicate (UPPER(col), col::date, LIKE '%x%')	Red flag	Rewrite predicate; use expression / trigram / FTS index
Estimate ≠ actual rows (stale stats)	Red flag (fix = stats)	Run ANALYZE; don't add an index blindly
Scan on inner side of nested-loop join	Red flag	Index the join key; cost multiplies per outer row
```

**Code:**

```sql
-- Red flag: Seq Scan on a large hot-path table
EXPLAIN (ANALYZE, BUFFERS)
SELECT id, amount, created_at
FROM transactions
WHERE user_id = 91234;
-- Seq Scan on transactions  (actual rows=42 ... Rows Removed by Filter=49,999,958)  <-- bad

-- Fix 1: selective B-tree index makes it an Index Scan
CREATE INDEX idx_txn_user ON transactions (user_id);

-- Fix 2: covering / composite index for "latest N for a user" (index-only scan, pre-sorted)
CREATE INDEX idx_txn_user_time
  ON transactions (user_id, created_at DESC) INCLUDE (amount);

-- Non-sargable predicate forces a scan; make it sargable
-- BAD:
SELECT * FROM users WHERE UPPER(email) = 'A@B.COM';
-- GOOD: expression index + matching predicate
CREATE INDEX idx_users_lower_email ON users (lower(email));
SELECT * FROM users WHERE lower(email) = 'a@b.com';

-- Date-cast kills the index; use a half-open range instead
-- BAD:  WHERE created_at::date = DATE '2026-01-01'
-- GOOD:
SELECT * FROM orders
WHERE created_at >= '2026-01-01' AND created_at < '2026-01-02';

-- Acceptable scan: the fix is statistics, not an index
ANALYZE transactions;  -- refresh planner stats when estimates are wrong
```

**Example input / output:**

```text
EXPLAIN ANALYZE, BEFORE (red flag):
  Seq Scan on transactions
    Filter: (user_id = 91234)
    Rows Removed by Filter: 49,999,958
    actual time = 8123 ms   (reads ~all pages, high buffers read)

AFTER CREATE INDEX idx_txn_user (fixed):
  Index Scan using idx_txn_user on transactions
    Index Cond: (user_id = 91234)
    actual rows = 42
    actual time = 0.9 ms

ACCEPTABLE scan (unselective predicate, planner is right to scan):
  Query:  SELECT count(*) FROM orders WHERE status = 'ACTIVE'   -- 60% of rows
  Plan:   Seq Scan on orders  (index would be slower: random heap fetches for most rows)
```

**Flow:**

1. No small/config
2. Yes
3. No batch/report
4. Yes
5. No returns big fraction
6. Yes
7. No
8. Yes
9. No
10. Yes
11. EXPLAIN ANALYZE shows a fulltable scan
12. Table large & growing?
13. Acceptable: leave it
14. On a hot high-QPS path?
15. Likely acceptable
16. Result set selective few rows?
17. Acceptable: sequential beatsrandom fetches
18. Estimate matches actual rows?
19. Stale stats -> ANALYZE
20. Predicate sargable?
21. Rewrite: remove func oncolumn / range / trigram-FTS
22. Add selective or coveringindex

