---
id: "01_Databases_and_SQL/Query_Performance/Indexing/Reading_EXPLAIN_ANALYZE"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Query_Performance/Indexing"
tools: ["MySQL"]
---

# Reading EXPLAIN ANALYZE

**L4 Advanced** · [↑ Indexing](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** `EXPLAIN` shows the plan the optimizer *chose* with *estimated* costs and rows; `EXPLAIN ANALYZE` actually runs the query and adds *actual* rows, loops, time and buffers — and the gap between estimate and actual is where almost every bad plan is explained.

### 1. The big picture
A SQL query says *what* you want; the optimizer decides *how*: which indexes, which join order, which join algorithm. You cannot tune what you cannot see, so every engine exposes the chosen plan: `EXPLAIN` in MySQL/PostgreSQL, `EXPLAIN ANALYZE` (executes), the graphical *estimated/actual execution plan* and `SET STATISTICS IO/TIME` in SQL Server, `explain()` in MongoDB, `EXPLAIN` in Trino/Spark.

Reading a plan is the core diagnostic skill of query tuning. You are always asking the same four questions: (1) how is each table accessed (scan vs seek/index), (2) in what order and how are they joined, (3) where are the expensive operators (sort, hash, temp), and (4) where do estimated rows differ from actual rows — because the optimizer picks plans based on estimates, and wrong estimates produce wrong plans.

### 2. Core concepts and vocabulary
- **Execution plan** — a tree of operators; data flows from the leaves (table/index access) up to the root (result).
- **Estimated plan** — what the optimizer predicts, without running. **Actual plan** — measured values from a real run.
- **Cost** — unitless internal estimate (PostgreSQL: startup cost..total cost; SQL Server: relative %), useful only for *comparison*.
- **Cardinality / rows** — number of rows an operator produces; *estimated* vs *actual*.
- **Seq scan / table scan / `type=ALL`** — reads the entire table. **Index scan** — reads a whole index (or range) in order. **Index seek / `ref`/`range`/`const`** — navigates the B-tree directly to matching entries.
- **Index-only scan / covering index** — query answered from the index without touching the table.
- **Join algorithms** — nested loop (good for few outer rows + indexed inner), hash join (large unsorted inputs, builds a hash table), merge join (both inputs sorted).
- **Filesort / Sort** — an explicit sort step; **Using temporary** — intermediate temp table (MySQL).
- **Loops** — how many times a node ran (inner side of nested loop); per-loop times must be multiplied.
- **Buffers (shared hit / read)** — pages found in cache vs read from disk (PostgreSQL `BUFFERS`); SQL Server's equivalent is logical vs physical reads.
- **Logical read** — page read from the buffer pool (memory); **physical read** — page fetched from disk into the pool.
- **Sargable** — predicate written so an index seek can be used.

### 3. How it works, step by step
**Life of a query (SQL Server view, similar elsewhere)**
1. The client sends the statement; the engine parses it and checks syntax and object names.
2. The optimizer enumerates candidate plans, using table/column *statistics* to estimate row counts and costs.
3. It picks the cheapest estimated plan and caches it.
4. The executor runs the operator tree, pulling rows from the leaves up.
5. `EXPLAIN ANALYZE`/actual plan records per-operator actual rows, loops, time and buffer usage while running.

**How to read a plan**
1. **Read bottom-up / inner-most first.** The most indented nodes execute first; parents consume children.
2. **For each leaf**, check access type: seek/ref/range vs scan/ALL, and which index.
3. **Compare `rows` estimated vs actual** at each node. A factor of 10× or more is the red flag.
4. **Find the expensive node** — highest actual time (× loops), most buffers read, a sort that spilled (`Sort Method: external merge`), or a hash that went to disk.
5. **Check joins**: nested loops with a large outer set and no inner index; hash join spilling.
6. **Ask what could remove work**: index, rewritten predicate, fresh statistics, smaller projection.

**MySQL `EXPLAIN` columns quick map**
| Column | Meaning | Good / bad |
|---|---|---|
| `type` | access method: `const` (1 row), `eq_ref`, `ref`, `range`, `index` (full index scan), `ALL` (full table) | const…range good; `ALL` suspect on big tables |
| `key` / `possible_keys` | index chosen / candidates | `NULL` key = no index used |
| `rows` | estimated rows examined | compare with actual need |
| `filtered` | % of examined rows expected to survive the WHERE | low % = lots of wasted reads |
| `Extra` | `Using index` (covering, good), `Using where`, `Using filesort`, `Using temporary` | filesort/temporary mean extra work, not always bad |

### 4. Worked example
```sql
-- PostgreSQL
EXPLAIN (ANALYZE, BUFFERS)
SELECT c.name, SUM(o.total)
FROM   customers c
JOIN   orders    o ON o.customer_id = c.id
WHERE  o.created_at >= DATE '2026-09-01'
GROUP  BY c.name;
```
A representative output (trimmed):
```
HashAggregate  (cost=18432..18480 rows=480 width=40) (actual time=412.1..412.4 rows=51203 loops=1)
  ->  Hash Join  (cost=240..17900 rows=106000 width=14) (actual time=8.2..330.0 rows=2150000 loops=1)
        Hash Cond: (o.customer_id = c.id)
        ->  Seq Scan on orders o (cost=0..15200 rows=106000 width=10)
              (actual time=0.1..210.5 rows=2150000 loops=1)
              Filter: (created_at >= '2026-09-01')
              Rows Removed by Filter: 7850000
              Buffers: shared hit=3200 read=61800
        ->  Hash  (cost=140..140 rows=8000) (actual time=7.9..7.9 rows=100000 loops=1)
              ->  Seq Scan on customers c ...
Planning Time: 0.4 ms   Execution Time: 415.0 ms
```
Reading it:
- **Estimated 106,000 vs actual 2,150,000 rows** on `orders`: a 20× underestimate. The planner thought the date filter was very selective; statistics were stale (or the column is correlated with something). That misestimate drives choosing a hash join + HashAggregate sized for few rows.
- `Rows Removed by Filter: 7,850,000` and `Seq Scan` tell you 80% of rows were read and thrown away — an index on `created_at` (or partitioning) would be worth testing *if* the true selectivity is low; here 21% of the table qualifies, so a seq scan is legitimately cheaper than an index.
- `Buffers: shared read=61800` vs `hit=3200`: almost everything came from disk — a cold cache, not a CPU-bound query. Run it again; if `read` drops to 0 the first run was cache warm-up.
- Action: `ANALYZE orders;` then re-run, compare estimated vs actual. If still off, consider extended statistics on correlated columns or a higher statistics target.

MySQL equivalent check: `EXPLAIN SELECT …` with `type=ALL`, `key=NULL`, `rows=9,800,000` says full scan; adding `CREATE INDEX idx_created ON orders (created_at)` should change `type` to `range`, `key` to `idx_created` and drop `rows` — *if* the filter is selective enough.

### 5. Under the hood
- **Estimates come from statistics**: row counts, distinct counts, histograms, most-common-values. When data is skewed, correlated across columns, or changed since the last `ANALYZE`/`UPDATE STATISTICS`, estimates drift. The optimizer assumes independence between predicates, so `WHERE city='X' AND zip='Y'` is underestimated when the columns are correlated.
- **`EXPLAIN ANALYZE` executes the statement.** For `INSERT/UPDATE/DELETE` it really modifies data; wrap it: `BEGIN; EXPLAIN ANALYZE …; ROLLBACK;`. Timing instrumentation itself adds overhead (per-row clock calls), so very small nodes look slower than they are (`TIMING OFF` option).
- **Loops multiply.** `actual time=0.02..0.03 rows=1 loops=50000` means ~1.5 s total for that node, not 0.03 ms. The displayed time is *per loop*.
- **Cost units** aren't milliseconds. They are built from tunables like `seq_page_cost` (1.0) and `random_page_cost` (4.0 default). On SSDs where random reads are nearly as cheap as sequential, leaving `random_page_cost` at 4.0 makes the planner *over-penalize index scans* and choose sequential scans or bitmap scans when an index would be faster — lower it (≈1.1) for SSD-backed databases. Setting it too *high* is the symptom: indexes ignored.
- **Filesort doesn't necessarily mean disk.** In MySQL "Using filesort" means a sort step that isn't satisfied by index order; it happens in memory unless the sort buffer is exceeded. An index whose column order matches `WHERE` equality columns then `ORDER BY` columns can eliminate it. "Using temporary" means an intermediate table for `GROUP BY`/`DISTINCT`/`UNION` that can't be done by streaming; often unavoidable and not automatically bad.
- **Plan reuse.** SQL Server caches plans by statement text/parameters; a plan compiled for one parameter value ("sniffing") may be wrong for another, which shows up as estimated vs actual mismatch only for some inputs. Slow-log + `auto_explain` (PostgreSQL) or Query Store (SQL Server) capture plans for those intermittent cases.
- **Logical vs physical reads.** Logical reads are the stable, repeatable measure of work; physical reads depend on cache state. Tune against logical reads and check physical to explain run-to-run variance.

### 6. Performance and design implications
- Hunt for: large *estimate vs actual* gaps; scans on big tables with highly selective predicates; key lookups multiplied over many rows; sorts/hashes spilling; nested loops with huge outer input.
- Use buffer counts to separate **I/O-bound** (high `read`/physical reads) from **CPU-bound** (all hits, long time) from **row-count-bound** (millions of rows moved).
- Fix the cause in order of cost: (1) statistics, (2) predicate rewrite for sargability, (3) index, (4) query shape/joins, (5) hints/forcing as a last resort.
- Check what the application *actually sends*: ORMs generate surprising SQL; enable ORM SQL logging (`hibernate.show_sql`, Sequelize `logging`) or the server's general/slow query log, then explain *that* statement with real parameter values.
- Run `EXPLAIN ANALYZE` on production with care: use a replica, a transaction that rolls back for DML, `statement_timeout`, and off-peak windows.
- Capture plans for intermittent slowness with `auto_explain` (log plans over a threshold), slow query log, or Query Store.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `EXPLAIN <query>` | Shows chosen plan with estimates, no execution | First look, safe on any statement |
| `EXPLAIN (ANALYZE, BUFFERS)` (PG) | Executes, adds actual rows/time/buffers | Find real bottleneck; wrap DML in a rolled-back transaction |
| `EXPLAIN ANALYZE` (MySQL 8.0.18+) | Executes, adds actual times/rows per iterator | MySQL timing and misestimates |
| `SET STATISTICS IO, TIME ON` + actual plan (SQL Server) | Logical/physical reads, CPU, elapsed | Compare plan variants objectively |
| `ANALYZE` / `UPDATE STATISTICS` | Refresh optimizer statistics | After large data changes, estimate gaps |
| `CREATE STATISTICS … (a,b)` (PG extended stats) | Capture column correlation | Correlated predicates misestimated |
| `auto_explain.log_min_duration` | Logs plans of slow queries | Occasional bad plans |
| `random_page_cost` | Planner's random I/O penalty | Lower for SSD/NVMe |
| MongoDB `explain("executionStats")` | `IXSCAN` vs `COLLSCAN`, docs examined vs returned | Document store index checks |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| **Plain `EXPLAIN`** — quick check, writes or production queries you can't run | Estimates may be badly off; you can't see actual behaviour |
| **`EXPLAIN ANALYZE`** — need actual rows, timing, buffers | Statement modifies data or is very long-running; timing overhead on tiny operators |
| **Add an index** — selective filter, join or sort column, query is frequent | Predicate is non-selective (returns a large fraction of rows) or table is write-heavy |
| **Refresh/extend statistics** — large estimate gaps, correlated columns | Estimates already right: the plan is simply expensive |
| **Rewrite the query** — non-sargable predicates, implicit conversions, `SELECT *` | Change would alter semantics; verify results |
| **Hints / plan forcing** — emergency stabilisation | They freeze a plan that may become wrong as data grows |

### 9. Common mistakes
- **Trusting estimated cost as time** → chasing "high cost" nodes that are cheap → compare actual time/loops/buffers.
- **Reading `EXPLAIN` without `ANALYZE` and calling it proof** → plan may differ from reality → confirm with actuals.
- **Running `EXPLAIN ANALYZE` on `DELETE`/`UPDATE` in production** → data modified → use a transaction with `ROLLBACK`.
- **Ignoring `loops`** → underestimating a node's total time → multiply time by loops.
- **Believing "Using filesort"/"Using temporary" are always bad** → pointless rewrites → check size, whether an index can supply order, and measured time.
- **Seeing a seq scan and demanding an index** → index slower when most rows match → check selectivity.
- **Comparing runs with different cache states** → misleading speedups → compare logical reads or warm both.
- **Explaining a different statement than the app runs** → wrong parameter types/values → capture the real SQL from the ORM/log.
- **Stale statistics** → bad join orders and under-sized memory grants → refresh and schedule stats maintenance.

### 10. 🎯 Interview kit
**30-second answer:** "`EXPLAIN` prints the plan the optimizer chose with estimated rows and cost; `EXPLAIN ANALYZE` runs it and adds actual rows, time, loops and buffers. I read it from the innermost node outward: access method (seek vs scan), join algorithm and order, expensive sorts or hashes, and most importantly estimated vs actual rows — a big gap means bad statistics or correlation and explains the bad plan."

**Structure a longer answer:** clarify (engine, query, size, is it slow or sometimes slow) → read the plan (access, joins, sorts) → check estimates vs actuals and buffers → hypothesis (stats, index, sargability, parameter sniffing, SSD cost settings) → fix and re-measure → close with monitoring (slow log/`auto_explain`/Query Store).

**Likely follow-ups:**
- *Actual rows 100× estimate?* → stale/insufficient stats, skewed or correlated columns, parameter sniffing, table variable/no stats; fix with `ANALYZE`, extended stats, higher statistics target, rewrite.
- *What does `type = ALL` mean?* → full table scan (MySQL); `const/eq_ref/ref/range` are index-based.
- *Shared hit vs shared read?* → cache hits vs disk reads; high reads = I/O-bound or cold cache.
- *Does an index always remove filesort?* → Only if index order matches ORDER BY after equality predicates; mixed directions or range columns may break it.
- *Safely analyzing a write?* → transaction + rollback, replica, timeouts.
- *Why would `random_page_cost` matter?* → index plans penalized on SSD; lowering lets planner prefer index.

**Signals interviewers look for:** methodical reading order, focus on estimate-vs-actual, distinguishing I/O from CPU, safety with `ANALYZE` on prod, and fixing causes (stats, predicates) before hinting.

### 11. Check yourself
- [ ] Take a slow query and identify its leaf access method, join algorithm and biggest node by actual time × loops.
- [ ] Explain why a Seq Scan can be the correct choice even with an index available.
- [ ] Use `BUFFERS` output to decide whether a query is disk- or CPU-bound.
- [ ] List four reasons for a 100× row misestimate and a fix for each.
- [ ] Show how to run `EXPLAIN ANALYZE` on an `UPDATE` without keeping its changes.
- [ ] Describe what "Using filesort" and "Using temporary" mean and when each is acceptable.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Basic] [How would you explain the difference between a logical and a physical read when profiling a slow pipeline query to a new teammate?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20332_How_would_you_explain_the_difference_between_a_logical_and_a.md>)
- [Intermediate] [What does the 'type' column in a MySQL EXPLAIN (const, ref, range, ALL) tell a data engineer about whether their extract query uses an index?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20313_What_does_the_type_column_in_a_MySQL_EXPLAIN_const_ref_range.md>)
- [Intermediate] [What does EXPLAIN (or EXPLAIN ANALYZE) tell you about a query?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19469_What_does_EXPLAIN_or_EXPLAIN_ANALYZE_tell_you_about_a_query.md>)
- [Intermediate] [When you see "Using filesort" in EXPLAIN, what does it mean and is it always bad?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19571_When_you_see_Using_filesort_in_EXPLAIN_what_does_it_mean_and.md>)
- [Intermediate] [How would you read a simple EXPLAIN output to tell if an index is being used?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20006_How_would_you_read_a_simple_EXPLAIN_output_to_tell_if_an_ind.md>)
- [Intermediate] [How do you find the actual SQL that your ORM (like Hibernate or Sequelize) sends, so you can check its indexes?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20036_How_do_you_find_the_actual_SQL_that_your_ORM_like_Hibernate_.md>)
- [Deep] [What does "Using temporary" in a MySQL plan indicate and which operations trigger it?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19572_What_does_Using_temporary_in_a_MySQL_plan_indicate_and_which.md>)
- [Deep] [How would you use EXPLAIN (ANALYZE, BUFFERS) to decide whether a slow query is limited by disk reads or by CPU-bound row processing?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20094_How_would_you_use_EXPLAIN_ANALYZE_BUFFERS_to_decide_whether_.md>)

## Related concepts

- [Reading Execution Plans](<../Execution_Plans/Reading_Execution_Plans.md>) *(Execution Plans)*
- [Reading EXPLAIN](<../../../TOOLS/MySQL/MySQL_Platform/Reading_EXPLAIN.md>) *(MySQL Platform)*
- [Reading EXPLAIN](<../../SQL_Language/SQL_Fundamentals/Reading_EXPLAIN.md>) *(SQL Fundamentals)*

## References

- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Introducing SQL Server › 1.2.1 Overview of the Database Engine

