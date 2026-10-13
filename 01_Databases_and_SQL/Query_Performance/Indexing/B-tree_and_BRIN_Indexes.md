---
id: "01_Databases_and_SQL/Query_Performance/Indexing/B-tree_and_BRIN_Indexes"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Query_Performance/Indexing"
tools: []
---

# B-tree and BRIN Indexes

**L4 Advanced** · [↑ Indexing](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A B-tree index keeps keys sorted in a shallow, page-based tree so any key or range is found in a few page reads; a BRIN index instead stores just a min/max summary per block range, making it tiny and ideal for huge tables whose values correlate with physical order.

### 1. The big picture
Disks and buffer pools move data in fixed-size pages (8 KB in SQL Server and PostgreSQL, 16 KB in InnoDB). A good on-disk index minimises the number of page reads needed to find data. The **B-tree** (in practice a **B+ tree**, where all keys live in leaf pages and leaves are linked) is the dominant answer: it is balanced, so every lookup costs the same few page reads; it keeps keys sorted, so equality, range scans and `ORDER BY` are all efficient; and it handles inserts and deletes by splitting and merging pages.

Its weakness is size and write cost: every row gets an entry, and every insert or update of an indexed column must maintain the tree. For multi-terabyte append-only tables (logs, time series, IoT) where you only need "roughly where is this time range?", that's overkill. A **BRIN** (*Block Range INdex*, PostgreSQL) takes the opposite approach: for each group of consecutive pages record only summary values (min and max), so the whole index might be thousands of times smaller, at the cost of precision.

### 2. Core concepts and vocabulary
- **Page** — fixed-size block read/written by the storage engine.
- **Root / intermediate / leaf pages** — top, middle and bottom levels of the tree; internal pages hold separator keys and child pointers; leaves hold the keys (and row locator or the row itself).
- **Fanout** — number of children per internal page; typically hundreds.
- **Depth (height)** — number of levels; usually 3–4 even for billions of rows.
- **Clustered index** — leaf level *is* the table, ordered by the key (SQL Server, InnoDB).
- **Nonclustered/secondary index** — separate tree whose leaves point to the row (RID or clustered key).
- **Page split** — a full page divides in two when an insert doesn't fit, causing fragmentation.
- **Fill factor** — percentage of each leaf page to fill at build, leaving room for inserts.
- **Leftmost prefix rule** — a composite index can seek on its leading column(s) only.
- **BRIN block range / `pages_per_range`** — number of heap pages summarised per index entry (default 128).
- **Physical correlation** — how well column value order matches physical row order.
- **Lossy index** — returns candidate blocks that must be rechecked.

### 3. How it works, step by step
**B-tree point lookup (`WHERE id = 42`)**
1. Read the root page; binary-search its separator keys to choose a child.
2. Descend through intermediate pages the same way.
3. Reach a leaf; binary-search within it to find the key.
4. Return the row (clustered) or follow the pointer/lookup (nonclustered).

**Range scan**: do steps 1–3 for the lower bound, then follow the leaf-level next-page links until the upper bound — sequential, no re-descending.

**Insert**: find the leaf; if there's room, insert in sorted position; if full, split — allocate a new page, move about half the entries, add a separator to the parent (which may split in turn, up to the root, increasing height by one).

**BRIN scan (`WHERE ts BETWEEN a AND b`)**
1. Read the entire (tiny) BRIN index sequentially.
2. For each range, keep it if `[min,max]` overlaps `[a,b]`.
3. Read only those heap block ranges and re-check each row.

### 4. Worked example
Fanout arithmetic: 8 KB page, 8-byte `bigint` key + 6-byte child pointer + overhead ≈ 16–20 bytes per entry → roughly 400–500 entries per internal page. Level 1 (root): 1 page; level 2: ~450 pages; level 3: ~200,000 pages ≈ 90 million leaf-ish entries; add one more level and ~40 billion. So a 100-million-row table needs **3–4 page reads** per lookup, and the top two levels stay cached in memory.

```sql
-- PostgreSQL: append-only events table, ~2 TB, ts correlates with insertion order
CREATE TABLE events (id bigserial, ts timestamptz NOT NULL, payload jsonb);

CREATE INDEX ix_events_ts_btree ON events (ts);                     -- ~ 40 GB
CREATE INDEX ix_events_ts_brin  ON events USING brin (ts)
       WITH (pages_per_range = 64);                                 -- ~ 20 MB

EXPLAIN (ANALYZE, BUFFERS)
SELECT count(*) FROM events WHERE ts >= '2026-10-01' AND ts < '2026-10-02';
-- Bitmap Heap Scan on events
--   Recheck Cond: (ts >= ... AND ts < ...)
--   Rows Removed by Index Recheck: 31000
--   Heap Blocks: lossy=4800
--   -> Bitmap Index Scan on ix_events_ts_brin
```

Reading it: the BRIN pointed to 4,800 candidate blocks (lossy), the heap scan rechecked rows and discarded some. For a one-day slice that's tiny compared with scanning the table, and the index costs ~0.05 % of the B-tree's space. If rows were inserted in random time order, every range would have a wide min–max and BRIN would select nearly the whole table — useless.

### 5. Under the hood
- **B+ tree pages** store a sorted slot array plus variable-length entries; internal pages hold only separator keys, so more keys fit and the tree is shallow. Leaf pages are doubly linked.
- **Splits and fragmentation.** Random keys (GUIDs) split pages everywhere, giving ~50–70 % fill and poor sequential read locality; ever-increasing keys append at the right edge (cheap, but a hot last page under high concurrency: `PAGELATCH` contention, `OPTIMIZE_FOR_SEQUENTIAL_KEY`). PostgreSQL has deduplication (v13+) and bottom-up deletion (v14) to reduce bloat.
- **Variants.** B-link trees add right-sibling pointers so readers don't hold locks during splits; Fractal/LSM trees buffer writes for write-heavy workloads; SQL Server columnstore and many analytics engines use zone maps (min/max per segment) — the same idea as BRIN.
- **Clustered vs heap.** In InnoDB the secondary index leaf stores the primary key, so a wide PK enlarges every secondary index; in PostgreSQL the table is a heap and indexes hold a `ctid`.
- **BRIN internals.** Each entry stores `(first_block, min, max)` per range; *summarisation* happens at VACUUM (or `brin_summarize_new_values` / `autosummarize`). Other operator classes: `minmax-multi` (several intervals per range, tolerates outliers, PG14), `inclusion` (bounding box for geometric types), `bloom` (equality on uncorrelated columns).
- **Correlation check.** `SELECT correlation FROM pg_stats WHERE tablename='events' AND attname='ts';` — values near ±1 mean BRIN will work.
- **SQL Server has no BRIN**; analogous benefits come from partitioning (partition elimination) and columnstore segment elimination.

### 6. Performance and design implications
- A B-tree is the right default for OLTP lookups, uniqueness, joins and sort order.
- Column order in composite indexes matters: equality columns first, then range, then ordering; the leftmost prefix rule limits which queries can seek.
- Narrow keys → more fanout, smaller indexes, more of the tree cached.
- Heavy indexing taxes writes (each insert modifies each index) and VACUUM/maintenance.
- BRIN shines for append-only, time-correlated data and bulk-loaded sorted data; it needs `CLUSTER` or insertion order to hold. Updates that move rows to new pages degrade correlation.
- Measure index size (`pg_relation_size`, `sys.dm_db_index_physical_stats`), usage (`pg_stat_user_indexes`, `sys.dm_db_index_usage_stats`) and fragmentation before deciding.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE INDEX … (a, b)` | B-tree (default) | General purpose |
| `CREATE INDEX … USING brin (col) WITH (pages_per_range = n)` | Block-range index | Huge, naturally ordered tables |
| `fillfactor` / `FILLFACTOR = 80` | Leave free space per leaf | Update-heavy, random inserts |
| `pg_stats.correlation` | Physical-order correlation | BRIN suitability |
| `brin_summarize_new_values()` / `autosummarize` | Summarise new ranges | After bulk loads |
| `ALTER INDEX … REBUILD/REORGANIZE` | Defragment (SQL Server) | Fragmentation maintenance |
| `INCLUDE (col)` | Covering columns in leaf | Avoid key lookups |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| B-tree for selective lookups and ordering | Table is enormous, append-only, and you only need coarse range pruning |
| BRIN for time-series scans of large slices | Rows arrive in random order or are frequently updated |
| Low `pages_per_range` for precision | More entries → bigger index |
| Composite index with INCLUDE | Many wide included columns |
| Random GUID keys for distribution | Fragmentation and cache churn; consider sequential GUIDs / bigint |

### 9. Common mistakes
- Indexing every column "just in case" → slow writes, huge storage → index from query patterns.
- Wrong column order in a composite index → index unused → leading column must match the filter.
- BRIN on uncorrelated data → scans almost everything → check correlation first.
- Reorganising/rebuilding on a schedule without evidence → wasted I/O → act on measured fragmentation and page density where it matters.
- Using a long varchar as a clustered key → every secondary index swollen → use a narrow, ever-increasing key.
- Forgetting that functions on the column defeat B-tree seeks → rewrite or add an expression index.

### 10. 🎯 Interview kit
**30-second answer:** A B-tree is a balanced, sorted, page-based tree — a few page reads find any key and leaves are linked for range scans. BRIN stores only min/max per block range, so it's tiny and great for huge append-only tables whose values follow physical order, but it's lossy and useless when data isn't correlated.

**Structure a longer answer:** page and I/O model → tree structure and operations (search, insert, split) → clustered vs secondary → when B-tree is wrong → BRIN/zone maps and their prerequisites.

**Likely follow-ups:**
- *Why B+ tree and not binary tree?* → high fanout matches page I/O; shallow depth.
- *What causes page splits and how do you mitigate?* → full pages on out-of-order inserts; fill factor, sequential keys.
- *Why does composite index order matter?* → sorted by first column then second; leftmost prefix.
- *When would you choose BRIN?* → multi-TB time-ordered logs, bulk loaded data.

**Signals interviewers look for:** you reason with page counts and fanout, understand clustered vs secondary, and know BRIN's correlation requirement.

### 11. Check yourself
- [ ] Compute tree depth for 500 million rows with ~400 entries per page.
- [ ] Describe a page split step by step.
- [ ] Explain the leftmost prefix rule with an example.
- [ ] State how BRIN decides which blocks to read and why it must recheck.
- [ ] Name two reasons a BRIN may become ineffective over time.
<!-- explainer:end -->

## Practice questions

- [Basic] [Explain what a B+ tree index gives you that makes both an equality lookup and a date-range scan fast in a warehouse table.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20319_Explain_what_a_B_tree_index_gives_you_that_makes_both_an_equ.md>)
- [Deep] [Why is a B+ tree preferred over a binary search tree for database indexes?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19458_Why_is_a_B_tree_preferred_over_a_binary_search_tree_for_data.md>)
- [Deep] [When would a hash index be a better choice than a B-tree index?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19460_When_would_a_hash_index_be_a_better_choice_than_a_B-tree_ind.md>)
- [Deep] [Trace how a query WHERE id = 500 uses a B+ tree index to find the row.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19535_Trace_how_a_query_WHERE_id_500_uses_a_B_tree_index_to_find_t.md>)
- [Deep] [If a table has 200 million rows and a B+ tree with a fanout of 500, roughly how many levels deep is the index?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19564_If_a_table_has_200_million_rows_and_a_B_tree_with_a_fanout_o.md>)
- [Deep] [Explain, at a beginner-friendly level, what a B-tree structure gives an index that makes lookups and ranges both fast.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20054_Explain_at_a_beginner-friendly_level_what_a_B-tree_structure.md>)
- [Deep] [A query planner switched from an index scan to a bitmap heap scan after data grew, and latency doubled. How do you decide whether the switch was correct and what you'd tune?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19775_A_query_planner_switched_from_an_index_scan_to_a_bitmap_heap.md>)
- [Expert] [What is an LSM tree and how does its indexing differ from a B-tree?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19538_What_is_an_LSM_tree_and_how_does_its_indexing_differ_from_a_.md>)

## Related concepts

- [Leftmost Prefix Rule](<../../SQL_Language/SQL_Fundamentals/Leftmost_Prefix_Rule.md>) *(SQL Fundamentals)*
- [Index Access Paths](<../../Database_Internals/DBMS_Concepts/Index_Access_Paths.md>) *(DBMS Concepts)*
- [B-tree vs LSM Storage Engines](<../../Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/B-tree_vs_LSM_Storage_Engines.md>) *(Storage Engines)*

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — B-Tree Structure
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Clustered Index
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Clustered Index › Nonclustered Index
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Full-Text Search Index Catalog Views and Properties › Range Indexes
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › B-Trees
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › B-tree optimizations
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › Comparing B-Trees and LSM-Trees
- *Designing Data-Intensive Applications* — Martin Kleppmann — B
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Data Storage, Indexing, and Replication › B-tree structures
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Data Storage, Indexing, and Replication › Binary tree writes
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Data Storage, Indexing, and Replication › Indexing
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Data Storage, Indexing, and Replication › Permutations of B-trees
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Infrastructure Operations › Indexing
- *Database Internals* — Alex Petrov — B-Tree Basics › Binary Search Trees
- *Database Internals* — Alex Petrov — B-Tree Basics › Tree Balancing
- *Database Internals* — Alex Petrov — B-Tree Basics › Trees for Disk-Based Storage
- *Database Internals* — Alex Petrov — B-Tree Basics › Ubiquitous B-Trees
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 9 Index Architecture

