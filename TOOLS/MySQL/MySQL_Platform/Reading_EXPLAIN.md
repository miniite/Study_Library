---
id: "TOOLS/MySQL/MySQL_Platform/Reading_EXPLAIN"
level: 4
type: "concept"
topic: "TOOLS/MySQL/MySQL_Platform"
tools: ["MySQL"]
---

# Reading EXPLAIN

**L4 Advanced** · [↑ MySQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** `EXPLAIN` shows the plan MySQL *intends* to run (access type, index, estimated rows, extra work), and `EXPLAIN ANALYZE` runs the query and shows what *actually* happened; you read them together to find why a query is slow.

### 1. The big picture
Every slow MySQL query is slow for a reason that is visible in its execution plan: it reads too many rows, it reads them through a poor access path, or it does extra work afterwards (sorting, building temporary tables). `EXPLAIN` is the optimizer's own report of the plan it picked. You never guess which index is used or in what order tables are joined; you ask.

MySQL gives you three levels of detail. Classic tabular `EXPLAIN` is the quick overview. `EXPLAIN FORMAT=JSON` adds cost estimates and details that the table hides (for example which part of a composite index is used). `EXPLAIN ANALYZE` (MySQL 8.0.18+) executes the statement and prints a tree annotated with measured time and row counts, so you can compare the optimizer's estimates with reality. This lesson teaches how to read all three and how to turn the reading into a fix.

### 2. Core concepts and vocabulary
- **Access type (`type` column)** — how MySQL finds rows in one table. Best to worst: `system`/`const` (one row via primary or unique key), `eq_ref` (one row per row of the previous table through a unique index, typical in joins), `ref` (equality lookup on a non-unique index, may return several rows), `range` (index range scan: `BETWEEN`, `<`, `>`, `IN`), `index` (full scan of an index), `ALL` (full table scan).
- **`possible_keys` / `key`** — indexes the optimizer could use vs. the one it chose. `key = NULL` means no index is used.
- **`key_len`** — bytes of the index actually used; tells you how many columns of a composite index take part in the lookup.
- **`rows`** — the optimizer's *estimate* of rows it must examine in this step (not the rows returned).
- **`filtered`** — estimated percentage of examined rows that survive the rest of the `WHERE` after this step. `rows × filtered%` is the estimated number of rows passed to the next join step.
- **`Extra`** — the most informative column: flags for additional work such as `Using index`, `Using where`, `Using filesort`, `Using temporary`.
- **Covering index** — an index containing every column the query needs, so the table itself is never touched (`Using index` in Extra).
- **Filesort** — MySQL's sort step when it cannot read rows in the requested order. The name is misleading: it sorts in memory (the sort buffer) and spills to disk files only when the data is too large.
- **Estimate vs. actual** — `EXPLAIN` numbers come from index statistics; `EXPLAIN ANALYZE` numbers are measured.

### 3. How it works, step by step
1. **Run `EXPLAIN <statement>`.** MySQL parses and optimizes the statement but does not execute it. You get one row per table (or per subquery/derived table) in the order MySQL will read them: the first row is the driving table of a join.
2. **Check the access type of each row.** `ALL` on a large table, or `index` on a wide index, is the first suspect. `ref`/`eq_ref`/`range` are normally good.
3. **Check `key` and `key_len`.** Is the index you expected used? Is only the first column of a three-column index used (short `key_len`)? That means later predicates are not sargable or the column order is wrong.
4. **Multiply the row estimates down the join order.** With nested-loop joins the work is roughly `rows(1) × rows(2) × …` when each step is a lookup. A single step with `rows = 2,000,000` or a driving table that is not the most selective one is the usual cost center.
5. **Read `Extra`.**

| `Extra` value | Meaning | Usual reaction |
|---|---|---|
| `Using index` | Covering index, table not read | Good; keep |
| `Using where` | A `WHERE` condition is evaluated after rows are read from the engine | Fine alone; suspicious if `rows` is huge because the index did not filter |
| `Using index condition` | Index Condition Pushdown: the storage engine tests part of the condition inside the index | Good |
| `Using filesort` | Extra sort pass | Add or reorder an index to deliver the order, or accept it for small sets |
| `Using temporary` | Internal temporary table (for `GROUP BY`, `DISTINCT`, `UNION`, some derived tables) | Often fixable with an index that matches the grouping columns |
| `Using join buffer (hash join)` | No usable index for the join; MySQL 8 builds a hash table | Add an index on the join column if the join is frequent |

6. **Switch to `EXPLAIN FORMAT=JSON`** when you need `used_key_parts`, per-step `cost_info`, or details of what is hidden behind `Using filesort`/`Using temporary` (for example `using_filesort: true` inside `ordering_operation`, or `using_temporary_table: true`).
7. **Run `EXPLAIN ANALYZE`** on the suspect query (on a safe copy or a read replica for heavy queries; it really executes it, including `SELECT`s that take minutes). The tree shows for each node `cost`, `rows` estimate, `actual time=first_row..last_row`, `rows=` actual and `loops=` number of executions.

### 4. Worked example
```sql
CREATE TABLE orders (
  id         BIGINT PRIMARY KEY,
  customer_id INT NOT NULL,
  status     VARCHAR(12) NOT NULL,
  created_at DATETIME NOT NULL,
  total      DECIMAL(10,2) NOT NULL,
  KEY idx_customer (customer_id)
) ENGINE=InnoDB;

EXPLAIN
SELECT id, total
FROM orders
WHERE customer_id = 42 AND status = 'PAID'
ORDER BY created_at DESC
LIMIT 20;
```
Typical output (abbreviated):

| id | table | type | possible_keys | key | key_len | rows | filtered | Extra |
|---|---|---|---|---|---|---|---|---|
| 1 | orders | ref | idx_customer | idx_customer | 4 | 5200 | 10.00 | Using where; Using filesort |

Reading it: MySQL uses `idx_customer` (`type=ref`, `key_len=4` is one `INT`), expects 5,200 rows for that customer, filters them by `status` (only ~10 % pass) and then sorts the survivors by `created_at`. Two inefficiencies: 5,200 rows are read to keep ~520, and a sort runs even though only 20 rows are wanted.

Fix: an index that matches equality columns first, then the sort column:
```sql
ALTER TABLE orders ADD KEY idx_cust_status_created (customer_id, status, created_at);
EXPLAIN SELECT id, total FROM orders
WHERE customer_id = 42 AND status = 'PAID'
ORDER BY created_at DESC LIMIT 20;
```
Now the plan shows `key = idx_cust_status_created`, `key_len = 4 + 50 = 54` (the `INT` is 4 bytes; a `NOT NULL VARCHAR(12)` in utf8mb4 is 12×4 + 2 length bytes; +1 byte per nullable column), `rows ≈ 520`, and `Extra` contains `Backward index scan` instead of `Using filesort`: the index order *is* the sort order, so `LIMIT 20` stops after 20 index entries.

Confirm with `EXPLAIN ANALYZE`:
```
-> Limit: 20 row(s)  (cost=... rows=20) (actual time=0.05..0.12 rows=20 loops=1)
    -> Index lookup on orders using idx_cust_status_created (customer_id=42, status='PAID') (reverse)
       (cost=... rows=520) (actual time=0.05..0.11 rows=20 loops=1)
```
The `actual rows=20` shows the limit is applied while scanning, not after reading all 520.

### 5. Under the hood
- **Statistics.** InnoDB estimates `rows` from random dives into index pages (`innodb_stats_persistent_sample_pages`, default 20) and cardinality statistics. Skewed data (one customer with 5 M rows) produces estimates that are far off; `ANALYZE TABLE` refreshes statistics, and MySQL 8 histograms (`ANALYZE TABLE t UPDATE HISTOGRAM ON col`) help for *non-indexed* columns used in filters.
- **Cost model.** The optimizer assigns costs from `mysql.server_cost` and `mysql.engine_cost` (row evaluate cost, key compare cost, page read cost, memory vs. disk temp table cost). Choosing between `ref` on a poor index and a full scan is a cost comparison, not a rule.
- **Join algorithms.** Nested-loop join is the default; MySQL 8.0.18+ replaces block-nested-loop with hash join for equi-joins without usable indexes. Merge join does not exist in MySQL. `EXPLAIN ANALYZE` names the algorithm in the tree.
- **Filesort variants.** If the rows fit in `sort_buffer_size` it is an in-memory quicksort; otherwise it writes sorted runs to temp files and merges them. `Sort_merge_passes` in `SHOW STATUS` counts merge passes. With `LIMIT n` MySQL uses a priority queue of n rows and never sorts everything.
- **Temporary tables.** Internal temp tables start in memory (the `TempTable` engine in 8.0, capped by `temptable_max_ram`) and spill to disk (InnoDB) when too large; `Created_tmp_disk_tables` rises.
- **EXPLAIN ANALYZE** is built on the iterator executor: it prints only the tree format, works for `SELECT` (and `TABLE`; multi-table `UPDATE`/`DELETE` from 8.0.19), and instruments each iterator with timers, which adds a little overhead of its own.

### 6. Performance and design implications
- Optimize for **rows examined**, not for "uses an index". An index scan over 2 M entries is still slow; compare `rows` with the rows you return.
- Estimates diverging from actuals by 10× or more (`rows=100` vs. `actual rows=1,000,000`) point to stale or skewed statistics, correlated predicates, or a function wrapped around an indexed column. Remedies in order: `ANALYZE TABLE`, histogram, rewrite the predicate to be sargable (`created_at >= '2026-01-01'` instead of `YEAR(created_at) = 2026`), optimizer hint (`USE INDEX`, `JOIN_ORDER`) as a last resort.
- `loops` in `EXPLAIN ANALYZE` multiplies `actual time`: a node with `actual time=0.01` and `loops=500000` costs 5 s.
- Use the slow query log (`long_query_time`, `log_queries_not_using_indexes`) and `performance_schema.events_statements_summary_by_digest` to *find* the queries, then `EXPLAIN` them; `SHOW STATUS LIKE 'Handler_read%'` after running a query shows how many rows were read via each path.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `EXPLAIN SELECT …` | Estimated plan, one row per table | First look at any query |
| `EXPLAIN FORMAT=TREE` | Iterator tree like ANALYZE but without running | See join algorithm and operator nesting |
| `EXPLAIN FORMAT=JSON` | Costs, used key parts, filesort/temporary details | Need hidden detail |
| `EXPLAIN ANALYZE …` | Runs the statement, shows actual time/rows/loops | Estimates look wrong; verify a fix |
| `EXPLAIN FOR CONNECTION <id>` | Plan of a statement currently running in another session | Diagnose a stuck long query |
| `ANALYZE TABLE t` | Refresh index statistics | Plan changed after bulk load |
| `SHOW INDEX FROM t` | Cardinality and column order of indexes | Check selectivity |
| `SET optimizer_switch='…'` | Toggle optimizer features (ICP, hash join, etc.) | Controlled experiments |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `EXPLAIN` — query is huge/slow or on production and you only need the shape | You need to know real row counts; estimates can mislead |
| `EXPLAIN ANALYZE` — you want ground truth and can afford one execution | Statement is a multi-minute report on a primary; run on replica or with a `LIMIT`/sample; note that it executes data-reading side effects of functions |
| Add a composite index to remove filesort | The table is write-heavy and the index serves one rare query |
| Rewrite query | The optimizer is right and the data volume is genuinely large; consider summary tables or partitioning |

### 9. Common mistakes
- Reading only `type` and `key` → missing `Using temporary; Using filesort` in Extra → look at Extra every time.
- Treating `rows` as the result size → it is rows *examined*; compare with `filtered`.
- Concluding "an index is used so it is fine" → a wide `index` scan or a low-selectivity `ref` still reads a lot → check `rows` × `loops`.
- Running `EXPLAIN` on a tiny dev table → optimizer picks a full scan because it is cheaper there → test with production-sized data or copy statistics.
- Wrapping indexed columns in functions or implicit conversions (`WHERE varchar_col = 123`) → index is ignored → match types and keep columns bare.
- Running `EXPLAIN ANALYZE` casually on production reports → full executions pile up → use a replica.

### 10. 🎯 Interview kit
**30-second answer:** "`EXPLAIN` shows the optimizer's plan without running the query: for each table the access type, the index chosen, estimated rows and `Extra` flags like filesort or temporary. I look for `ALL` scans, large `rows`, and `Using filesort`/`Using temporary`. `EXPLAIN ANALYZE` actually executes the query and shows real time, row counts and loops, so I can see where estimates are wrong."

**Structure a longer answer:** clarify what "slow" means (latency, rows, load) → get the plan (`EXPLAIN`, then `ANALYZE`) → walk tables in join order checking type/key/rows/Extra → name the fix (index, rewrite, statistics) → verify with `EXPLAIN ANALYZE` and compare before/after → mention monitoring (slow log, performance_schema).

**Likely follow-ups:**
- *What does `Using filesort` mean and how do you remove it?* → An extra sort pass because no index gives the order; create an index with equality columns first then the `ORDER BY` columns, with matching direction (MySQL 8 supports descending indexes).
- *`Using temporary` — what triggers it?* → `GROUP BY`/`DISTINCT` on non-indexed or differently ordered columns, `UNION` (not `UNION ALL`), some derived tables/subqueries; fix with an index matching the grouping or by rewriting.
- *EXPLAIN vs. EXPLAIN ANALYZE?* → estimated vs. measured; ANALYZE executes the query.
- *Estimated and actual rows differ by 100×?* → refresh statistics, add a histogram, check for skew/correlated columns, make predicates sargable, as last resort hints.
- *What does `key_len` tell you?* → how many leading columns of a composite index are used.

**Signals interviewers look for:** you read `Extra`, you talk about rows examined rather than "has an index", you know the estimate/actual distinction, and you validate the fix with a measurement.

### 11. Check yourself
- [ ] List the `type` values from best to worst and say what each means.
- [ ] Given a plan with `key_len=4` on a 3-column index, explain what it implies.
- [ ] Rewrite `WHERE DATE(created_at) = '2026-01-01'` so it can use an index.
- [ ] Explain why `ORDER BY a ASC, b DESC` can still require a filesort on pre-8.0 versions.
- [ ] Run `EXPLAIN ANALYZE` on a join and find the node with the largest `actual time × loops`.
- [ ] Explain when `Using index` and `Using where` appear together and why that is not a contradiction.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../_index.md>)

## Practice questions

- [Basic] [What are the main categories of SQL commands (DDL, DML, DCL, TCL) and give one example of each?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33337_What_are_the_main_categories_of_SQL_commands_DDL_DML_DCL_TCL.md>)
- [Intermediate] [Explain the ACID properties of a transaction with a real example.](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33036_Explain_the_ACID_properties_of_a_transaction_with_a_real_exa.md>)
- [Intermediate] [How would you read the output of EXPLAIN and what do the type, key, rows, and Extra columns tell you?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33060_How_would_you_read_the_output_of_EXPLAIN_and_what_do_the_typ.md>)
- [Deep] [How would you use EXPLAIN FORMAT=JSON to spot a hidden filesort or a temporary table that the classic EXPLAIN output hides?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33470_How_would_you_use_EXPLAIN_FORMAT_JSON_to_spot_a_hidden_files.md>)
- [Deep] [What does 'Using filesort' in EXPLAIN mean and how would you eliminate it?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33061_What_does_Using_filesort_in_EXPLAIN_mean_and_how_would_you_e.md>)
- [Deep] [What does 'Using temporary' in EXPLAIN indicate and which operations commonly trigger it?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33062_What_does_Using_temporary_in_EXPLAIN_indicate_and_which_oper.md>)
- [Deep] [What is the difference between EXPLAIN and EXPLAIN ANALYZE in MySQL 8 and when do you prefer each?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33063_What_is_the_difference_between_EXPLAIN_and_EXPLAIN_ANALYZE_i.md>)
- [Deep] [How would you use EXPLAIN to diagnose why a query is slow?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33136_How_would_you_use_EXPLAIN_to_diagnose_why_a_query_is_slow.md>)

## Related concepts

- [Reading Execution Plans](<../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Reading_Execution_Plans.md>) *(Execution Plans)*
- [Reading EXPLAIN ANALYZE](<../../../01_Databases_and_SQL/Query_Performance/Indexing/Reading_EXPLAIN_ANALYZE.md>) *(Indexing)*
- [Reading EXPLAIN](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Reading_EXPLAIN.md>) *(SQL Fundamentals)*

## References

- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Basic Operations on Delta Tables › The DESCRIBE Statement

