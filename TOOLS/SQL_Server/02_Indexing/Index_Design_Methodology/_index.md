---
id: "TOOLS/SQL_Server/02_Indexing/Index_Design_Methodology"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Index Design Methodology

**L4 Advanced** · [↑ Indexing](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Good indexing is a repeatable cycle — **monitor** how the workload uses data, **analyze** which indexes to create, change or drop (and what each change will affect), **implement** them safely through source control and review — then repeat, because workloads and data keep changing.

### 1. The big picture
The previous lessons explained *what* indexes are. This one is about *how to decide* which ones a database should have. Two failure modes are common: building indexes by intuition ("index every foreign key and every searchable column"), and applying tool suggestions blindly (missing-index DMVs, Database Engine Tuning Advisor). Both produce too many overlapping indexes, slow writes and still-slow queries.

A methodology replaces guessing with evidence. The one used in the indexing books is a loop of three phases — **Monitor → Analyze → Implement → (repeat)** — combined with a set of query-level design rules. The DMVs that feed it are covered in detail in the sibling *Index Analysis DMVs* lesson; this lesson explains the process and the design rules.

### 2. Core concepts and vocabulary
- **Workload** — the set of queries the system actually runs, weighted by frequency and importance (including rare but critical jobs like month-end).
- **Monitor phase** — collecting evidence: index usage, missing-index signals, waits, performance counters, expensive queries, Query Store.
- **Analyze phase** — turning evidence into a change list: create, modify (consolidate, add includes), drop; plus impact analysis.
- **Implement phase** — communicating, scripting into source control, peer review, execution and rollback plan.
- **Impact analysis** — which queries/plans use each index you plan to change, and their before/after measurements.
- **Duplicate index** — same key columns in the same order (same includes) as another index.
- **Overlapping index** — an index whose key is a left prefix of another's, e.g. (A) and (A, B); the wider one can usually serve both.
- **Unindexed foreign key** — an FK column with no index starting with it; causes scans on joins and on parent deletes/updates.
- **SARGable predicate** — written so an index seek can use it.
- **DTA (Database Engine Tuning Advisor)** — a tool that analyzes a workload sample and recommends indexes, indexed views and partitioning.
- **Index intersection / index join** — combining two NC indexes on one table to satisfy a query.

### 3. How it works, step by step
**Phase 1 — Monitor**
1. Snapshot index usage regularly (`sys.dm_db_index_usage_stats`, `sys.dm_db_index_operational_stats`) into tables — they reset on restart, so history must be persisted.
2. Capture missing-index signals (`sys.dm_db_missing_index_*`, missing-index hints in plans).
3. Track wait statistics, performance counters (Forwarded Records/sec, Page Splits/sec, Full Scans/sec, Index Searches/sec, Lock Waits) and the most expensive queries (Query Store, `sys.dm_exec_query_stats`).
4. Collect over a full business cycle — including month-end, quarter-end, batch nights.

**Phase 2 — Analyze**
1. *Server state review:* which databases consume the buffer pool, what waits dominate, which counters are abnormal. Decide where indexing effort pays off.
2. *Schema discovery:* find heaps, duplicate indexes, overlapping indexes, unindexed foreign keys.
3. *Usage review:* indexes with many updates but no seeks/scans/lookups (candidates to drop), indexes with heavy lookups (candidates to cover), missing-index groups with high impact.
4. *Plan review:* for top queries, inspect plans for scans, key lookups, sorts, spills and implicit conversions.
5. *Design the change* for each query family using the rules below; consolidate rather than add.
6. *Impact analysis:* for each index to drop or change, find the plans that use it (search plan cache/Query Store for the index name) and measure before/after on a test copy.

**Phase 3 — Implement**
1. *Communicate:* status report of what will change and why, with expected effects.
2. *Script and store:* deployment and rollback scripts in source control (index changes are schema changes).
3. *Peer review:* walk a colleague through each change and the impact analysis — catches missed business processes.
4. *Execute:* in a window or with `ONLINE = ON`, monitoring blocking and log growth.
5. *Verify* with the same metrics used in the Monitor phase — then the cycle restarts.

**Query-level design rules**
1. Start from the filtering clauses: `WHERE`, `JOIN`, `HAVING` — they benefit most.
2. Key column order: **equality predicates first, then the range or sort column**. Among equality columns, more selective first is a reasonable default, but the stronger rule is "put the columns that most queries share as their leading predicate first", so one index serves many queries.
3. Add output-only columns as `INCLUDE`, not to the key.
4. Keep keys narrow; prefer integer and date types over long strings.
5. Make predicates SARGable: `OrderDate >= '2025-01-01' AND OrderDate < '2026-01-01'`, not `YEAR(OrderDate) = 2025`.
6. Choose structure by query type: point lookups and small ranges → rowstore B-tree; large aggregations → columnstore; repeated identical aggregation → indexed view.
7. Prefer modifying an existing index (adding a column, an include) to adding a near-duplicate — but check that reordering doesn't hurt other queries.
8. Index foreign keys that are joined on or whose parent rows get deleted.
9. Always weigh the write cost: each new index slows inserts/deletes and updates of its columns.

### 4. Worked example
```sql
-- Query family from Query Store: top CPU consumer
SELECT poh.PurchaseOrderID, poh.OrderDate, poh.TotalDue
FROM Purchasing.PurchaseOrderHeader AS poh
WHERE poh.EmployeeID = @emp AND poh.OrderDate >= @from
ORDER BY poh.OrderDate;

-- Existing: IX_POH_EmployeeID (EmployeeID), IX_POH_VendorID (VendorID)
-- Plan: seek on EmployeeID → key lookup for OrderDate/TotalDue → sort

-- Analyze: is IX_POH_EmployeeID used elsewhere? Search cached plans for the name
SELECT TOP (20) qs.execution_count, SUBSTRING(st.text, 1, 200) AS stmt
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_query_plan(qs.plan_handle) qp
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) st
WHERE CAST(qp.query_plan AS nvarchar(max)) LIKE '%IX_POH_EmployeeID%';

-- Design: extend instead of adding a new index (equality, then range/sort; output in INCLUDE)
CREATE NONCLUSTERED INDEX IX_POH_EmployeeID
ON Purchasing.PurchaseOrderHeader (EmployeeID, OrderDate)
INCLUDE (TotalDue)
WITH (DROP_EXISTING = ON, ONLINE = ON);
```
**What to verify:** the plan becomes a single Index Seek with no Key Lookup and no Sort (rows already arrive in OrderDate order within the employee). Reads drop from tens to 2–3. The queries that used the old index still work, because `EmployeeID` is still the leading column — the old index's capability is a subset of the new one. This "extend, don't add" move is the core of consolidation. Record the before/after reads and durations in the change ticket as the impact analysis.

### 5. Under the hood
- **Why index intersection/joins exist.** When no single index covers a query, the optimizer can seek two NC indexes and join their row locators (Hash Match or Merge Join). It saves building a wide index, but a composite or covering index is usually faster: in a book example, combining two single-column indexes into one compound key dropped a query from 10 reads to 2.
- **Why tool recommendations mislead.** Missing-index DMVs and DTA evaluate one query (or one sample) at a time: they don't merge suggestions, ignore column order subtleties (the DMV lists equality columns then inequality columns in table-column order), ignore write cost, and only know the workload you gave them. A real-world failure from the books: someone ran DTA against the plan cache a few hours after a restart — the cache was nearly empty, the tool suggested dropping most indexes as unused, they applied it, and the instance lost ~90% of its indexes.
- **Usage counters are volatile.** `sys.dm_db_index_usage_stats` resets on restart, database offline/detach, and (for an index) on rebuild in some versions; persisting snapshots is mandatory for drop decisions.

### 6. Performance and design implications
- The aim is the *smallest set* of indexes giving acceptable performance for the important queries — not maximum read speed for every query.
- A typical OLTP table ends up with a clustered index plus a handful of NC indexes; more is a signal to look for duplicates and overlap.
- Write-heavy tables: weigh `user_updates` against `user_seeks + user_scans + user_lookups`; an index updated a million times and read ten times is a cost.
- Re-run the cycle after releases, data growth or a change in usage patterns; indexes that were right a year ago may not be today.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sys.dm_db_index_usage_stats` | Seeks/scans/lookups/updates per index since restart | Usage review; drop candidates |
| `sys.dm_db_missing_index_details / _groups / _group_stats` | Optimizer's missing-index suggestions with impact | Starting point for create candidates |
| Query Store (`sys.query_store_*`) | Persisted query performance and plans | Finding top queries and impact analysis |
| `sys.dm_exec_query_stats` + `sys.dm_exec_query_plan` | Cached query stats and plans | Who uses an index |
| `CREATE INDEX … WITH (DROP_EXISTING = ON, ONLINE = ON)` | Modify an index in one step | Consolidation |
| `ALTER INDEX … DISABLE` | Keeps definition, removes data | Reversible "soft drop" test |
| Database Engine Tuning Advisor | Workload-based recommendations | Idea generation, never blind application |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Extending an existing index (keys/includes) | The reorder would break seeks for other queries |
| Adding a separate NC index | It duplicates or overlaps an existing one |
| Accepting index intersection with narrow indexes | A frequent query needs consistent low latency — use a composite index |
| Indexed view for repeated aggregations | Base tables are write-heavy (every change updates the view) |
| Dropping unused indexes after a full cycle of evidence | Evidence covers only days, or the index enforces uniqueness/supports FKs |

### 9. Common mistakes
- Applying every DTA/missing-index suggestion → duplicates, overlap, write slowdown → consolidate and review.
- Using usage stats right after a restart → "unused" critical indexes dropped → persist snapshots across a business cycle.
- Designing indexes before knowing the queries (beyond PK/unique keys) → wrong columns, wrong order → index from the workload.
- Ignoring non-SARGable predicates → indexes exist but are scanned → rewrite predicates first.
- No rollback script, no source control → can't restore a dropped index definition → script every change.
- Forgetting critical rare jobs (month-end, audits) → tuned for daily traffic, month-end overruns → include them in the workload.

### 10. 🎯 Interview kit
**30-second answer:** "I treat indexing as a cycle: monitor usage, missing-index data, waits and top queries over a full business cycle; analyze — find duplicates, overlaps, unused indexes, lookups and unindexed FKs, and design query-shaped indexes with equality columns first, range columns next and outputs in INCLUDE; then implement through source control with impact analysis, peer review and rollback scripts, and verify. Tool suggestions are input, not decisions."

**Structure a longer answer:** why a method (avoid guessing/over-indexing) → three phases → design rules → consolidation example → validation and repetition.

**Likely follow-ups:**
- *How do you find unused indexes safely?* → persisted usage snapshots across a cycle, check uniqueness/FK roles, disable before drop.
- *Why not trust the missing-index DMV?* → per-query, no consolidation, order not optimal, no write cost.
- *Key column order?* → equality first, then range/sort; shared leading columns help reuse.
- *How do you know which queries use an index?* → plan cache / Query Store XML search for the index name.

**Signals interviewers look for:** evidence-based approach, consolidation mindset, awareness of write cost and volatile DMVs, change management discipline.

### 11. Check yourself
- [ ] Describe each phase of the cycle and one concrete output of each.
- [ ] Turn three overlapping indexes on one table into one or two.
- [ ] Write a SARGable version of three non-SARGable predicates.
- [ ] Produce an impact analysis for dropping an index: users, before/after reads, rollback script.
- [ ] Explain why a DTA run after a restart can be dangerous.
<!-- explainer:end -->

## References

- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — load customer names › Chapter 23 A Query Tuning Methodology
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Indexes › 11.10 #76 Relying too heavily on Database Engine Tuning Advisor
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Execution › Repeat
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 16: Best Practices for T-SQL Development and Database Design › Improper Index Usage
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Impact Analysis
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Unindexed Foreign Keys › The Indexing Method
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Peer Review with Impact Analysis
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Negative Impacts of Indexes on Performance
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — � D › � I, J, K
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 5: Views, Functions, and Stored Procedures › Indexed Views

