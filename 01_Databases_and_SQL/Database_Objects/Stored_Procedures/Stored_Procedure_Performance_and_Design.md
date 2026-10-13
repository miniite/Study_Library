---
id: "01_Databases_and_SQL/Database_Objects/Stored_Procedures/Stored_Procedure_Performance_and_Design"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Objects/Stored_Procedures"
tools: ["SQL_Server"]
---

# Stored Procedure Performance and Design

**L4 Advanced** · [↑ Stored Procedures](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A fast, maintainable procedure comes from plan-cache-friendly code (stable text, sensible parameters, no needless recompiles), set-based statements, short transactions and idempotent, well-named, testable design — not from the mere fact that it is a "stored procedure".

### 1. The big picture
Stored procedures are often described as "fast because they're precompiled". The precise truth (Fritchey): a procedure's statements are compiled when first executed, a plan is cached per statement, and later calls reuse that plan with different parameter values. The real performance wins come from (a) plan reuse and parameterisation, (b) keeping logic near the data and moving less over the network, (c) one call instead of many round trips, and (d) a stable, tunable unit you can monitor. The real losses come from the same mechanisms misused: parameter sniffing, unnecessary recompiles, row-by-row loops, scalar functions, giant monolithic procedures, long transactions and unbounded result sets.

This lesson covers the performance model (plan cache, recompilation, sniffing, SET options), concrete coding practices (`SET NOCOUNT ON`, set-based logic, batching, temp tables vs table variables), design practices (re-runnable ETL procedures, naming, size, documentation, denormalise-vs-optimise decisions) and how to find the expensive procedures.

### 2. Core concepts and vocabulary
- **Plan cache** — memory area holding compiled plans keyed by statement text/object and SET options.
- **Compile vs recompile** — building a plan first time vs rebuilding for an existing procedure/statement (schema change, statistics change, `RECOMPILE`).
- **Parameter sniffing** — optimizer uses the first call's parameter values to estimate; plan reused for others.
- **Plan reuse / parameterisation** — executing the same statement text with different values using one plan.
- **Deferred object resolution / deferred compile** — statements referencing objects that don't exist yet (temp tables) compile at execution.
- **`SET NOCOUNT ON`** — stop the engine sending a "rows affected" message per statement.
- **Logical reads** — pages read from memory; **elapsed/CPU time**; **duration** — measures to rank procedures.
- **Idempotent procedure** — can be run repeatedly with the same input and produce the same final state.
- **Batching / chunking** — splitting big modifications into smaller transactions.
- **Sequence/identity caching** — pre-allocating blocks of generated numbers to avoid a logged update per value.
- **DETERMINISTIC / NOT DETERMINISTIC** (MySQL, Oracle) — declares whether a routine always yields same output for same input; affects binary logging safety, function-based index and result caching eligibility.
- **Inlining** — engine expands a function's body into the calling query (SQL Server 2019 scalar UDF inlining, inline TVFs), letting the optimizer see through it.
- **Query Store / AWR / DMVs** — built-in telemetry.

### 3. How it works, step by step
**Life of a plan**
1. First `EXEC`: statements compile; each statement's plan is cached with the SET options in effect.
2. Subsequent `EXEC`: cache lookup; plan reused (cheap) unless something forces recompilation.
3. Recompile triggers: schema changes to referenced objects, statistics updated beyond threshold, `sp_recompile`, `WITH RECOMPILE`/`OPTION (RECOMPILE)`, temp-table row-count thresholds, differing SET options (e.g. `ARITHABORT`, `ANSI_NULLS`) producing separate cache entries.
4. Cache eviction under memory pressure; restart/failover clears the cache.

**Parameter sniffing in action**
1. First call `@Customer = 7` (5 rows) → optimizer chooses seek + nested loops.
2. Plan cached. Next call `@Customer = 1` (2 million rows) reuses nested-loop plan → millions of lookups.
3. The reverse also occurs: a scan/hash plan compiled for a big customer penalises tiny ones.
4. Symptom: same procedure sometimes instant, sometimes minutes; after `sp_recompile` it "fixes itself" until the next unlucky first call.

**Systematic tuning**
1. Rank procedures by total cost (`sys.dm_exec_procedure_stats`: `total_worker_time`, `total_logical_reads`, `execution_count`) — cheap-but-frequent often beats rare-but-slow.
2. Inside the top procedure, rank *statements* (`sys.dm_exec_query_stats` with statement offsets or Query Store).
3. Get the actual plan with representative parameters (several!); compare estimated vs actual rows.
4. Fix the root cause: index/predicate, set-based rewrite, statistics, sniffing strategy, transaction scope.
5. Re-measure logical reads and duration; keep a before/after record.

### 4. Worked example
```sql
-- BEFORE: chatty, row-by-row, sniffing-prone
CREATE PROCEDURE dbo.usp_ArchiveOldOrders @Cutoff DATE
AS
    DECLARE @id INT;
    DECLARE c CURSOR FOR SELECT OrderID FROM dbo.Orders WHERE OrderDate < @Cutoff;
    OPEN c; FETCH NEXT FROM c INTO @id;
    WHILE @@FETCH_STATUS = 0 BEGIN
        INSERT dbo.OrdersArchive SELECT * FROM dbo.Orders WHERE OrderID = @id;
        DELETE dbo.Orders WHERE OrderID = @id;
        FETCH NEXT FROM c INTO @id;
    END
    CLOSE c; DEALLOCATE c;
GO

-- AFTER: set-based, batched, restartable, minimal logging, no cursor
CREATE OR ALTER PROCEDURE dbo.usp_ArchiveOldOrders
    @Cutoff DATE, @BatchSize INT = 5000
AS
BEGIN
    SET NOCOUNT ON; SET XACT_ABORT ON;
    DECLARE @moved INT = 1;

    WHILE @moved > 0
    BEGIN
        BEGIN TRAN;

        DELETE TOP (@BatchSize) o
        OUTPUT deleted.OrderID, deleted.OrderDate, deleted.CustomerID, deleted.Total
        INTO dbo.OrdersArchive (OrderID, OrderDate, CustomerID, Total)
        FROM dbo.Orders o
        WHERE o.OrderDate < @Cutoff;

        SET @moved = @@ROWCOUNT;
        COMMIT;                                    -- short transactions, log can truncate
    END
END;
```
Why this is better: one `DELETE … OUTPUT INTO` moves rows atomically per batch (no window between copy and delete), no per-row statement overhead, the transaction log stays small (5,000-row chunks), locks are held briefly, and it is **re-runnable**: if it dies midway, rerunning continues with what remains because "not yet archived" is defined by data, not by in-memory state. An index on `Orders(OrderDate)` makes each batch a seek.

Typical measurements to show: before — 4 M iterations, CPU 38 min, log 40 GB; after — about 800 batches, a few minutes, log reused. Verify with `SET STATISTICS IO,TIME ON` on one batch and `sys.dm_exec_procedure_stats` before/after.

A sniffing fix pattern:
```sql
-- keep the plan stable but data-aware per statement
SELECT ... FROM dbo.Orders WHERE CustomerID = @CustomerID
OPTION (OPTIMIZE FOR (@CustomerID UNKNOWN));      -- average-density plan
-- or: OPTION (RECOMPILE)                         -- compile per call (OK for infrequent/variable queries)
```

### 5. Under the hood
- **Statement-level caching.** Plans are per statement within the batch; a single bad statement can be recompiled on its own. Extended Events show `recompile_cause` (schema changed, statistics changed, deferred compile, option RECOMPILE).
- **Temp tables and recompiles.** A temp table created in the same procedure triggers deferred compile for statements that use it; once row counts cross thresholds the statements recompile based on updated statistics. Table variables don't have statistics (a fixed estimate of 1 row until deferred compilation in SQL Server 2019+) so they suit tiny sets but mislead the optimizer with large ones; temp tables carry statistics and indexes and are usually better for big intermediate results.
- **`SET NOCOUNT ON`.** Each DML statement otherwise sends a `DONE_IN_PROC` token to the client. For a procedure with dozens of statements called thousands of times per second this is non-trivial network and client-side processing, and some clients treat extra row-count messages as result sets. It does not affect `@@ROWCOUNT`.
- **SET option plan fragmentation.** Different connection settings (`ARITHABORT`, `ANSI_WARNINGS`, `DATEFIRST`, `LANGUAGE`) create separate plans for the same procedure; the classic "fast in SSMS, slow in the app" mystery is a different plan from different options (and sniffing).
- **Sequence/identity caching.** Generating values requires a persisted update of the generator; caching N values in memory (`CACHE 1000` for sequences; identity cache) amortises that cost, increasing insert throughput at the price of gaps after a crash/restart. Don't rely on gap-free numbering.
- **Functions.** Pre-2019 SQL Server scalar UDFs run once per row, are not inlined, and force serial plans; inline TVFs are expanded into the query like views. Modern engines inline simple scalar functions automatically, but features like non-deterministic calls, table access with complex logic or `GETDATE()` can block inlining. `DETERMINISTIC` in MySQL/Oracle is a promise: wrong claims cause wrong replication results or stale cached values.
- **Join-order problems.** Bad join orders mostly come from bad cardinality estimates (statistics, correlated predicates, multi-statement TVF/table variable). Remedies in order: update/create statistics (or multi-column), fix predicates and types, add indexes, simplify the query (stage an intermediate in a temp table), then — as a last resort — hints (`FORCE ORDER`, `LOOP/HASH JOIN`, `USE PLAN`). Hints freeze a decision that was right on today's data and wrong in a year; document them and review.
- **Denormalise vs optimise.** First fix the access path and query shape; denormalise only when the join cost is inherent (very hot read path, joining the same dimensions every time) and you accept write amplification and consistency work. Alternatives: indexed/materialized views, summary tables, covering indexes.
- **Unbounded result sets.** Returning 100,000 rows to show 20 wastes server, network and client; push paging into the procedure (keyset/`OFFSET-FETCH`) and consider a row-count cap.

### 6. Performance and design implications
**Coding**
- `SET NOCOUNT ON`; schema-qualify; avoid `SELECT *`; use set-based logic; avoid cursors/loops except batching.
- Keep predicates sargable (no functions on columns, matching types); avoid `OR` catch-alls without `RECOMPILE`/dynamic SQL.
- Avoid string concatenation in loops (quadratic cost, CPU burn): use `STRING_AGG`/set operations.
- Pass lists as TVP/array rather than comma strings; if you must split, split once into a temp table.
- Break huge `UPDATE/DELETE` into batches of a few thousand rows to limit log, locks and rollback time; use an indexed predicate for each batch.
- Keep transactions as small as possible and acquire locks in a consistent order to reduce deadlocks.

**Design**
- **Single responsibility**: a procedure does one job; orchestration procedures call small procedures. Don't split so far that call overhead and transaction nesting hide the logic ("over-splitting").
- **Idempotent/re-runnable ETL**: use staging, high-water marks or status columns, `MERGE`/delete-insert by key, run IDs, logging table with step status; a rerun must not duplicate or skip.
- **Naming**: `usp_<Verb><Noun>` (GetCustomerOrders, ArchiveOldOrders), avoid vague verbs, enforce via code review/static analysis (e.g. in CI with a script that queries `sys.procedures`).
- **Documentation**: header comment with purpose, parameters, return codes, side effects, change log; inline comments for non-obvious decisions.
- **Version & deploy**: migrations in source control, `CREATE OR ALTER`, automated tests (tSQLt), rollback scripts.
- **Observe**: Query Store (regressed queries, forced plans), `sys.dm_exec_procedure_stats`, Oracle AWR "SQL ordered by …", PostgreSQL `pg_stat_statements` + `auto_explain`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SET NOCOUNT ON` | Suppress row-count messages | First line of almost every proc |
| `OPTION (RECOMPILE)` / `WITH RECOMPILE` | Fresh plan per execution | Highly variable parameters, rare execution |
| `OPTION (OPTIMIZE FOR (@p = v / UNKNOWN))` | Control sniffed value | Stable plan for typical value |
| `sp_recompile 'proc'` | Mark for recompile | Emergency fix / after bulk change |
| `sys.dm_exec_procedure_stats` | Per-procedure cost counters | Ranking expensive procedures |
| `sys.dm_exec_query_stats` + `sys.dm_exec_text_query_plan` | Statement-level cost & plans | Drill into a procedure |
| Query Store (`sp_query_store_force_plan`) | History, regressions, plan forcing | Production plan stability |
| `SET STATISTICS IO, TIME ON` | Logical reads, CPU, elapsed | Before/after benchmarking |
| `CREATE SEQUENCE … CACHE n` | Cached number generation | High-insert throughput |
| `DELETE TOP (n) … OUTPUT … INTO` | Batched move | Archival/purge |
| `DETERMINISTIC` / `NOT DETERMINISTIC` | Routine determinism declaration | MySQL/Oracle function correctness & optimisation |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| **Cached plan reuse** (default) — stable-selectivity parameters, high call rate | Skewed data where one value's best plan is awful for another |
| **`OPTION (RECOMPILE)`** — infrequent/analytic procedures, catch-all searches | Thousands of calls/sec (compile CPU) |
| **`OPTIMIZE FOR UNKNOWN` / local variables** — avoid extremes of sniffing | You want value-specific plans for known hot values |
| **Plan forcing (Query Store)** — immediate stabilisation of regressions | Data changes require different plans; forced plan becomes wrong |
| **Join hints** — one proven pathological join | As a general habit; they disable optimizer adaptation |
| **Temp table** — intermediates over a few hundred rows, need stats/indexes | Tiny sets, very high frequency (creation overhead) |
| **Table variable** — small sets, fewer recompiles | Large sets without deferred compilation |
| **Denormalisation** — hot read path with immutable-ish data | Frequent updates, many writers, limited governance |
| **Batching DML** — big maintenance operations | Atomic all-or-nothing business semantics required |
| **Splitting procedure** — reuse and testability | Excessive nesting, hidden transaction coupling |

### 9. Common mistakes
- **Believing "precompiled = always fast"** → ignoring sniffing and plan variance → monitor and handle.
- **Cursor/loop instead of set-based** → CPU and log explosion → single statements, batches.
- **Giant single transaction** → log growth, blocking, long rollback → batch and commit.
- **String concatenation in a loop** → quadratic CPU → set-based aggregation.
- **Scalar UDF in WHERE/SELECT on large tables** → per-row execution, serial plan → inline logic or inline TVF.
- **Splitting CSV at runtime in every call** → CPU waste → TVP/temp table.
- **Forgetting `SET NOCOUNT ON`** → extra messages, ADO quirks → set it.
- **Catch-all `@p IS NULL OR col=@p` without RECOMPILE/dynamic SQL** → one bad plan for all → fix pattern.
- **Hinting first** → brittle plans → fix statistics, predicates, indexes first.
- **Returning full result sets for pagination at client** → network and memory waste → server-side paging.
- **Non-idempotent ETL procedures** → duplicates after restart → status tracking, merge-by-key.
- **Over-indexing as default fix** → write slowdown → verify the plan first.
- **Overly generic names / no standards** → unmaintainable estate → naming convention enforced in review/CI.

### 10. 🎯 Interview kit
**30-second answer:** "Procedures help performance mainly through plan reuse, less network traffic and logic near the data, but they can also hurt via parameter sniffing, recompiles and row-by-row code. I tune them by ranking by total cost, looking at per-statement plans with real parameters, fixing statistics/indexes/predicates, replacing cursors with set-based or batched DML, handling sniffing with RECOMPILE/OPTIMIZE FOR, and keeping transactions short. For ETL I make them idempotent and restartable."

**Structure a longer answer:** where the cost is (measure) → plan cache and sniffing → statement-level fixes → structure (batching, temp tables, functions) → design (re-runnable, naming, size) → monitoring and regression control.

**Likely follow-ups:**
- *What does `SET NOCOUNT ON` do and why?* → Suppresses per-statement row-count messages (`DONE_IN_PROC`), reducing network chatter and client confusion; doesn't change `@@ROWCOUNT`.
- *Bad join order across five tables?* → check estimates/statistics, rewrite or stage intermediates, indexes; hints (`FORCE ORDER`) last, with risk of freezing plans.
- *Denormalise vs optimise the procedure?* → fix plan/indexing first; denormalise when joins are inherent and read-heavy, accept write cost and consistency maintenance.
- *Make a procedure safely re-runnable?* → state tracking/watermarks, idempotent writes (merge/delete-insert by key), transactions per unit, logging, no dependency on session state.
- *Inline logic vs function?* → modern engines inline simple scalar functions; non-inlinable ones are per-row; prefer inline TVFs or inline expressions in hot paths.
- *DETERMINISTIC vs NOT?* → declaration of same output for same input; allows caching/optimisation, safe replication; wrong claim → wrong results.
- *Sequence caching?* → fewer generator persists, higher throughput, gaps after restart.
- *Batching updates?* → smaller log/locks, faster rollback, avoid escalation; trade atomicity.
- *Rank expensive procedures?* → DMVs/Query Store/AWR: total CPU/reads × executions.

**Signals interviewers look for:** you measure before tuning, understand plan cache mechanics, name concrete fixes, and care about idempotency and operability instead of cargo-cult rules.

### 11. Check yourself
- [ ] Explain parameter sniffing with a numeric example and list four mitigations.
- [ ] Rewrite a cursor-based archive procedure into a batched, restartable set-based one.
- [ ] State precisely what `SET NOCOUNT ON` does and doesn't change.
- [ ] Describe how to find the top 5 most expensive procedures and drill to statements.
- [ ] Argue when you'd denormalise instead of tuning the procedure, and what you'd need to maintain consistency.
- [ ] Define an idempotent ETL procedure and list the mechanisms that give you that property.
<!-- explainer:end -->

## Implementations (tools)

- [SQL Server](<../../../TOOLS/SQL_Server/_index.md>)

## Practice questions

- [Basic] [What are some advantages of using stored procedures over writing SQL directly in application code?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Stored_Procedures/Q-41764_What_are_some_advantages_of_using_stored_procedures_over_wri.md>)
- [Intermediate] [A reporting procedure aggregates over a huge fact table each time. How would you use materialized views or summary tables to speed it up?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Stored_Procedures/Q-41674_A_reporting_procedure_aggregates_over_a_huge_fact_table_each.md>)
- [Intermediate] [How do you decide whether to add a NOLOCK-style hint to speed up a reporting procedure, and what would you warn stakeholders about?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Stored_Procedures/Q-41746_How_do_you_decide_whether_to_add_a_NOLOCK-style_hint_to_spee.md>)
- [Deep] [A procedure occasionally times out only for one large customer's data. How would you diagnose data-skew-driven performance problems?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Stored_Procedures/Q-41654_A_procedure_occasionally_times_out_only_for_one_large_custom.md>)
- [Deep] [When would you use MERGE in a procedure over separate INSERT/UPDATE branches, and what performance pitfalls does MERGE have?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Stored_Procedures/Q-41700_When_would_you_use_MERGE_in_a_procedure_over_separate_INSERT.md>)
- [Deep] [A stored procedure that ran in 200ms suddenly takes 30 seconds after a data load. Walk me through how you would diagnose whether it is a plan-caching problem versus a statistics problem.](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Stored_Procedures/Q-41755_A_stored_procedure_that_ran_in_200ms_suddenly_takes_30_secon.md>)
- [Deep] [A procedure returns different result-set shapes depending on a flag parameter. Why is this problematic for callers and how would you redesign it?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Stored_Procedures/Q-41802_A_procedure_returns_different_result-set_shapes_depending_on.md>)
- [Deep] [Explain how you would use SET STATISTICS IO/TIME or EXPLAIN ANALYZE to compare logical reads before and after tuning a procedure.](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Stored_Procedures/Q-41844_Explain_how_you_would_use_SET_STATISTICS_IO_TIME_or_EXPLAIN_.md>)

## Related concepts

- [UDF Performance](<../../SQL_Language/SQL_Functions/UDF_Performance.md>) *(SQL Functions)*
- [Filtered Indexes and Plan Cache](<../../../TOOLS/SQL_Server/T-SQL/Filtered_Indexes_and_Plan_Cache.md>) *(T-SQL)*
- [Read Committed Locking Anomalies](<../../../TOOLS/SQL_Server/SQL_Server_Platform/Read_Committed_Locking_Anomalies.md>) *(SQL Server Platform)*
- [Dynamic SQL](<../../Database_Internals/DBMS_Concepts/Dynamic_SQL.md>) *(DBMS Concepts)*
- [Clustered Indexes and Indexed Views](<../../../TOOLS/SQL_Server/SQL_Server_Platform/Clustered_Indexes_and_Indexed_Views.md>) *(SQL Server Platform)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Functions vs. Views vs. Stored Procedures
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Stored Procedures
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Performance Considerations
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Best Practices for Stored Procedures
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.)
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 6 Using The Query Store For Query Performance And Execution Plans
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 8 Query Recompilation
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 19 Memory-Optimized Oltp Tables And Procedures
- *Designing Data-Intensive Applications* — Martin Kleppmann — S
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — Refactoring Data Access You'll recall we presented a design for the Reservation Service in Chapter
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Thinking About Performance › Solutions to Exercise 13-4: Stored Procedures
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Programmable objects
- *SQL Server Tutorial* (YouTube series) — Part 21: Advantages of stored procedures ([watch](https://www.youtube.com/watch?v=uDcVd4vUU3s))
- *SQL Server Tutorial* (YouTube series) — Part 147: Dynamic SQL vs Stored Procedure ([watch](https://www.youtube.com/watch?v=imSuP9vSjfk))

