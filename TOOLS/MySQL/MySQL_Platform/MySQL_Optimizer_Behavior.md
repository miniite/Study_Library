---
id: "TOOLS/MySQL/MySQL_Platform/MySQL_Optimizer_Behavior"
level: 4
type: "concept"
topic: "TOOLS/MySQL/MySQL_Platform"
tools: ["MySQL"]
---

# MySQL Optimizer Behavior

**L4 Advanced** · [↑ MySQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** The MySQL optimizer picks an access path per table from the indexes and statistics it has; you steer it by designing indexes whose column order matches *equality → sort → range*, writing predicates it can use, and reading `EXPLAIN` to confirm what it actually chose.

### 1. The big picture
You write *what* you want; the optimizer decides *how*: which index (if any) to use for each table, in which order to join, whether to sort or use index order, whether to materialize a subquery or turn it into a semi-join. It's cost-based: it estimates rows from index statistics (cardinality) and chooses the cheapest-looking plan. Most "slow query" tickets are one of four things: no usable index, the wrong column order in a composite index, a predicate that hides the column from the index, or a query shape that multiplies rows (N+1, join fan-out).

An index works like the index at the back of a book: a sorted list of terms pointing to pages. It speeds reads of the terms you look up and slows writes, because every insert/update/delete must maintain it. This lesson covers how to create and judge indexes, how composite order works, how subqueries/EXISTS/CTEs are treated, and the diagnosis habits (`EXPLAIN`, invisible indexes, redundant-index detection) that keep a schema healthy.

*(Related book context: analytic-SQL material such as AWS's Athena SQL chapter shows the same query shapes — subqueries, CTEs — on engines that plan very differently; in MySQL the plan details below are what matter.)*

### 2. Core concepts and vocabulary
- **Primary (clustered) index** — the PK B+tree that holds the rows. **Secondary index** — separate B+tree whose leaves hold the indexed columns + the PK.
- **Composite (multi-column) index** — `(a, b, c)`; usable for a leftmost prefix: `a`, `a,b`, `a,b,c`.
- **Covering index** — contains all columns the query needs; `EXPLAIN` shows `Using index`; no table lookup.
- **Cardinality** — number of distinct values in an index; low-cardinality columns (boolean, status) filter poorly alone.
- **Selectivity** — fraction of rows a predicate keeps; lower = better for indexes.
- **Sargable predicate** — can use an index for a seek/range (`col = ?`, `col > ?`, `col LIKE 'x%'`); not `FUNC(col) = ?` or `col + 1 = ?`.
- **filesort** — an explicit sort step (`Using filesort` in `EXPLAIN`); not necessarily on disk.
- **Semi-join / `EXISTS`** — "does a matching row exist" without returning duplicates; the optimizer can stop at the first match.
- **Correlated subquery** — references the outer query; logically re-evaluated per outer row. **Non-correlated** — independent; evaluated once.
- **CTE (`WITH`)** — named subquery; in 8.0 either merged into the outer query or materialized.
- **N+1 problem** — 1 query for the list + N queries for each row's details.
- **Invisible index** — an index the optimizer ignores but that is still maintained; lets you test a drop safely.

### 3. How it works, step by step
**From SQL to plan**
1. Parse and resolve names; rewrite (merge derived tables/views, convert `IN (subquery)` to a semi-join, simplify constants).
2. For each table list candidate indexes whose columns match predicates, joins, `ORDER BY`, `GROUP BY`.
3. Estimate rows using index statistics (`ANALYZE TABLE` refreshes them; InnoDB samples pages).
4. Search join orders (greedy, depth-bounded) and cost: random lookups, scans, temp tables, sorts.
5. Emit the plan; the executor runs it nested-loop style (plus hash join in 8.0.18+ for non-indexed equi-joins).

**Designing a composite index — the E-S-R rule**
1. **E**quality columns first (`status = ?`, `user_id = ?`).
2. Then the **S**ort columns (`ORDER BY created_at DESC`) so rows come out in order.
3. Then the **R**ange column (`created_at > ?`, `amount BETWEEN …`) — columns after the first range column can't be used for seeking (they can still filter via index condition pushdown).
4. Append extra selected columns to make it covering if the benefit justifies the size.

### 4. Worked example
```sql
-- Create / inspect / drop
CREATE INDEX ix_orders_user_created ON orders (user_id, created_at);
SHOW INDEX FROM orders;
ALTER TABLE orders DROP INDEX ix_orders_created;

-- Query: a user's recent orders, newest first
SELECT order_id, total
FROM   orders
WHERE  user_id = 42 AND status = 'PAID'
ORDER  BY created_at DESC
LIMIT  20;
```
With only `INDEX(created_at)` the optimizer either scans `created_at` backwards, testing `user_id` on each row, or uses another index and sorts: `Using where; Using filesort`. With `INDEX(user_id, status, created_at)` (equality, equality, sort) it seeks to `(42,'PAID')` and reads 20 entries in order — no sort, no extra rows:
```
type: ref   key: ix_user_status_created   rows: ~20   Extra: Backward index scan
```
**The (status, created_at) puzzle:** `WHERE status = 'PAID' AND created_at > ? ORDER BY amount` still sorts: the index gives rows ordered by `created_at`, not `amount`. The index can only provide *one* ordering; choose it from the dominant query.

**EXISTS vs JOIN + GROUP BY**
```sql
-- Parents having at least one paid child — stops at the first match, no duplicates
SELECT c.customer_id, c.name
FROM   customer c
WHERE  EXISTS (SELECT 1 FROM orders o
               WHERE o.customer_id = c.customer_id AND o.status = 'PAID');

-- JOIN version multiplies rows, then must collapse them
SELECT c.customer_id, c.name
FROM   customer c JOIN orders o ON o.customer_id = c.customer_id AND o.status = 'PAID'
GROUP  BY c.customer_id, c.name;
```
With `INDEX(customer_id, status)` on `orders`, `EXISTS` probes once per customer (`FirstMatch` strategy). The join creates a row per matching order and aggregates — more work and the source of the "counts don't match rows" bug when a parent has several children: `COUNT(*)` after the join counts child rows, not parents; use `COUNT(DISTINCT c.customer_id)` or an `EXISTS`/pre-aggregated derived table.

**N+1 and its fix**
```sql
-- ORM lazy-load: 1 + N queries
SELECT * FROM posts LIMIT 50;
SELECT * FROM comments WHERE post_id = 1;  -- ... repeated 50 times

-- Fix: one extra query with IN, or a JOIN, or aggregate
SELECT post_id, COUNT(*) FROM comments WHERE post_id IN (1,2,…,50) GROUP BY post_id;
```
Find it in the slow/general log or `performance_schema.events_statements_summary_by_digest`: a cheap statement with enormous `COUNT_STAR`.

**CTE**
```sql
WITH recent AS (
  SELECT user_id, SUM(total) AS spend
  FROM orders WHERE created_at >= NOW() - INTERVAL 30 DAY GROUP BY user_id
)
SELECT u.name, r.spend FROM users u JOIN recent r USING (user_id) WHERE r.spend > 1000;
```
A CTE is readability, not a performance fence in 8.0 — a simple CTE is merged like a derived table; a CTE referenced multiple times, or with aggregation, is materialized once.

**Pagination with total in one round trip (8.0)**
```sql
SELECT o.*, COUNT(*) OVER () AS total_rows
FROM orders o WHERE user_id = 42
ORDER BY created_at DESC LIMIT 20 OFFSET 40;
```
(`SQL_CALC_FOUND_ROWS` is deprecated.) For large tables, prefer keyset pagination and an approximate/cached total.

### 5. Under the hood
- **Leftmost prefix & skip.** B+tree entries are ordered by the first column, then second, etc. A predicate on `b` alone can't seek into `(a, b)`; 8.0.13+ may use *skip scan* when `a` has few distinct values.
- **Index condition pushdown (ICP).** For columns in the index but after the seek columns, InnoDB tests the condition in the index before fetching the row (`Using index condition`).
- **Why low cardinality is weak.** Index on a boolean returns ~50% of rows by random lookup; the optimizer will often prefer a scan since sequential reads beat millions of random lookups. Combine it as a trailing/leading column in a composite, or use histograms (8.0: `ANALYZE TABLE … UPDATE HISTOGRAM`) to help estimates for non-indexed columns.
- **Statistics.** `innodb_stats_persistent` (default ON) stores sampled stats; `innodb_stats_persistent_sample_pages` controls accuracy. Skewed data or stale stats produce bad row estimates — compare `EXPLAIN` estimate with `EXPLAIN ANALYZE` actuals.
- **Covering and the clustered index.** Secondary index leaves contain the PK, so `SELECT pk, indexed_col` is covered for free.
- **Subqueries.** `IN (SELECT …)` is typically rewritten to a semi-join (FirstMatch, LooseScan, DuplicateWeedout, Materialization). `NOT IN` is hazardous with `NULL`s in the subquery (it returns nothing); prefer `NOT EXISTS`.
- **Hash join (8.0.18+)** replaces block-nested-loop for equi-joins without an index — better, but an index on the join column still wins for selective lookups.
- **Invisible indexes (8.0).** `ALTER TABLE t ALTER INDEX ix INVISIBLE;` keeps the index maintained but hides it from the optimizer; if nothing regresses over a representative period, drop it. Making it `VISIBLE` again is instant — a safe, reversible test.

### 6. Performance and design implications
- Every index costs write amplification, buffer-pool space and disk; a table with 10 indexes does ~10 B-tree updates per insert.
- **Duplicate/redundant indexes** (`(a)` when `(a,b)` exists) waste space — `sys.schema_redundant_indexes`; **unused indexes** — `sys.schema_unused_indexes` (uptime-dependent; collect over a full business cycle).
- **Denormalize** a hot lookup into the main table only when the join is measurably the bottleneck; then you own the sync problem (triggers, application writes, or periodic reconciliation) and must tolerate drift — often a cache/summary table is a better first step.
- **Measure success** on a ticket like "make the orders page faster" by capturing p95 latency, rows examined vs rows sent (`performance_schema` digest `SUM_ROWS_EXAMINED` / `SUM_ROWS_SENT`), and query counts per request.
- **Catch regressions in CI:** run `EXPLAIN FORMAT=JSON` for critical queries against a production-sized fixture and assert on `access_type`, `key`, absence of `filesort`/`using_temporary_table`, and "covering" flags.
- Beware `SELECT *`: a query covered by an index silently regresses to table lookups when a column is added.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE [UNIQUE] INDEX ix ON t (a,b)`; `ALTER TABLE t ADD INDEX …` | Build an index | New access path |
| `SHOW INDEX FROM t`; `information_schema.STATISTICS` | List indexes, cardinality | Audit |
| `DROP INDEX ix ON t` | Remove index | After invisible-test |
| `ALTER INDEX ix INVISIBLE / VISIBLE` | Hide from optimizer | Safe drop trial |
| `EXPLAIN`, `EXPLAIN FORMAT=JSON/TREE`, `EXPLAIN ANALYZE` | Plan and actuals | Every tuning task |
| `ANALYZE TABLE t [UPDATE HISTOGRAM ON col]` | Refresh stats / histogram | After bulk changes, skewed columns |
| `USE INDEX / FORCE INDEX / IGNORE INDEX`, optimizer hints `/*+ … */` | Override choice | Last resort, document it |
| `sys.schema_redundant_indexes`, `sys.schema_unused_indexes` | Find waste | Index hygiene |
| `optimizer_switch` | Toggle strategies | Troubleshooting regressions |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Composite index matching E-S-R of the hot query | Many different sort orders on the same filter set (can't serve all) |
| Covering index for a hot read path | Wide indexes on write-heavy tables |
| `EXISTS` for "has any child" | You actually need child columns/aggregates |
| Join + aggregate for per-parent summaries | Fan-out causes wrong counts — pre-aggregate in a derived table |
| CTE for clarity | Assuming it forces materialization (it doesn't) |
| Denormalized copy | You can't guarantee synchronization |
| Index hints | The data distribution changes (the hint becomes a trap) |

### 9. Common mistakes
- **Function on indexed column** (`WHERE DATE(created_at) = …`, `LOWER(email) = …`) → index unusable → range predicate / functional index.
- **Wrong composite order** (range column first) → filesort or wide scan → equality, sort, range.
- **Indexing each column separately** and hoping for the best → index merge at best → one purpose-built composite.
- **Indexing booleans alone** → ignored → combine or drop.
- **Implicit conversions** (comparing a `VARCHAR` column to a number, or mismatched collations) → full scan → match types.
- **`NOT IN` with NULLs** → empty results → `NOT EXISTS`.
- **`COUNT(*)` after a one-to-many join** → inflated counts → distinct/pre-aggregate.
- **Fixing N+1 by caching the symptom** → still N queries on a miss → eager load or batch `IN`.
- **Adding indexes without removing old ones** → bloat → review quarterly.

### 10. 🎯 Interview kit
**30-second answer:** An index is a sorted structure that lets MySQL seek instead of scan; it speeds reads and costs writes. For composites put equality columns first, then the sort column, then the range column. Check with `EXPLAIN` for `type`, `key`, `rows`, `Using filesort`, `Using index`.

**Structure a longer answer:** show the query → the plan before → the index and why that order → plan after → write/space cost → how you'd verify (invisible index, `EXPLAIN ANALYZE`, digest stats) → CI guard.

**Likely follow-ups:**
- *Why does `(status, created_at)` still filesort for `ORDER BY amount`?* → index order is by `created_at`; one ordering per index.
- *Correlated vs non-correlated subquery?* → references outer row or not; optimizer often rewrites to semi-join.
- *EXISTS or JOIN?* → `EXISTS` for existence (stops at first match, no duplicates).
- *What is cardinality and why does a boolean index fail?* → distinct values; half the rows returned, random I/O beats a scan.
- *How do you prove an index is dead?* → invisible index + digest stats over a full cycle, then drop.
- *CTE vs subquery performance?* → equivalent in 8.0; readability differs.

**Signals interviewers look for:** you read `EXPLAIN`, reason about row counts, mention write cost, and have a safe rollout/rollback path.

### 11. Check yourself
- [ ] Design an index for `WHERE user_id=? AND status=? AND created_at>? ORDER BY created_at DESC LIMIT 20`.
- [ ] Explain `Using index` vs `Using index condition` vs `Using filesort`.
- [ ] Rewrite a `NOT IN (subquery)` safely and explain why.
- [ ] Detect and fix an N+1 from a slow-log digest.
- [ ] Use an invisible index to retire an unused index.
- [ ] Explain why `COUNT(*)` changed after adding a join.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../_index.md>)

## Practice questions

- [Basic] [What is a primary index versus a secondary index in simple terms?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33427_What_is_a_primary_index_versus_a_secondary_index_in_simple_t.md>)
- [Intermediate] [How would you rewrite a correlated subquery in the SELECT list (a per-row lookup) into a single JOIN or window function, and when is the rewrite not equivalent?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33459_How_would_you_rewrite_a_correlated_subquery_in_the_SELECT_li.md>)
- [Intermediate] [What is a subquery and how does a correlated subquery differ from a non-correlated one?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33147_What_is_a_subquery_and_how_does_a_correlated_subquery_differ.md>)
- [Intermediate] [A query joins a table on a VARCHAR column against another's INT and it's slow with implicit casting — how do you spot the type-coercion index kill and fix it?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33311_A_query_joins_a_table_on_a_VARCHAR_column_against_another_s_.md>)
- [Deep] [A query joins five tables and the optimizer's chosen join order produces a giant intermediate — how do you use STRAIGHT_JOIN or a rewrite to guide it, and why is that a last resort?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33505_A_query_joins_five_tables_and_the_optimizer_s_chosen_join_or.md>)
- [Deep] [How does an index on a nullable column behave, and are NULLs stored in a B-tree index?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33054_How_does_an_index_on_a_nullable_column_behave_and_are_NULLs_.md>)
- [Deep] [How does MySQL decide whether to use an index or do a full table scan?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33583_How_does_MySQL_decide_whether_to_use_an_index_or_do_a_full_t.md>)
- [Deep] [A recursive CTE traversing an org hierarchy blows up to millions of rows or loops forever — how do you debug and bound it, and when is a closure/path table better?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33200_A_recursive_CTE_traversing_an_org_hierarchy_blows_up_to_mill.md>)

## Related concepts

- [Index Tuning in Practice](<../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/_index.md>) *(Indexing)*
- [Query Rewrites and Plan Stability](<../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Query_Rewrites_and_Plan_Stability.md>) *(Execution Plans)*
- [UDF Performance](<../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/UDF_Performance.md>) *(SQL Functions)*
- [Leftmost Prefix Rule](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Leftmost_Prefix_Rule.md>) *(SQL Fundamentals)*
- [Index Access Paths](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Index_Access_Paths.md>) *(DBMS Concepts)*

## References

- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Presentation Formats › SQL Analytics Using Amazon Athena
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Conclusion › Q

