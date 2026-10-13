---
id: "01_Databases_and_SQL/Query_Performance/Indexing/Covering_Indexes_and_Index_Strategy_for_Sharding"
level: 5
type: "deep-dive"
topic: "01_Databases_and_SQL/Query_Performance/Indexing"
tools: []
---

# Covering Indexes and Index Strategy for Sharding

**L5 Expert** · [↑ Indexing](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A covering index contains every column a query needs, so the engine answers entirely from the index and skips the expensive per-row trip back to the table — at the price of a bigger index and slower writes.

### 1. The big picture
A nonclustered (secondary) index stores the key columns plus a **row locator** (the clustering key in SQL Server's clustered tables, a heap RID/TID otherwise). When a query needs a column that isn't in the index, the engine must follow the locator back to the base table for every matching row: a **key lookup** (SQL Server, clustered table), **RID lookup** (heap) or **heap fetch** (PostgreSQL). Each lookup is a random page access; for hundreds or thousands of rows it dominates the cost, and past a threshold the optimizer abandons the index and scans the table instead.

A **covering index** removes the lookups by making the index itself contain all columns the query references (in `SELECT`, `WHERE`, `JOIN`, `ORDER BY`). This lesson also covers the strategic side at scale: how many covering indexes you can afford, how they interact with sharding/partitioning, how to prune them, and when a denormalized read table beats yet another wide index.

### 2. Core concepts and vocabulary
- **Key lookup / heap fetch** — the extra read of the base row to get columns missing from the index.
- **Covering index** — one that contains all columns needed by a given query; "covering" is relative to a query, not an absolute property.
- **Key column** — participates in ordering and can be seeked on.
- **Included (non-key) column** — stored only at the leaf level via `INCLUDE`; can be returned but not searched or sorted by.
- **Index-only scan** — PostgreSQL's name for answering from the index; additionally needs the visibility map (see the index-only scan lesson).
- **Write amplification** — each extra index multiplies the physical writes per logical write.
- **Index bloat** — dead/unused space in an index after many updates/deletes (PostgreSQL) or page splits (fragmentation).
- **Partial / filtered index** — indexes only rows satisfying a predicate (`WHERE is_deleted = false`), shrinking a covering index dramatically on soft-delete tables.

### 3. How it works, step by step
For the query

```sql
SELECT CustomerID, OrderDate, TotalAmount
FROM   Orders
WHERE  CustomerID = 101;
```
1. With only `IX(CustomerID)`: seek to customer 101 → for each of N matching entries, follow the clustered key to the base table → read `OrderDate`, `TotalAmount`. Cost ≈ index pages + N random lookups (+ nested-loop join operator joining index rows to lookup rows in the plan).
2. With the covering index `IX_Orders_Cover ON Orders (CustomerID) INCLUDE (OrderDate, TotalAmount)`: seek to customer 101 → read N adjacent leaf entries that already contain everything → done. No lookup, no nested loops.

In the plan you see one *Index Seek* instead of *Index Seek + Key Lookup + Nested Loops*. In the cited tuning examples, removing the lookup halved logical reads (4 → 2) and roughly halved duration even for a one-row query; for larger N the gap widens linearly.

Two ways to make an index covering:
- **Add the columns to the key** — works but widens every level of the tree, changes ordering semantics, and for unique indexes changes what is unique.
- **`INCLUDE` the columns** — only leaf pages get wider; internal pages and the uniqueness definition are unchanged. Preferred for pure "carry-along" columns.

### 4. Worked example
```sql
-- Setup (SQL Server syntax; PostgreSQL 11+ has the same INCLUDE)
CREATE INDEX IX_Contacts_Name ON dbo.Contacts (FirstName, LastName);

SET STATISTICS IO ON;
SELECT ContactID, FirstName, LastName, IsActive
FROM   dbo.Contacts
WHERE  FirstName = 'Catherine' AND LastName = 'Cox';
-- Plan: Index Seek + Key Lookup (IsActive missing) + Nested Loops
-- logical reads: 5 for ONE row

CREATE INDEX IX_Contacts_Name_Cover
ON dbo.Contacts (FirstName, LastName) INCLUDE (IsActive)
WITH (DROP_EXISTING = OFF);

-- Re-run: Index Seek only, logical reads: 2
```
Read it like this: `ContactID` was already available because it is the clustered key, which every nonclustered index carries. `IsActive` was the one missing column → one lookup per row. Adding it as an included column eliminated the lookup. The same shape applies in PostgreSQL: `CREATE INDEX … INCLUDE (is_active)` followed by `EXPLAIN (ANALYZE, BUFFERS)` shows `Index Only Scan … Heap Fetches: 0` after a `VACUUM`.

### 5. Under the hood
- **Leaf-page layout.** An included column is stored once per leaf entry, not in the intermediate pages. Fanout (entries per internal page) is therefore unchanged, so tree depth stays the same, but leaf pages hold fewer entries → more pages to scan for range queries.
- **What every secondary index already contains.** In SQL Server the clustered key is appended to each nonclustered index; in InnoDB the primary key is. So the primary key is "free" to select. PostgreSQL stores a TID instead, so the primary key column is *not* automatically covered.
- **Large values.** `varchar(max)`/`TEXT` columns in `INCLUDE` can bloat the index and may be impossible (key size limits ≈ 900 bytes clustered / 1700 bytes nonclustered in SQL Server; ≈ 1/3 page in PostgreSQL B-tree). Fix: leave the wide column out and accept one lookup, or store a hash/prefix.
- **Why the optimizer sometimes ignores a covering index.** If statistics say the predicate matches a large fraction of rows, a scan of the narrower covering index (or the table) may be chosen; or the query selects a column that isn't included (the classic `SELECT *` regression).
- **Maintenance.** Updating an included column changes the index entry (an update becomes delete+insert in the index if it moves, or an in-place update). In PostgreSQL, updating any indexed (including `INCLUDE`d) column disables **HOT updates** for that row, so wide covering indexes on frequently-updated columns hurt twice.
- **Sharding/partitioning.** A *local* index covers only its shard/partition and can't enforce global uniqueness or serve cross-shard queries without scatter-gather; a *global* index spans shards and costs distributed writes. Choose the shard key as the leading column of the indexes that serve most queries so that a query touches one shard and uses a local covering index.

### 6. Performance and design implications
- **Reads:** fewer logical reads, no lookups, sequential leaf scans; huge win for queries returning many rows or for joins in the hot path.
- **Writes:** more bytes per insert/update, bigger log, more page splits. Measure by comparing write latency/throughput with and without the index on a production-like load.
- **Storage:** every covering index is a partial copy of the table. On a multi-petabyte fleet a 30 % overrun from covering indexes needs a **prioritised trimming framework**: rank indexes by (bytes × 0 or low usage) using usage DMVs / `pg_stat_user_indexes`, look for prefix-redundant and near-duplicate indexes, check whether the only consumer is a rare report, then stage the removal (disable → observe → drop) with a rollback script.
- **Cache interplay.** A result cache makes a covering index redundant only for *repeated identical* queries within the staleness window; the index still serves cache misses, cold starts and parameter variations.
- **Alternatives when many access patterns hit one entity:** (1) a few well-chosen covering indexes on the canonical table, (2) a denormalized read table/materialized view per pattern (maintained by trigger, CDC or batch), (3) a separate store. Start with (1); move to (2) when index count or write amplification becomes the bottleneck and staleness is acceptable.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE INDEX … INCLUDE (cols)` | Adds non-key leaf columns | Cover a query without widening the key |
| `… WHERE predicate` (partial/filtered) | Index subset of rows | Soft deletes, hot "open" rows |
| `SET STATISTICS IO ON`, `EXPLAIN (ANALYZE, BUFFERS)` | Compare reads | Prove the lookup is gone |
| `CREATE INDEX CONCURRENTLY` (PG), `WITH (ONLINE = ON)` (SQL Server Enterprise) | Build without blocking writes | Adding a large covering index in production |
| `REINDEX CONCURRENTLY`, `pg_repack` | Rebuild bloated indexes | Bloat from heavy updates |
| `sys.dm_db_index_usage_stats` / `pg_stat_user_indexes.idx_scan` | Usage counters (reset on restart; check retention window) | Finding never-scanned covering indexes |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Hot, frequent query returning few columns | Query is `SELECT *` or columns change often |
| Read-heavy table, modest write rate | Write-heavy table where the index doubles write latency |
| Included columns are narrow and stable | Included column is a wide text/JSON blob |
| Soft-delete table → *partial* covering index on live rows | Only 5 % of rows are queried but index covers 100 % |
| Need exact current data | Staleness acceptable and many access patterns → read model instead |

### 9. Common mistakes
- **`SELECT *` in code that relied on a covering index** → silent return of key lookups → select only needed columns; catch in review and with plan regression tests.
- **Dropping an `INCLUDE` column in a migration** → query 10× slower with no error → keep index definitions in version control and assert plan shape in a performance test.
- **Putting included columns in the key** → bigger tree, changed uniqueness, unnecessary sort order constraints → use `INCLUDE`.
- **Covering everything "just in case"** → write cost and storage balloon → cover only measured hot queries.
- **Trusting `idx_scan = 0` blindly** → dropping an index used only at month-end or by a failover replica → look at a full business cycle and all replicas.
- **Forgetting concurrency when building** → blocking DDL on a live table → use concurrent/online build.

### 10. 🎯 Interview kit
**30-second answer:** A covering index holds all columns the query needs, so the engine returns results from the index alone and avoids a lookup into the table for every row. You add the non-searched columns with `INCLUDE`. The cost is a larger index and slower writes, so you cover only hot, narrow, stable queries.

**Structure a longer answer:** secondary index + row locator → lookup cost grows with matching rows → covering removes it (`INCLUDE` vs key columns) → evidence (plan, reads) → write/storage cost, `SELECT *` fragility → alternatives (read model, cache) → real scenario.

**Likely follow-ups:**
- *Covering index vs narrow index + buffer cache?* → Cache hides physical I/O but not CPU/latch cost of N random lookups; covering gives sequential leaf reads. Narrow wins if lookups are few and writes are heavy.
- *`INCLUDE` vs extra key columns?* → `INCLUDE` doesn't affect ordering or uniqueness and keeps internal pages small; key columns are required if you must seek or sort on them.
- *How do you decide partial vs full for soft-delete tables?* → If queries always filter live rows, partial: smaller, cheaper writes; full only if some paths read deleted rows.
- *How would you trim a 30 % storage overrun safely?* → Usage data → rank by bytes and benefit → disable/observe → drop → monitor.
- *Covering index vs denormalized read table?* → Index: strongly consistent, simple, write-amplifying; read table: flexible shape, asynchronous, more operational surface.

**Signals interviewers look for:** you explain the lookup mechanism, quantify with reads, know `INCLUDE`, discuss write/storage cost and a safe rollout/removal process.

### 11. Check yourself
- [ ] Draw the plan with and without a covering index and identify the operators removed.
- [ ] Explain which columns a nonclustered index "already covers" in SQL Server vs InnoDB vs PostgreSQL.
- [ ] Write a covering index for a 3-column select with one equality and one range filter.
- [ ] Describe a safe procedure to remove an apparently unused covering index in production.
- [ ] Explain why updating an included column can hurt PostgreSQL HOT updates.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [Your report query does an index scan but still reads the heap for every row — how would you turn it into an index-only scan in PostgreSQL?](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-00516_Your_report_query_does_an_index_scan_but_still_reads_the_hea.md>)
- [Deep] [Walk me through how a covering index eliminates a table lookup and when you'd choose one over a plain secondary index.](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-00513_Walk_me_through_how_a_covering_index_eliminates_a_table_look.md>)
- [Deep] [Explain the trade-off between a wide covering index and a narrow index plus a well-tuned buffer cache for heap fetches.](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-00571_Explain_the_trade-off_between_a_wide_covering_index_and_a_na.md>)
- [Deep] [How do you handle covering indexes when a column's data distribution shifts over time and old assumptions no longer hold?](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-00609_How_do_you_handle_covering_indexes_when_a_column_s_data_dist.md>)
- [Deep] [How do you decide, as an architect, between a partial covering index and a full one for a soft-delete-heavy table, and how do you keep the choice consistent across the estate?](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-00654_How_do_you_decide_as_an_architect_between_a_partial_covering.md>)
- [Deep] [How do you decide whether a hot query deserves a covering index or a dedicated denormalized read table, factoring in team ownership, consistency needs, and long-term maintenance?](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-00687_How_do_you_decide_whether_a_hot_query_deserves_a_covering_in.md>)
- [Deep] [A principal probe: how do you reason about the interplay between covering indexes and query-result caching layers — when does the index become redundant with the cache, and vice versa?](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-00712_A_principal_probe_how_do_you_reason_about_the_interplay_betw.md>)
- [Expert] [You're asked to reduce a 30% storage overrun caused by covering indexes on a multi-petabyte fleet — walk through the prioritization framework you'd use to trim safely.](<../../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-00640_You_re_asked_to_reduce_a_30_storage_overrun_caused_by_coveri.md>)

## Related concepts

- [Index Strategy and Governance at Scale](<../Index_Tuning_in_Practice/Index_Strategy_and_Governance_at_Scale.md>) *(Indexing)*
- [Index Tuning in Practice](<../Index_Tuning_in_Practice/_index.md>) *(Indexing)*
- [Query Rewrites and Plan Stability](<../../Execution_Plans/Query_Rewrites_and_Plan_Stability.md>) *(Execution Plans)*
- [Leftmost Prefix Rule](<../../../SQL_Language/SQL_Fundamentals/Leftmost_Prefix_Rule.md>) *(SQL Fundamentals)*
- [Index Access Paths](<../../../Database_Internals/DBMS_Concepts/Index_Access_Paths.md>) *(DBMS Concepts)*
- [Composite Indexes and Column Order](<../Composite_Indexes_and_Column_Order.md>) *(Indexing)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Covering Indexes
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Best Practices for Index Design
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 10 Index Behaviors
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 11 Key Lookups And Solutions
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 14 Query Design Analysis
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — sys.fulltextindexcolumns › Chapter 2
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Balance Index Count › Specify Fill Factors
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Covering Indexes
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Included Columns
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — NoN-Clustered ColumNstore INdexes oN rowstore tables - LUSTERED COLUM STORE! DEXESO OWSTORE
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Index Performance › Memory-Optimized Columnstore Indexes
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Ordered Columnstore Indexes

