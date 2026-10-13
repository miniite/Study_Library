---
id: "TOOLS/SQL_Server/T-SQL/Filtered_Indexes_and_Plan_Cache"
level: 4
type: "concept"
topic: "TOOLS/SQL_Server/T-SQL"
tools: ["SQL_Server"]
---

# Filtered Indexes and Plan Cache

**L4 Advanced** · [↑ T-SQL](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A filtered index indexes only a slice of a table (`WHERE` on the index), making it small and cheap — but the optimizer can use it only when it can *prove* the query's predicate falls inside that slice, which interacts badly with parameters and cached plans.

### 1. The big picture
Many tables have skew: 95 % of orders are `Shipped`, 0.1 % are `Pending`; a column is `NULL` for most rows; only active rows are queried. Indexing everything wastes space and write cost. A **filtered nonclustered index** (SQL Server 2008+) stores entries only for rows matching a simple predicate, giving a small, hot, well-statistics'd structure.

The catch is the plan cache: plans for parameterized queries are reused for different parameter values, but a plan that uses a filtered index is valid only for values the filter covers. This lesson connects indexing (B-tree structure, covering, statistics) with query processing (parameter sniffing, plan reuse, recompiles).

### 2. Core concepts and vocabulary
- **Filtered index** — nonclustered index created with `WHERE <simple predicate>`.
- **Filter predicate** — comparisons on columns of the table, combined with `AND`/`OR`/`IN`, constants only.
- **Parameter sniffing** — compile-time use of the first parameter value for estimates.
- **Plan cache / plan reuse** — compiled plans stored by query text and SET options.
- **Simple vs forced parameterization** — whether literals are replaced by parameters automatically.
- **Unmatchable plan** — plan using the filtered index cannot be shared for values outside the filter.
- **Sargable** — predicate form that allows an index seek.
- **Filtered statistics** — statistics with their own `WHERE`.
- **Covering / key lookup** — index contains all needed columns / fetch rest from clustered key.

### 3. How it works, step by step
1. You create: `CREATE INDEX IX_Orders_Pending ON dbo.Orders(OrderDate) INCLUDE (CustomerID) WHERE Status = 'Pending';`
2. Only rows meeting the filter get index entries; DML maintains the index only for those rows.
3. At compile time the optimizer compares the query's `WHERE` against the filter. If the query predicate **implies** the filter (e.g. `Status = 'Pending'`), the index is a candidate.
4. If the query uses a *variable or parameter* (`Status = @s`), the optimizer can't know the value at all compile times, so it generally **will not** use the index for a reusable plan.
5. With `OPTION (RECOMPILE)` or a literal, the value is embedded and the filtered index can be matched.

### 4. Worked example
```sql
CREATE NONCLUSTERED INDEX IX_Orders_Pending
ON dbo.Orders (OrderDate) INCLUDE (CustomerID, Amount)
WHERE Status = 'Pending';

-- A) literal: matches
SELECT OrderDate, CustomerID FROM dbo.Orders
WHERE Status = 'Pending' AND OrderDate >= '20260101';        -- Index Seek on IX_Orders_Pending

-- B) parameter: usually does NOT match in a reusable plan
EXEC sp_executesql
  N'SELECT OrderDate, CustomerID FROM dbo.Orders WHERE Status = @s AND OrderDate >= @d',
  N'@s varchar(10), @d date', @s = 'Pending', @d = '20260101';  -- scan/other index

-- C) fix: embed value at compile time
... WHERE Status = @s AND OrderDate >= @d OPTION (RECOMPILE);   -- can seek filtered index
```
Check with the actual plan: look at the *Index Seek* object name and the "Warnings"/`Unmatched Indexes` entry in the plan XML (`<UnmatchedIndexes>`), which explicitly lists filtered indexes the optimizer could not use due to parameterization.

### 5. Under the hood
- The filter must be **deterministic simple comparisons**: no functions, no `LIKE`, no `BETWEEN`, no computed expressions, no subqueries. `IS NULL`, `IS NOT NULL`, `=`, `<`, `>`, `IN` are fine.
- The optimizer's matching is syntactic/logical implication, not data-driven. `WHERE Status <> 'Done'` can match a filter of `Status = 'Pending'` only if it can derive it — it can't; be explicit.
- Filtered indexes have their own statistics histogram over just the filtered rows → much better cardinality estimates for the rare slice than a full-column histogram with 200 steps.
- Statistics refresh threshold is based on the *whole table* modification counter, so filtered stats can go stale on a table with few changes in the slice — consider manual updates.
- Required SET options (`ANSI_NULLS`, `QUOTED_IDENTIFIER`, etc.) must be ON for DML on a table with filtered indexes, or inserts fail with error 1934.
- Plan cache: ad-hoc vs prepared; parameterization, `optimize for ad hoc workloads`, `sys.dm_exec_cached_plans` and `sys.dm_exec_query_stats` expose the cached objects; different SET options give different plan copies (`sys.dm_exec_plan_attributes`).

### 6. Performance and design implications
- Smaller index → fewer pages, less buffer pool, cheaper writes (only qualifying rows).
- Excellent for: queue tables (`WHERE Processed = 0`), soft deletes (`IsDeleted = 0`), sparse columns, "unique among active rows" constraints (`UNIQUE … WHERE Deleted = 0`).
- Procedures with parameterized filters: use dynamic SQL with literals, branch on parameter (`IF @s = 'Pending'`), or `OPTION (RECOMPILE)` — each with plan-cache trade-offs (compile CPU vs bloat).
- Measure: `sys.dm_db_index_usage_stats`, `sys.dm_db_partition_stats` for size, Query Store for plan regression.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE INDEX … WHERE` | filtered index | skewed/sparse slices |
| `OPTION (RECOMPILE)` | compile with actual values | filtered index + parameters |
| `OPTION (OPTIMIZE FOR (@p = 'x'))` | compile for a specific value | stable plan shape |
| `sp_executesql` / dynamic SQL | literal embedding | controlled parameter variants |
| `DBCC FREEPROCCACHE` / `DBCC FREEPROCCACHE (plan_handle)` | evict plans | testing only |
| `sys.dm_exec_cached_plans`, `dm_exec_query_stats` | inspect cache | diagnosis |
| `PARAMETERIZATION FORCED` | auto-parameterize literals | can *prevent* filtered-index use |
| `UPDATE STATISTICS … WITH FULLSCAN` | refresh filtered stats | stale slice |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| A small, stable slice is queried frequently | Slice values come in as parameters in reusable plans |
| Need uniqueness among a subset | Predicate isn't expressible as simple comparisons |
| Writes to the table are mostly outside the slice | Table has `FORCED` parameterization |
| Many NULLs and queries target non-NULL | Optimizer must pick between many near-identical indexes |

### 9. Common mistakes
- Expecting filtered index use with `@param` → full scan → recompile hint, literal, or branch.
- Filter on column not in the key and not selected → still fine, but query needs the column to *imply* filter in its own `WHERE`.
- Forgetting required SET options → error 1934 on DML → fix connection settings.
- Forced parameterization on → index never matched → disable or use plan guide.
- Using filtered index for `IS NOT NULL` queries with `OR` conditions → not matched → rewrite.
- Index hint on a filtered index that can't satisfy the query → error 8622 → drop hint.

### 10. 🎯 Interview kit
**30-second answer:** A filtered index covers only rows matching a `WHERE` clause, making it small and cheap. The optimizer uses it only if the query predicate provably falls inside the filter, so parameterized queries in cached plans often can't use it unless you recompile or use literals.

**Structure a longer answer:** clarify → what/why → matching rules → plan cache and sniffing issue → fixes → a real scenario (queue table).

**Likely follow-ups:** *Why not used with a variable?* → plan must be valid for all values. *How to detect?* → `UnmatchedIndexes` in plan XML. *Filtered stats vs index?* → stats can exist alone. *Uniqueness trick?* → unique filtered index on active rows.

**Signals interviewers look for:** understanding of plan reuse, ability to diagnose from plan XML, practical fixes.

### 11. Check yourself
- [ ] Create a filtered index and show a literal query seeks while a parameterized one scans.
- [ ] Find `UnmatchedIndexes` in a plan.
- [ ] Explain why forced parameterization defeats filtered indexes.
- [ ] Design a "unique where not deleted" constraint.

<!-- related-deep-dives -->

### 🔗 Related deep dives
- [Same Syntax, Different Semantics (UNIQUE and NULL)](<Dialects_and_Portability/04_Same_Syntax_Different_Semantics.md>)
- [Parser, Binder and Error Messages (SET options and plan reuse)](<Dialects_and_Portability/05_Parser_Binder_and_Error_Messages.md>)
- [Series hub: T-SQL, ANSI SQL and Dialects](<Dialects_and_Portability/_index.md>)
<!-- explainer:end -->

## Implementations (tools)

- [SQL Server](<../_index.md>)

## Practice questions

- [Basic] [What is an index and how does it improve query performance?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42879_What_is_an_index_and_how_does_it_improve_query_performance.md>)
- [Deep] [What is denormalization and why might you deliberately denormalize a schema?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42925_What_is_denormalization_and_why_might_you_deliberately_denor.md>)
- [Deep] [How do you interpret and act on a high signal from sys.dm_db_index_usage_stats when deciding to drop an index?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42761_How_do_you_interpret_and_act_on_a_high_signal_from_sys_dm_db.md>)
- [Deep] [How does fill factor affect page splits and write versus read performance, and how would you tune it for a high-insert table?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42771_How_does_fill_factor_affect_page_splits_and_write_versus_rea.md>)
- [Deep] [What is the plan cache, how do you find the most expensive cached plans with sys.dm_exec_query_stats, and what does plan cache bloat look like?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42791_What_is_the_plan_cache_how_do_you_find_the_most_expensive_ca.md>)
- [Deep] [Why can passing an NVARCHAR parameter against a VARCHAR column defeat an index, and how do collation and data-type precedence drive it?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42836_Why_can_passing_an_NVARCHAR_parameter_against_a_VARCHAR_colu.md>)
- [Deep] [How do OPTION (RECOMPILE) OPTIMIZE FOR and the KEEPFIXED PLAN hint each address parameter sniffing and what are their trade-offs?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42954_How_do_OPTION_RECOMPILE_OPTIMIZE_FOR_and_the_KEEPFIXED_PLAN_.md>)
- [Expert] [How do you decide an appropriate MAXDOP and cost threshold for parallelism for a mixed OLTP and reporting workload?](<../../../PRACTICE/TOOLS/SQL_Server/T-SQL/Q-42786_How_do_you_decide_an_appropriate_MAXDOP_and_cost_threshold_f.md>)

## Related concepts

- [Clustered Indexes and Indexed Views](<../SQL_Server_Platform/Clustered_Indexes_and_Indexed_Views.md>) *(SQL Server Platform)*
- [Index Access Paths](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Index_Access_Paths.md>) *(DBMS Concepts)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Ultimate T-SQL for SQL Server Professionals › Preface
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 7: Dynamic SQL, Triggers, and Table-Valued Functions (TVFs) › Introduction
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Introduction
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Detecting Implicit Conversions
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Optimization › 10 Optimization
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Optimization › 10.5 #58 Working against the optimizer
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Indexes › 11.3 #69 Reorganizing indexes to fix page density
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Indexes › 11.8.2 Considerations for SORTINTEMPDB
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — BatCh exeCution H XE UTION › Batch Mode vs. Row Mode Performance
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Delete anD UpDate OperatiOns UP ATE OPERATI › CHAPTER 10
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Non-clustered Rowstore Indexes on Columnstore Tables › Compression for Non-clustered Rowstore Indexes
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Index Maintenance › Reorganize to Remove Delta Rowgroups
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 4 Analyzing Query Behavior Using Execution Plans
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 11 Key Lookups And Solutions
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 12 Dealing With Index Fragmentation
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 15 Reduce Query Resource Use
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Simple SELECT Queries › Thinking About Performance
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Discovering Windowing Functions › Thinking About Performance
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Splits
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Compression
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Index IO › Contiguous Reads
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — T-SQL Scripts

