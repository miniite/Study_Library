---
id: "01_Databases_and_SQL/Query_Performance/Indexing/Index-only_Scans"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Query_Performance/Indexing"
tools: []
---

# Index-only Scans

**L4 Advanced** · [↑ Indexing](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** An index-only scan answers a query using just the index pages, skipping the table entirely — and in PostgreSQL it only stays truly "index-only" if the visibility map says the pages are all-visible, which is why VACUUM matters.

### 1. The big picture
Most indexes help by *locating* rows; the expensive part comes afterwards, when the engine hops to the table to fetch the rest of each row. An **index-only scan** (PostgreSQL's term; SQL Server shows a plain Index Seek/Scan with no lookup, MySQL shows `Using index`) is the plan shape where the index alone contains every column the query touches.

In engines with MVCC that keep old row versions in the table (PostgreSQL, Oracle in its own way), there's a catch: an index entry doesn't say whether the row version is *visible to your transaction*. The engine usually has to check the table row's transaction information. The **visibility map** is the shortcut that lets it skip that check for pages known to contain only rows visible to everyone. So "index-only" in the plan can still do heap fetches at runtime if the map is stale. This lesson explains when the optimizer can choose such a scan, how to verify it really avoids the table, and the related planner/statistics knobs (`ANALYZE`, `VACUUM`, `random_page_cost`, index usage statistics) that decide whether an index is used at all.

### 2. Core concepts and vocabulary
- **Index-only scan** — a scan satisfied from index entries without reading the table row.
- **Covering index** — an index containing all columns required by a query (see covering index lesson); a prerequisite.
- **Heap** — PostgreSQL's table storage; rows in no particular order.
- **MVCC** — multi-version concurrency control: updates create new row versions; old versions remain until vacuumed.
- **Dead tuple** — a row version no transaction can see anymore; space not yet reclaimed.
- **Visibility map (VM)** — per-table bitmap, one bit per 8 KB heap page, set when *all* tuples on the page are visible to all transactions.
- **Heap fetch** — a visit to the table for a row the index scan couldn't confirm as visible.
- **`VACUUM`** — reclaims dead tuples, sets VM bits and updates free-space info. **`ANALYZE`** — collects column statistics for the planner.
- **`seq_page_cost` / `random_page_cost`** — planner's relative cost of sequential vs random page reads (default 1.0 and 4.0; ~1.1 on SSD).
- **Index condition pushdown (ICP, MySQL)** — evaluating parts of the `WHERE` on index columns inside the storage engine before fetching the row.

### 3. How it works, step by step
PostgreSQL index-only scan:
1. The planner finds a B-tree (or GiST/SP-GiST with the right opclass) whose columns include every referenced column (key columns and `INCLUDE`d columns), and estimates the fraction of heap pages that are all-visible from `pg_class.relallvisible`.
2. At execution it walks the index for matching entries.
3. For each entry's TID, it checks the **visibility map bit for that heap page**.
4. If the bit is set → return values from the index entry directly (no heap I/O).
5. If not set → **heap fetch**: read the heap page, check tuple visibility for this snapshot, then return.
6. `EXPLAIN (ANALYZE)` reports `Heap Fetches: N`. A good index-only scan has N near 0.

Other engines:
- **SQL Server:** no visibility map issue (versions live in tempdb/PVR, locks or RCSI); a covering nonclustered index shows only a seek/scan, and absence of *Key Lookup* is the signal.
- **MySQL InnoDB:** a secondary index implicitly contains the primary key; `EXPLAIN` `Extra: Using index` means covering. `Using index condition` means ICP only — the row is still fetched for non-indexed columns.

### 4. Worked example
```sql
-- PostgreSQL
CREATE TABLE orders (
  id bigserial PRIMARY KEY,
  customer_id int NOT NULL,
  created_at timestamptz NOT NULL,
  total numeric(12,2) NOT NULL,
  note text
);
CREATE INDEX ix_orders_cust ON orders (customer_id) INCLUDE (created_at, total);

-- after bulk load, before vacuum
EXPLAIN (ANALYZE, BUFFERS)
SELECT created_at, total FROM orders WHERE customer_id = 42;
--  Index Only Scan using ix_orders_cust on orders
--    Heap Fetches: 1873          <- plan says index-only, runtime still hits the heap
--    Buffers: shared hit=...

VACUUM (ANALYZE) orders;          -- sets visibility-map bits

EXPLAIN (ANALYZE, BUFFERS)
SELECT created_at, total FROM orders WHERE customer_id = 42;
--  Index Only Scan ... Heap Fetches: 0
```
What to read: `Heap Fetches` (plan vs execution divergence), `Buffers` (page counts), and compare with `SELECT *`, which silently reverts to an ordinary `Index Scan` because `note` isn't in the index. To find indexes that exist but contribute nothing:
```sql
SELECT relname, indexrelname, idx_scan, idx_tup_read, idx_tup_fetch
FROM pg_stat_user_indexes ORDER BY idx_scan;   -- idx_scan = 0 since stats reset => candidate for removal
```

### 5. Under the hood
- **Visibility map maintenance.** A bit is set by `VACUUM` (or autovacuum) when every tuple on the page is old enough to be visible to all snapshots; it is **cleared by any insert/update/delete on that page**. So write-heavy tables have low all-visible fractions between vacuums and index-only scans degrade — the "plan vs execution" gap. Fixes: tune autovacuum (`autovacuum_vacuum_scale_factor`, `autovacuum_vacuum_insert_scale_factor` for insert-only tables in PG13+), run manual `VACUUM`, reduce long-running transactions that hold back the xmin horizon.
- **Bloat and dead tuples.** MVCC leaves dead tuples; without vacuum, heap and indexes bloat, scans read more pages, and visibility bits can't be set. `VACUUM` reclaims space for reuse but doesn't shrink the file (that needs `VACUUM FULL`, `pg_repack`, `REINDEX`).
- **HOT updates.** If an update changes no indexed column and the new version fits on the same page, it's a heap-only tuple: no new index entries. Adding more (or covering) columns to indexes reduces HOT eligibility.
- **Planner cost model.** Cost of index scan ≈ index pages + `random_page_cost` × heap pages touched; seq scan ≈ `seq_page_cost` × total pages. On HDD-backed or cold storage a sequential scan can legitimately win; on SSD, lowering `random_page_cost` (1.1–1.5) makes index plans more attractive. To influence one query rather than globally: `SET LOCAL enable_indexscan = off` or `SET LOCAL random_page_cost = …` inside a transaction, or per-tablespace costs.
- **Bitmap heap scan.** For mid-selectivity predicates PG may collect TIDs from the index into a bitmap, sort them by heap page, then read pages in physical order — turning random I/O into near-sequential and allowing multiple indexes to be combined (BitmapAnd/Or).
- **MySQL ICP.** The storage engine evaluates indexed-column predicates before fetching the row, cutting row reads for composite indexes — but it does not make the query covering.
- **Statistics.** `ANALYZE` stores per-column most-common values, histograms, null fraction (`nullfrac`), n_distinct; stale values mislead the planner (e.g., a stale `nullfrac` makes a predicate look unselective and a new index ignored). `pg_stat_statements` aggregates per-query time, calls, `shared_blks_hit/read`, to find worst queries; `shared_blks_read` high vs `hit` low means data isn't cached.

### 6. Performance and design implications
- Index-only is worth designing for on **hot, narrow, read-mostly queries**: dashboards, lookups returning a handful of columns, counts of rows with index-covered predicates.
- Keep covering indexes narrow; include only columns used in the output.
- Append-only/insert-mostly tables give high all-visible ratios after vacuum; hot-update tables don't.
- Observe: `Heap Fetches` in `EXPLAIN ANALYZE`; `pg_stat_user_tables.n_dead_tup`, `last_autovacuum`; `pg_class.relallvisible / relpages`.
- Don't conflate the three questions: *is the index chosen?* (statistics/costs), *is it covering?* (columns), *is the heap really untouched?* (visibility map).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `EXPLAIN (ANALYZE, BUFFERS)` | Plan with actual rows, heap fetches, buffer hits | Confirm index-only really avoided the heap |
| `VACUUM (ANALYZE) t` | Reclaim dead tuples, set VM bits, refresh stats | After bulk loads / before benchmarks |
| `autovacuum_vacuum_scale_factor`, `autovacuum_vacuum_insert_scale_factor` | When autovacuum triggers | Large tables where defaults are too lazy |
| `CREATE INDEX … INCLUDE (…)` | Add payload columns | Make an index covering |
| `random_page_cost`, `seq_page_cost`, `effective_cache_size` | Planner cost model | Tune for SSD/cache; use `SET LOCAL` for one query |
| `pg_stat_user_indexes`, `pg_stat_statements` | Index and query statistics | Find unused indexes and worst queries |
| `REINDEX CONCURRENTLY`, `pg_repack` | Rebuild bloated indexes/tables online | Index bloat (note `pg_repack` limits with partitioned tables) |
| MySQL `EXPLAIN` → `Using index` / `Using index condition` | Covering vs ICP | Distinguish true covering from pushdown |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Read-mostly table, vacuumed regularly | Hot-update table with vacuum lag — VM mostly clear |
| Small number of output columns | `SELECT *` or wide payloads |
| Predicate selective enough to benefit | Predicate matches a large share → seq scan is better anyway |
| SSD with tuned `random_page_cost` | HDD with default cost; forcing index use may slow things |

### 9. Common mistakes
- **Believing "Index Only Scan" in the plan means zero heap reads** → high `Heap Fetches` at runtime → vacuum, check dead tuples, read ANALYZE output.
- **Mixing up ANALYZE and VACUUM** → ANALYZE updates planner statistics; VACUUM removes dead tuples and updates the visibility map and free space. Autovacuum does both but on its own schedule.
- **Dropping an index because `idx_scan = 0` since a recent stats reset** → check how long counters have accumulated and whether replicas serve reads.
- **Overusing `INCLUDE`** → big indexes, no HOT updates → include only what's needed.
- **Disabling index scans globally for an HDD problem** → hurts every query → scope with `SET LOCAL` or fix cost settings per tablespace.
- **Forgetting ICP's boundary** (MySQL) → `Using index condition` still fetches rows for unindexed columns.

### 10. 🎯 Interview kit
**30-second answer:** An index-only scan returns results straight from the index when it contains every column the query needs. In PostgreSQL it also needs the visibility map: pages marked all-visible need no heap check, otherwise the engine does heap fetches. So you design a covering index and keep the table vacuumed.

**Structure a longer answer:** covering condition → why MVCC needs visibility → visibility map set by vacuum, cleared by writes → verify via `Heap Fetches` → operational tuning (autovacuum, bloat) → contrast with SQL Server/MySQL.

**Likely follow-ups:**
- *Plan says index-only but heavy heap fetches — why?* → VM bits not set: recent writes, vacuum behind, long transactions pinning xmin.
- *ANALYZE vs VACUUM?* → Stats for planner vs dead-tuple cleanup/visibility map.
- *Index scan with random I/O slower than seq scan on HDD?* → Adjust `random_page_cost` (per session/tablespace) or `SET LOCAL enable_indexscan = off` for that query.
- *What is ICP?* → Evaluate indexed-column predicates in the engine before row fetch; reduces fetches, not covering.
- *What is a bitmap heap scan?* → Collects TIDs, sorts by page, reads heap in order; used for mid-selectivity or multi-index combos.

**Signals interviewers look for:** MVCC awareness, ability to read `EXPLAIN (ANALYZE, BUFFERS)`, operations mindset (vacuum, bloat, monitoring).

### 11. Check yourself
- [ ] Explain why PostgreSQL needs the visibility map for index-only scans.
- [ ] Write an `EXPLAIN` command that shows heap fetches and interpret the number.
- [ ] Describe two causes of high heap fetches on an apparently covering index.
- [ ] Explain the difference between `Using index` and `Using index condition` in MySQL.
- [ ] State what `pg_stat_user_indexes.idx_scan = 0` does and doesn't prove.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [What is pg_stat_statements and how do you use it to find your worst queries?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19651_What_is_pg_stat_statements_and_how_do_you_use_it_to_find_you.md>)
- [Deep] [What is an index-only scan and what conditions must be met for one?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19510_What_is_an_index-only_scan_and_what_conditions_must_be_met_f.md>)
- [Deep] [How do you use idx_scan, idx_tup_read, and idx_tup_fetch from index statistics to judge whether an index is pulling its weight?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20104_How_do_you_use_idx_scan_idx_tup_read_and_idx_tup_fetch_from_.md>)
- [Deep] [How would you decide the retention and indexing strategy for pg_stat_statements or the slow query log itself, so your observability doesn't become a performance problem?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19765_How_would_you_decide_the_retention_and_indexing_strategy_for.md>)
- [Deep] [An index scan is chosen but random I/O on an HDD-backed volume makes it slower than a seq scan. How do you get the planner to prefer the seq scan without a global cost change?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19834_An_index_scan_is_chosen_but_random_I_O_on_an_HDD-backed_volu.md>)
- [Expert] [What is the visibility map in PostgreSQL and why is it required for an index-only scan?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19583_What_is_the_visibility_map_in_PostgreSQL_and_why_is_it_requi.md>)
- [Expert] [What is index condition pushdown (ICP) in MySQL and how does it reduce heap fetches?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19594_What_is_index_condition_pushdown_ICP_in_MySQL_and_how_does_i.md>)
- [Expert] [A query's plan shows a bitmap heap scan with a high 'lossy' block count. What does lossy mean here and how does work_mem relate to it?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20117_A_query_s_plan_shows_a_bitmap_heap_scan_with_a_high_lossy_bl.md>)

## Related concepts

- [Indexing Multi-join Queries](<../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Indexing_Multi-join_Queries.md>) *(PostgreSQL Platform)*
- [Index Access Paths](<../../Database_Internals/DBMS_Concepts/Index_Access_Paths.md>) *(DBMS Concepts)*
- [Materialized Views vs Incremental Aggregates](<../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Materialized_Views_vs_Incremental_Aggregates.md>) *(PostgreSQL Platform)*

