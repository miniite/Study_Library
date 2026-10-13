---
id: "TOOLS/Oracle/Oracle_Platform/Oracle_SQL_and_Indexes"
level: 4
type: "concept"
topic: "TOOLS/Oracle/Oracle_Platform"
tools: ["Oracle"]
---

# Oracle SQL and Indexes

**L4 Advanced** · [↑ Oracle Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Oracle SQL is ANSI SQL plus Oracle-specific objects (schemas = users, `DUAL`, sequences, GTTs), and performance depends on indexes, optimizer statistics and plan stability features such as bind peeking, adaptive cursor sharing and SQL plan baselines.

### 1. The big picture
Writing correct SQL is the entry ticket; writing SQL that the Oracle optimizer can execute quickly is the job. This lesson ties together the basics (schemas, constraints, joins, set operators, DDL/DML differences, views, indexes) with the performance mechanics: join methods, index types, tracing with SQL Trace/TKPROF, global temporary tables, and the features that stabilize plans (adaptive cursor sharing and SQL plan management).

### 2. Core concepts and vocabulary
- **Database vs. instance** — the database is the files on disk (data files, control files, redo logs); the instance is memory (SGA) plus background processes that open it. One database may be opened by several instances (RAC).
- **Schema** — the collection of objects owned by a database user; in Oracle schema = user, one-to-one.
- **`DUAL`** — one-row, one-column table for evaluating expressions (`SELECT SYSDATE FROM dual`).
- **Constraint** — `NOT NULL`, `UNIQUE`, `PRIMARY KEY`, `FOREIGN KEY`, `CHECK`; PK/UNIQUE are backed by an index.
- **DDL / DML / DCL / TCL** — define / manipulate / grant / transaction control.
- **`DELETE` vs. `TRUNCATE` vs. `DROP`** — row removal with undo and WHERE (DML, rollbackable) / deallocate all rows quickly, no WHERE, DDL with implicit commit, resets the high-water mark / remove the object.
- **B-tree index** — default, balanced tree, good for high-cardinality equality and range; **bitmap index** — one bitmap per distinct value, for low-cardinality columns in read-mostly warehouses; **function-based index** — on an expression; **composite index** — several columns.
- **Join methods** — nested loops, hash join, sort-merge join.
- **Bind peeking** — optimizer inspects the bind value at first hard parse to choose a plan.
- **Adaptive cursor sharing (ACS)** — marks cursors "bind-sensitive" and creates multiple child cursors with different plans per selectivity range.
- **SQL Plan Baseline** — an accepted set of plans for a statement, managed by SQL Plan Management (SPM).
- **GTT (global temporary table)** — permanent definition, session-private (or transaction-private) data.

### 3. How it works, step by step
**Join methods**
1. **Nested loops**: for each row from the outer (driving) row source, probe the inner via an index. Best when the outer set is small and the inner has a selective index; returns first rows fast.
2. **Hash join**: build an in-memory hash table on the smaller input, scan the larger input and probe. Needs an equality join; good for large unsorted sets; spills to temp if the build side exceeds PGA workarea.
3. **Sort-merge**: sort both inputs and merge. Useful for non-equality (`<`, `>`) joins or already-sorted inputs.
The optimizer picks the cheapest by estimated cost from statistics (cardinality, selectivity, clustering factor, I/O and CPU costs).

**Statement lifecycle:** parse (syntax → semantic → soft/hard parse via shared pool hash on SQL text) → optimize → row source generation → execute → fetch.

**Tracing a slow PL/SQL batch**
1. Enable trace for the session: `EXEC DBMS_MONITOR.SESSION_TRACE_ENABLE(session_id=>:sid, serial_num=>:ser, waits=>TRUE, binds=>FALSE);` (or `ALTER SESSION SET sql_trace = TRUE` in your own session; set `tracefile_identifier`).
2. Run the job; disable trace.
3. Find the file in `V$DIAG_INFO` ("Default Trace File"), run `tkprof trace.trc out.txt sys=no sort=exeela,fchela`.
4. In the report, per statement see parse/execute/fetch counts, `cpu`, `elapsed`, `disk`, `query`, `current`, `rows`, plus the row-source plan with actual rows. Look for statements with huge `query` (logical reads) per row, many executes of a tiny statement (row-by-row processing → bulk it), and wait events summary.
5. Alternatively, use `DBMS_PROFILER`/`DBMS_HPROF` to find the slow PL/SQL line.

**Plan stability**
1. Capture: `DBMS_SPM.LOAD_PLANS_FROM_CURSOR_CACHE(sql_id=>…)` or automatic capture (`optimizer_capture_sql_plan_baselines=TRUE`) after the statement runs twice.
2. Evolve: new plans found later are stored as *unaccepted*; `DBMS_SPM.EVOLVE_SQL_PLAN_BASELINE` (or the automatic task in 12c+) tests them and accepts only plans at least as good.
3. Effect: after a stats refresh or upgrade the optimizer may *find* a new plan but will only *use* accepted ones, preventing regressions.

### 4. Worked example
```sql
-- Schema and data
CREATE TABLE customers (cust_id NUMBER PRIMARY KEY, name VARCHAR2(80), region VARCHAR2(10));
CREATE TABLE orders    (order_id NUMBER PRIMARY KEY,
                        cust_id  NUMBER NOT NULL REFERENCES customers,
                        status   VARCHAR2(10),
                        order_dt DATE);
CREATE INDEX orders_cust_ix ON orders(cust_id);
CREATE INDEX orders_status_ix ON orders(status, order_dt);

-- Get the real plan with runtime statistics
SELECT /*+ GATHER_PLAN_STATISTICS */ c.name, COUNT(*)
FROM   customers c JOIN orders o ON o.cust_id = c.cust_id
WHERE  o.status = :s AND o.order_dt >= :d
GROUP  BY c.name;

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY_CURSOR(NULL, NULL, 'ALLSTATS LAST +PEEKED_BINDS'));
```
Read: `Starts`, `E-Rows` (estimated) vs. `A-Rows` (actual), `Buffers` (logical reads) per step; a big E-Rows/A-Rows gap points to stale statistics, skew (needs a histogram) or correlated predicates (extended statistics `DBMS_STATS.CREATE_EXTENDED_STATS`).

**Bind peeking problem.** Suppose `status` has values `'DONE'` (98 %) and `'OPEN'` (0.1 %). If the first execution peeks `'OPEN'` the plan uses the index; later executions with `'DONE'` re-use it and perform millions of single-block reads. ACS (11g+) notices the buffer-gets difference, marks the cursor *bind-aware* and builds a second child cursor with a full scan for the unselective value. Check `V$SQL.IS_BIND_SENSITIVE`, `IS_BIND_AWARE`, `V$SQL_CS_SELECTIVITY`.

**UNION vs. UNION ALL**
```sql
SELECT cust_id FROM orders_2025
UNION            -- sort/hash unique to remove duplicates
SELECT cust_id FROM orders_2026;

SELECT cust_id FROM orders_2025
UNION ALL        -- just appends, faster; use when duplicates impossible or acceptable
SELECT cust_id FROM orders_2026;
```
**BETWEEN** is inclusive of both ends: `x BETWEEN 1 AND 5` includes 1 and 5 (so with `DATE` columns carrying times, `BETWEEN d1 AND d2` misses `d2 13:00`).

**Global temporary table**
```sql
CREATE GLOBAL TEMPORARY TABLE gtt_work (id NUMBER, payload VARCHAR2(100))
ON COMMIT DELETE ROWS;       -- or ON COMMIT PRESERVE ROWS (session lifetime)
```
Rows are visible only to the inserting session and stored in the temp tablespace; the *definition* is permanent. Use when you must stage a result set across multiple statements in one transaction/session without redo-heavy permanent tables. Note: statistics on GTTs are tricky (default is shared across sessions; set session-private stats in 12c+ with `DBMS_STATS.SET_TABLE_PREFS('…','GLOBAL_TEMP_TABLE_STATS','SESSION')`), and a CTE with `/*+ MATERIALIZE */` or a plain inline view is often enough.

### 5. Under the hood
- **B-tree index structure:** root, branch and leaf blocks; leaves store (key, ROWID) sorted and linked bidirectionally. Height is usually 2–4 even for hundreds of millions of rows. Index range scan = descend + walk leaves; `INDEX UNIQUE SCAN` = one probe; `INDEX FAST FULL SCAN` = multiblock read of all leaves; `INDEX SKIP SCAN` = use a composite index without its leading column when that column has few distinct values.
- **Clustering factor** measures how well index order matches table order; a high clustering factor (close to row count) makes range scans via the index costly (many table block visits), and the optimizer then prefers full scans.
- **Table access paths:** `TABLE ACCESS FULL`, `BY INDEX ROWID`, `BY INDEX ROWID BATCHED` (12c).
- **Cardinality estimation** relies on `NUM_ROWS`, `NUM_DISTINCT`, `DENSITY`, histograms; dynamic sampling (`OPTIMIZER_DYNAMIC_SAMPLING`) fills gaps; `STALE_PERCENT` (10 %) governs when auto stats jobs regather.
- **SQL plan baseline** is stored in the SYSAUX `SQL management base`, keyed by a *signature* of the normalized SQL text; fixed/enabled/accepted flags in `DBA_SQL_PLAN_BASELINES`. Compare with SQL Profiles (adjustments to estimates, via SQL Tuning Advisor) and SQL Patches (hints).
- **Why `OR` hurts:** a single index range scan cannot cover predicates on two different columns; Oracle may use `OR`-expansion / `CONCATENATION` but only if costed cheaper. A manual `UNION ALL` rewrite with `LNNVL` on the second branch preserves semantics.

### 6. Performance and design implications
- Choose index column order: equality columns first, then range column; put the most selective first only when they are all equalities.
- Index foreign keys; avoid over-indexing (each index adds DML cost: an `INSERT` writes to every index).
- Keep statistics current: `DBMS_STATS.GATHER_SCHEMA_STATS(… cascade=>TRUE)`; after bulk loads gather explicitly; lock stats for volatile staging tables when a representative profile is better.
- Use bind variables for OLTP; consider literals (or `CURSOR_SHARING=EXACT` with histograms) for rare skewed reporting queries.
- Prefer `MERGE` for upsert, `INSERT /*+ APPEND */` for direct-path loads, set-based `UPDATE … WHERE EXISTS`.
- `COUNT(*)` on big tables is a scan; maintain counters or use approximate counts (`APPROX_COUNT_DISTINCT`).
- Verify before/after with `V$SQL` (`elapsed_time/executions`, `buffer_gets/executions`) or ASH.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `EXPLAIN PLAN FOR …` + `DBMS_XPLAN.DISPLAY` | Estimated plan | Quick look |
| `DBMS_XPLAN.DISPLAY_CURSOR(sql_id,,'ALLSTATS LAST')` | Actual plan with row counts | Real diagnosis |
| `DBMS_MONITOR.SESSION_TRACE_ENABLE` + `tkprof` | Trace file → report | Slow batch jobs |
| `DBMS_STATS.GATHER_TABLE_STATS` | Statistics and histograms | After data changes |
| `ALTER INDEX … REBUILD ONLINE` | Rebuild | Rarely; not as a routine |
| `CREATE INDEX … INVISIBLE` | Test index without optimizer use | Safe experiments |
| `DBMS_SPM.LOAD_PLANS_FROM_CURSOR_CACHE` | Create baseline | Stabilize a good plan |
| `ALTER SESSION SET optimizer_adaptive_features` / `optimizer_adaptive_plans` | Adaptive behaviors | Controlled testing |
| `TRUNCATE TABLE t [REUSE STORAGE]` | Fast empty | Staging tables |
| `GRANT / REVOKE` | Privileges (object/system), roles bundle privileges | Security administration |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Nested loops | Both inputs are large and no good index |
| Hash join | Non-equality conditions; very small outer set with an index |
| Sort-merge | Inputs unsorted and large equality join (hash is usually better) |
| Bitmap index | OLTP with concurrent DML (locks many rows per bitmap) |
| Plan baseline | The workload and data legitimately change a lot (stale baseline) |
| GTT | Single-statement logic (use a subquery/CTE); highly concurrent sessions needing stats |
| `TRUNCATE` | Need rollback or partial delete |

### 9. Common mistakes
- Using `DELETE FROM big_table` to empty → undo/redo heavy and slow → `TRUNCATE` when rollback is not needed.
- `WHERE UPPER(name) = :n` with only a plain index → full scan → function-based index or normalized column.
- Comparing `DATE` with `BETWEEN` and ignoring time → missed rows → half-open range.
- Leaving stale statistics after a bulk load → poor plans → gather stats.
- "Fixing" with hints everywhere → brittle code → fix statistics/indexes first, hints or baselines last.
- Not checking `A-Rows` vs. `E-Rows` → tuning by guesswork → use `ALLSTATS LAST`.
- Single-row inserts/updates inside PL/SQL loops → millions of context switches → `FORALL`/`MERGE`.

### 10. 🎯 Interview kit
**30-second answer:** "Oracle's optimizer is cost-based; it picks join methods and access paths from statistics. Nested loops suit small outer sets with indexed probes, hash joins suit big equality joins, sort-merge suits non-equi joins or sorted inputs. I diagnose with `DBMS_XPLAN` actual plans, SQL Trace/TKPROF for batch jobs, and keep plans stable with bind-aware cursor sharing and SQL plan baselines."

**Structure a longer answer:** statement lifecycle → access paths and indexes → join methods and costing → tools to measure → stability features → rules of thumb (binds, stats, set-based).

**Likely follow-ups:**
- *Instance vs. database?* → memory+processes vs. files.
- *`WHERE` vs. `HAVING`; inner vs. outer join; DELETE/TRUNCATE/DROP; PK vs. unique?* → standard answers; PK = unique + not null, one per table, unique allows NULLs and many.
- *What is adaptive cursor sharing?* → bind-aware multiple children per selectivity.
- *What is a SQL plan baseline?* → whitelist of accepted plans; new plans evolve before use.
- *How does TKPROF help?* → per-statement parse/exec/fetch statistics, wait events, row source plan with actuals.
- *Heap vs. global temporary table?* → permanent regular table with persistent data vs. session-private data in temp space.

**Signals interviewers look for:** reading plans with actual rows, understanding why cardinality estimates go wrong, and choosing the minimal stable fix.

### 11. Check yourself
- [ ] Explain what a schema is in Oracle and how it relates to a user.
- [ ] Choose a join method for three scenarios and justify.
- [ ] Walk through producing and reading a TKPROF report.
- [ ] Explain bind peeking and how ACS mitigates it.
- [ ] Create a plan baseline from the cursor cache and verify it is used.
- [ ] Decide between `DELETE`, `TRUNCATE` and `DROP` for emptying a staging table.
<!-- explainer:end -->

## Implementations (tools)

- [Oracle](<../_index.md>)

## Practice questions

- [Basic] [What is a schema in Oracle and how does it relate to a user?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35098_What_is_a_schema_in_Oracle_and_how_does_it_relate_to_a_user.md>)
- [Basic] [What is the BETWEEN operator in Oracle and is it inclusive or exclusive of its boundaries?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35293_What_is_the_BETWEEN_operator_in_Oracle_and_is_it_inclusive_o.md>)
- [Intermediate] [What does the ON DELETE CASCADE option do on a foreign key in Oracle?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35239_What_does_the_ON_DELETE_CASCADE_option_do_on_a_foreign_key_i.md>)
- [Deep] [What is the difference between a B-tree index and a bitmap index in Oracle?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35246_What_is_the_difference_between_a_B-tree_index_and_a_bitmap_i.md>)
- [Deep] [What is a covering index (index-only scan) in Oracle and how do you design an index so the query never touches the table?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35124_What_is_a_covering_index_index-only_scan_in_Oracle_and_how_d.md>)
- [Deep] [Why is a mass DELETE of millions of rows expensive in Oracle and what alternatives (CTAS, partition drop) do you consider?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35167_Why_is_a_mass_DELETE_of_millions_of_rows_expensive_in_Oracle.md>)
- [Expert] [What is adaptive cursor sharing in Oracle and how does it mitigate the problems of bind variable peeking?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35107_What_is_adaptive_cursor_sharing_in_Oracle_and_how_does_it_mi.md>)
- [Expert] [What is a SQL Plan Baseline in Oracle and how does SPM prevent plan regressions after a stats or upgrade change?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35159_What_is_a_SQL_Plan_Baseline_in_Oracle_and_how_does_SPM_preve.md>)

## Related concepts

- [Index Access Paths](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Index_Access_Paths.md>) *(DBMS Concepts)*
- [Query Rewrites and Plan Stability](<../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Query_Rewrites_and_Plan_Stability.md>) *(Execution Plans)*
- [Filtered Indexes and Plan Cache](<../../SQL_Server/T-SQL/Filtered_Indexes_and_Plan_Cache.md>) *(T-SQL)*
- [Scalar Subqueries](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Scalar_Subqueries.md>) *(SQL Fundamentals)*
- [Dynamic SQL](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Dynamic_SQL.md>) *(DBMS Concepts)*
- [WHERE vs HAVING](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/WHERE_vs_HAVING.md>) *(DBMS Concepts)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › ANSI SQL: The Foundation
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Governance and cataloguing › Oracle
- *Building Data Integration Solutions* — Jay Borthen — Data Stores and Management Systems › Oracle Database

