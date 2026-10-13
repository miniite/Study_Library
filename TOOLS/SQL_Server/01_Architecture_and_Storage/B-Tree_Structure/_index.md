---
id: "TOOLS/SQL_Server/01_Architecture_and_Storage/B-Tree_Structure"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# B-Tree Structure

**L4 Advanced** · [↑ Architecture and Storage](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A B-tree keeps index keys sorted across a shallow, balanced hierarchy of 8 KB pages, so SQL Server can find any key in a handful of page reads and then scan neighbours through a linked leaf level.

### 1. The big picture
SQL Server stores a table in one of two shapes: a **heap** (no order) or a **B-tree** (sorted). Both clustered and nonclustered rowstore indexes are B-trees, so this structure underlies almost every seek, range scan, ordered scan, page split and fragmentation discussion you will meet later in the indexing block.

The point of the B-tree is to minimise page reads. Without ordering, finding one row means reading every page. With a B-tree, the engine starts at one page (the root), reads one page per level, and lands on the page holding the key. Because the tree is *balanced* — every leaf is the same distance from the root — the cost is the same for any key: finding the value 5 and the value 25 both take the same number of reads. Even tables with hundreds of millions of rows usually have only three or four levels.

### 2. Core concepts and vocabulary
- **B-tree (balanced tree)** — a multi-level tree where every path from root to leaf has the same length. (In SQL Server it is technically a B+ tree: all row data lives at the leaf level.)
- **Root page** — the single top page. It holds key values and pointers to pages on the next level down.
- **Intermediate level** — index pages between the root and the leaves. Small indexes have none; most have one or two.
- **Non-leaf levels** — root plus intermediate levels; they hold only keys and child-page pointers (index rows).
- **Leaf level** — the bottom level. In a **clustered** index the leaf pages *are* the data pages (whole rows). In a **nonclustered** index the leaf pages hold the index key, any `INCLUDE` columns, and a row locator (clustering key or heap RID).
- **Index row** — an entry on a non-leaf page: the lowest key value of a child page plus that page's address (file:page).
- **Doubly linked list** — every page at a level stores the IDs of its previous and next pages in its header, so a scan can move sideways without going back up the tree.
- **IAM page (Index Allocation Map)** — tracks which extents belong to the index within a 4 GB GAM interval; it tells the engine where the tree's pages are.
- **Index depth** — number of levels; returned by `INDEXPROPERTY(..., 'IndexDepth')` or `sys.dm_db_index_physical_stats`.
- **Fan-out** — how many child pointers fit on one non-leaf page. Narrow keys → high fan-out → fewer levels.

### 3. How it works, step by step
**Seeking a single key** (e.g. `WHERE Id = 5` on an index over 27 rows spread across 9 leaf pages):
1. The engine reads the **root page**. Its index rows say "keys from 1 → page A, from 10 → page B, from 19 → page C". 5 is ≥ 1 and < 10, so it follows page A.
2. It reads **intermediate page A**: "from 1 → leaf 1, from 4 → leaf 2, from 7 → leaf 3". 5 is ≥ 4 and < 7 → leaf 2.
3. It reads **leaf page 2** and uses the page's slot array (binary search) to locate the row with key 5.
4. Total: **3 logical reads** — one per level. A lookup of 25 also costs 3.

**Range scan** (e.g. `WHERE Id BETWEEN 5 AND 14`):
1. Seek to the first qualifying key (5) as above.
2. Read rows forward on that leaf page.
3. Follow the **next-page pointer** in the page header to the following leaf, and continue until a key exceeds 14. No return trip to the root.

**Ordered scan backwards** (`ORDER BY Id DESC`) works the same way using the previous-page pointer.

**Inserting** a key: seek to the leaf where it belongs. If there is room, add it and update the slot array. If not, the page **splits** (half the rows move to a new page and a new index row is added to the parent); if the parent is full it splits too, and if the root splits the tree gains a level. This is how the tree stays balanced — it grows from the top. (Detailed in the *Page Splits and Fill Factor* lesson.)

| Level | Contains | Pointers to |
|---|---|---|
| Root (1 page) | lowest key of each child + page id | Intermediate pages |
| Intermediate | lowest key of each child + page id | Leaf pages |
| Leaf (clustered) | entire data rows, sorted by key | prev/next leaf pages |
| Leaf (nonclustered) | key + INCLUDE columns + row locator | prev/next leaf pages |

### 4. Worked example
```sql
CREATE TABLE dbo.BTreeDemo (
    Id      INT IDENTITY PRIMARY KEY CLUSTERED,
    Filler  CHAR(400) NOT NULL DEFAULT 'x'
);
INSERT dbo.BTreeDemo DEFAULT VALUES;
GO 100000

-- How many levels and pages per level?
SELECT index_level, page_count, record_count, avg_record_size_in_bytes
FROM sys.dm_db_index_physical_stats(DB_ID(), OBJECT_ID('dbo.BTreeDemo'), 1, NULL, 'DETAILED')
ORDER BY index_level DESC;

-- Seek vs scan cost
SET STATISTICS IO ON;
SELECT * FROM dbo.BTreeDemo WHERE Id = 54321;           -- seek
SELECT * FROM dbo.BTreeDemo WHERE Id BETWEEN 1 AND 500;  -- range scan
SET STATISTICS IO OFF;

-- Look at the root page (undocumented, test systems only)
SELECT allocated_page_page_id, page_type_desc, page_level, previous_page_page_id, next_page_page_id
FROM sys.dm_db_database_page_allocations(DB_ID(), OBJECT_ID('dbo.BTreeDemo'), 1, NULL, 'DETAILED')
WHERE page_type_desc = 'INDEX_PAGE'
ORDER BY page_level DESC;
```
What to look for:
- `dm_db_index_physical_stats` returns one row per level. With ~400-byte rows, about 19 rows fit per page, so 100,000 rows need roughly 5,300 leaf pages (`index_level = 0`). Each non-leaf index row is tiny (a 4-byte int key plus a 6-byte page pointer plus overhead), so one page holds hundreds of them: about 15 pages at level 1 and a single root page at level 2. **Depth = 3.**
- The single-row seek reports **3 logical reads** — one per level.
- The range of 500 rows reports about 2 non-leaf reads plus ~27 leaf pages: the engine seeks once and then walks the linked list.
- The allocation query shows the root has no previous/next page, while intermediate and leaf pages are chained.

### 5. Under the hood
- **Page layout.** Every index page has a 96-byte header (page type, level, previous/next page IDs, free space), rows in the body, and a **slot array** at the end of the page that stores row offsets in key order. Rows themselves may be physically out of order on the page; the slot array provides the logical order and makes binary search possible.
- **Non-leaf key only.** Non-key columns are never stored on non-leaf pages. For a nonclustered index with `INCLUDE` columns, those columns appear only at the leaf level — this is why `INCLUDE` is cheaper than adding a column to the key.
- **Non-unique clustered indexes** get a hidden 4-byte **uniquifier** on duplicate keys so each row can be identified; non-unique nonclustered indexes carry the row locator in the non-leaf rows as well, to keep entries unique.
- **Fan-out arithmetic.** A non-leaf row for an `INT` key is about 11–12 bytes, so ~600+ fit on an 8,060-byte page. Three levels then address 600 × 600 leaf pages ≈ 360,000 pages; four levels ≈ 216 million pages. A wide key (say a 200-byte composite) cuts fan-out to ~40 and adds levels.
- **IAM chain.** Like a heap, a B-tree starts from an IAM page that maps its extents. The root page ID is stored in system metadata (`sys.system_internals_allocation_units.root_page`), so the engine knows where to start every seek.
- **Physical vs logical order.** The linked list gives logical order. After page splits, the next page may be physically far away on disk — this mismatch is *logical fragmentation*. It hurts large read-ahead scans on spinning disks more than single-row seeks.
- **Columnstore** indexes are *not* B-trees; they store column segments and are covered elsewhere.

### 6. Performance and design implications
- **Seek cost ≈ depth.** A well-keyed seek costs 2–4 logical reads regardless of table size; extra cost comes from key lookups, not from the tree.
- **Keep clustering keys narrow** — the key is repeated in every non-leaf row *and* in every nonclustered index row as the locator. Narrow keys mean higher fan-out and smaller nonclustered indexes.
- **Ever-increasing keys** (IDENTITY, sequential dates) insert at the end of the leaf chain: few page splits, but all inserts compete for the last page (a hot spot under very high concurrency — `PAGELATCH_EX` waits).
- **Random keys** (`NEWID()` GUIDs) insert all over the tree: many page splits, fragmentation and more log, but no single hot page.
- **Indexes have a write cost.** Each `INSERT`, `UPDATE` of a key column, or `DELETE` must also maintain every B-tree on the table — more storage, more memory, more log, more locks. Index for the workload, not for every column.
- **Measure** with `SET STATISTICS IO`, `sys.dm_db_index_physical_stats` (depth, page counts, fragmentation) and `sys.dm_db_index_operational_stats` (leaf inserts, page splits via `leaf_allocation_count`).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sys.dm_db_index_physical_stats(..., 'DETAILED')` | Pages, rows, record size, fragmentation per level | Inspecting tree shape and depth |
| `INDEXPROPERTY(obj, idx, 'IndexDepth')` | Number of levels | Quick depth check |
| `sys.dm_db_database_page_allocations` | Lists pages, type, level, prev/next | Learning/diagnostics (undocumented) |
| `DBCC IND` / `DBCC PAGE` | Older way to list and dump pages | Deep internals on test systems |
| `SET STATISTICS IO ON` | Logical reads per table | Verifying seek vs scan cost |
| `CREATE INDEX … INCLUDE (…)` | Adds columns to leaf only | Covering queries without widening the key |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| B-tree (clustered) for most OLTP tables — seeks, ranges, ordered output | Huge append-only scan workloads → consider columnstore |
| Narrow, unique, static, ever-increasing clustering key | Wide or frequently updated clustering keys — every nonclustered index pays |
| `INCLUDE` columns to cover queries | Including many large columns — the leaf level becomes a second copy of the table |
| Random keys to spread extreme insert concurrency | Random keys on spinning disks or large scans — splits and fragmentation |

### 9. Common mistakes
- **Thinking a bigger table means a much slower seek** → over-engineering partitioning for "speed" → depth grows logarithmically; check `IndexDepth` first.
- **Using a wide composite or `NVARCHAR(200)` clustering key** → low fan-out, bloated nonclustered indexes → use a surrogate key for clustering and enforce the natural key with a unique nonclustered index.
- **Putting every filter column in the key "to be safe"** → wider non-leaf pages and more maintenance → put seek/order columns in the key, the rest in `INCLUDE`.
- **Believing the leaf level is physically sorted on disk** → wrong conclusions about `ORDER BY` without an `ORDER BY` → only logical order is guaranteed, and only for the scan the plan chooses; always write `ORDER BY`.

### 10. 🎯 Interview kit
**30-second answer:** SQL Server rowstore indexes are B+ trees of 8 KB pages: a root, optional intermediate levels holding keys and child pointers, and a leaf level holding full rows (clustered) or key + includes + row locator (nonclustered). Seeks read one page per level — usually 3–4 — and range scans then follow a doubly linked list across leaf pages. The tree stays balanced by splitting pages and growing from the root.

**Structure a longer answer:** clarify (clustered vs nonclustered) → walk a seek level by level → range scans and the linked list → inserts and page splits → trade-offs of key width and key order → close with a scenario: GUID vs IDENTITY clustering key.

**Likely follow-ups:**
- *Why does a seek on 1 billion rows take only ~4 reads?* → fan-out of hundreds per page; depth is logarithmic.
- *What's stored in a nonclustered leaf?* → key, INCLUDE columns, and the clustering key (or RID for a heap).
- *What is fragmentation?* → logical order of leaf pages ≠ physical order, plus empty space from splits.
- *Why keep the clustering key narrow?* → it's copied into every nonclustered index.

**Signals interviewers look for:** you can compute rough depth from row size, you separate logical from physical order, and you connect key choice to write cost and nonclustered index size.

### 11. Check yourself
- [ ] Draw a three-level B-tree for keys 1–27 and trace a seek for 5 and for 25.
- [ ] Explain what a non-leaf index row contains and what a leaf row contains for clustered vs nonclustered indexes.
- [ ] Run `dm_db_index_physical_stats` in DETAILED mode and read the depth and page counts per level.
- [ ] Estimate the depth of an index with a 4-byte key over 50 million rows.
- [ ] Explain why a range scan doesn't need to return to the root.
<!-- explainer:end -->

## In this section (study in order)

1. [Page Splits and Fill Factor](<Page_Splits_and_Fill_Factor/_index.md>) (L5 Expert)

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — B-Tree Structure
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 9 Index Architecture
- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — Tree Data Structures › 5. Tree Data Structures
- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — A Graph For Testing › The End (or Is It the Beginning?)
- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — SQL Graph Table Basics › Traversing Variable Level Paths
- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — Tree Data Structures › Code To Create New Nodes
- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — SQL Graph Table Basics › Displaying the Last Node in the Path
- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — Other Directed Acyclic Graphs › Printing Out the Parts List for a Build
- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — Tree Structures, Algorithms, and Performance › 6. Tree Structures, Algorithms, and Performance
- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — Tree Data Structures › Aggregating Child Activity at Every Level

