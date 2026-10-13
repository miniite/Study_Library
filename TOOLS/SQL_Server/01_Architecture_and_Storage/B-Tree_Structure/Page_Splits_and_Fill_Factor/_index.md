---
id: "TOOLS/SQL_Server/01_Architecture_and_Storage/B-Tree_Structure/Page_Splits_and_Fill_Factor"
level: 5
type: "curriculum"
curriculum: "sql_server"
---

# Page Splits and Fill Factor

**L5 Expert** · [↑ B-Tree Structure](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** When a row must go onto a full B-tree page, SQL Server splits the page — moving about half its rows to a new page — which costs log, latches and leaves fragmentation; fill factor reserves free space at build time so fewer splits happen, at the price of more pages.

### 1. The big picture
A B-tree keeps keys in order, so every row has exactly one correct page. That is what makes seeks and range scans efficient, but it also means a new or grown row *must* fit on that specific page. Heaps solve "row doesn't fit" by **forwarding** the row elsewhere; B-trees (clustered and nonclustered) solve it with a **page split**. Splits are normal and unavoidable — every index grows by splitting — but *mid-index* splits caused by random inserts or growing updates are expensive and create the two kinds of fragmentation that index maintenance deals with.

**Fill factor** is the main design lever: it tells SQL Server to leave a percentage of each leaf page empty when an index is built or rebuilt, giving room for inserts and row growth before splitting is needed. It's a trade-off, not a free win: emptier pages mean more pages to read and cache.

### 2. Core concepts and vocabulary
- **Page split** — allocating a new page and moving part of a full page's rows to it so a new or enlarged row fits in key order.
- **End-of-index ("good") split** — at the rightmost leaf page during ascending inserts: SQL Server just starts a new page and moves nothing (a "100/0" split).
- **Mid-index ("bad") split** — in the middle of the key range: roughly half the rows move (a "50/50" split), every moved row is logged.
- **Internal fragmentation (low page density)** — unused space inside pages; measured by `avg_page_space_used_in_percent`.
- **External (logical) fragmentation** — leaf pages whose logical order (via next-page pointers) differs from physical order in the file; measured by `avg_fragmentation_in_percent`.
- **Fragment** — a run of physically contiguous leaf pages in logical order; `fragment_count` and `avg_fragment_size_in_pages` describe them.
- **Fill factor** — percent of each leaf page to fill on build/rebuild (1–100; 0 and 100 both mean "full").
- **PAD_INDEX** — applies the fill factor to non-leaf (intermediate) pages too.
- **Reorganize** — online, lightweight compaction and reordering of leaf pages in place.
- **Rebuild** — creates a fresh copy of the index (offline or `ONLINE = ON`), resetting density to the fill factor and removing fragmentation.

### 3. How it works, step by step
**A mid-index split** (the books' example): page 100 holds four ~2 KB rows (keys 1–4) and is full.
1. An update makes row 2 grow by 500 bytes (or an insert of key 2.5 arrives). It must stay on page 100 to keep key order, but there is no room.
2. SQL Server allocates a **new page** (101) — from an extent the index already owns if possible, otherwise from a new extent that may be physically far away.
3. About half the rows (e.g. keys 3 and 4) are **moved** to page 101. Each move is logged as a delete + insert.
4. The **linked list** is updated: page 100's next pointer → 101, 101's previous → 100, 101's next → the page that used to follow 100, and that page's previous pointer → 101. Those pages are latched while this happens.
5. A new **index row** for page 101 (its lowest key) is inserted into the **parent** non-leaf page. If the parent is full, it splits too; this can cascade up to the root, and a root split adds a new level.
6. The original operation now completes on whichever page has room.
7. Result: two half-full pages (internal fragmentation) and, if 101 isn't physically next to 100, an out-of-order page (external fragmentation).

**Why splits cost so much:** a single-row insert that would normally write one small log record instead logs moving ~half a page of rows plus allocation and parent updates — often tens of times more log — and holds an exclusive page latch, blocking other sessions on those pages.

**How fill factor changes this:**
1. `CREATE INDEX … WITH (FILLFACTOR = 80)` fills leaf pages to ~80% at build/rebuild time, leaving ~20% (about 1.6 KB) free.
2. New rows and row growth use that free space instead of splitting.
3. Normal inserts *after* the rebuild do **not** respect fill factor — pages fill to 100% before splitting. Fill factor is only re-applied at the next rebuild (or reorganize, which compacts toward it).
4. With `PAD_INDEX = ON`, non-leaf pages get the same free space (rarely needed).

**How deletes fragment:** deleting rows leaves free space (internal fragmentation). If a page becomes empty it's deallocated, leaving gaps in the extent. In the books' delete demo, removing alternating groups of 50 rows left page density low while the page count barely changed — scans read the same number of pages for fewer rows.

**Effect on scans:** imagine a range query for keys 25–90 where pages holding 30 and 90 were split into other extents. The scan switches extents three times (to get 30, back for 50, then to 90). On spinning disks each switch can mean a seek and smaller read-ahead I/Os; on SSDs the cost is much smaller. Internal fragmentation costs everywhere: half-empty pages mean twice the pages to read and twice the memory.

### 4. Worked example
```sql
CREATE TABLE dbo.ClusteredPageSplits (
    RowId      INT IDENTITY(1,1),
    FillerData VARCHAR(2500),
    CONSTRAINT PK_ClusteredPageSplits PRIMARY KEY CLUSTERED (RowId));
INSERT dbo.ClusteredPageSplits (FillerData)
SELECT TOP 24 REPLICATE('X', 2000) FROM sys.objects;      -- 4 rows/page → 6 full leaf pages

SELECT index_level, page_count, record_count,
       avg_fragmentation_in_percent, avg_page_space_used_in_percent
FROM sys.dm_db_index_physical_stats(DB_ID(), OBJECT_ID('dbo.ClusteredPageSplits'), NULL, NULL, 'DETAILED');

-- Grow every other row by 500 bytes → mid-index splits
UPDATE dbo.ClusteredPageSplits SET FillerData = REPLICATE('X', 2500) WHERE RowId % 2 = 0;

SELECT index_level, page_count, record_count,
       avg_fragmentation_in_percent, avg_page_space_used_in_percent
FROM sys.dm_db_index_physical_stats(DB_ID(), OBJECT_ID('dbo.ClusteredPageSplits'), NULL, NULL, 'DETAILED');

-- Page allocations caused by splits (leaf and non-leaf)
SELECT leaf_allocation_count, nonleaf_allocation_count
FROM sys.dm_db_index_operational_stats(DB_ID(), OBJECT_ID('dbo.ClusteredPageSplits'), 1, NULL);

-- Rebuild with free space
ALTER INDEX PK_ClusteredPageSplits ON dbo.ClusteredPageSplits REBUILD WITH (FILLFACTOR = 70);
```
Reading the results:
- Before: 6 leaf pages, fragmentation 0%, page density ~99%.
- After: about 12 leaf pages for the same 24 rows; density falls to ~60–70%; `avg_fragmentation_in_percent` jumps (new pages are out of order). Unlike a heap, there are no forwarded records — rows physically moved to keep key order.
- `leaf_allocation_count` counts new leaf page allocations — for indexes that aren't append-only, that is essentially the split count.
- After the rebuild with fill factor 70, pages are reordered and ~30% free — ready for row growth.

**Measuring split cost with the log** (test system):
```sql
SELECT [AllocUnitName], COUNT(*) AS splits
FROM fn_dblog(NULL, NULL)
WHERE Operation = 'LOP_DELETE_SPLIT'
GROUP BY [AllocUnitName];
```
`LOP_DELETE_SPLIT` records mark rows removed from the old page during a split — a direct split counter per index. An Extended Events session on `sqlserver.transaction_log` filtered on that operation does the same in production.

### 5. Under the hood
- **Split point choice.** For ascending inserts at the end of the index, SQL Server detects the pattern and splits "100/0" (new empty page, nothing moved). For inserts in the middle it normally splits about 50/50 by row count/space; for a growing row it chooses a point so the row fits.
- **Allocation.** The new page comes from free pages in the index's extents (via PFS/IAM); if none, a new uniform extent is allocated — often not adjacent, creating external fragmentation.
- **Logging.** Each moved row produces log records. Split-heavy workloads show high `Log Bytes Flushed/sec` relative to rows changed, larger log backups, and more data sent to availability-group secondaries.
- **Latching.** The splitting thread holds exclusive latches on the page, its new sibling, the neighbour (to fix pointers) and the parent. Concurrent inserts wait on `PAGELATCH_EX`.
- **Random keys.** `NEWID()` GUID keys insert uniformly across the whole leaf level, so once pages are full nearly every insert is a mid-index split; density settles around 70% over time. `NEWSEQUENTIALID()` or `IDENTITY` avoid this.
- **Ascending-key hotspot.** Sequential keys avoid splits but concentrate inserts on the last page (`PAGELATCH_EX` contention on very high-concurrency systems). SQL Server 2019 `OPTIMIZE_FOR_SEQUENTIAL_KEY = ON` mitigates convoy behaviour.
- **Counters.** `SQLServer:Access Methods\Page Splits/sec` counts *all* splits including the harmless end-of-index ones, so compare it as a ratio to `Batch Requests/sec` (one rule of thumb flags more than 1 split per 20 batch requests) and then find the indexes via `leaf_allocation_count`.
- **Heap rebuild vs index rebuild.** Rebuilding a heap changes every RID, forcing all nonclustered indexes to rebuild; rebuilding a clustered index doesn't touch nonclustered indexes (they use keys).

### 6. Performance and design implications
- **Choose keys that minimise mid-index splits:** ever-increasing clustering keys for insert-heavy tables; avoid random GUID clustering keys unless you need to spread hot-spot contention.
- **Avoid row growth on updates:** fixed-length columns for values that are filled later, or default values sized close to final length. Updates that grow `VARCHAR` columns are a common hidden split source.
- **Fill factor per index, not globally.** Server/database default 0 (=100) is recommended; lower it only for indexes with measured split problems — typically 90–95 for moderately volatile indexes, 70–80 for random-key indexes. Never lower fill factor on append-only (IDENTITY) indexes — the free space is never used.
- **More pages cost reads.** Fill factor 80 means 25% more pages for every scan and in memory; the optimizer may even prefer another index.
- **Maintenance decisions:** measure both density (`avg_page_space_used_in_percent`) and external fragmentation; for SSD-backed systems, page density usually matters more than logical order. Reorganizing compacts pages to the fill factor; if you deliberately set low density to avoid splits, aggressive reorganizing can undo that intent — consider density when planning.
- **Monitor:** `Page Splits/sec` ratio, `leaf_allocation_count`, `LOP_DELETE_SPLIT` events, page density trends after rebuild.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sys.dm_db_index_physical_stats` (`avg_fragmentation_in_percent`, `avg_page_space_used_in_percent`, `fragment_count`, `avg_fragment_size_in_pages`) | Fragmentation and density | Maintenance planning (use SAMPLED/DETAILED for density) |
| `sys.dm_db_index_operational_stats` (`leaf_allocation_count`, `nonleaf_allocation_count`) | Page allocations from splits | Find split-heavy indexes |
| `Page Splits/sec` counter | Instance split rate | Trend vs batch requests |
| `CREATE/ALTER INDEX … WITH (FILLFACTOR = n, PAD_INDEX = ON)` | Free space at build | Volatile indexes |
| `ALTER INDEX … REORGANIZE` | Online compaction and ordering | Light fragmentation |
| `ALTER INDEX … REBUILD [WITH (ONLINE = ON)]` | Full rebuild, applies fill factor | Heavy fragmentation, density reset |
| `sp_configure 'fill factor'` | Server default | Leave at 0 |
| `OPTIMIZE_FOR_SEQUENTIAL_KEY = ON` | Last-page insert contention mitigation | High-concurrency IDENTITY inserts (2019+) |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Fill factor 100 (default) — append-only, read-mostly, sequential keys | Random-key or growing-row indexes with measured splits |
| Fill factor 70–90 — random inserts, rows that grow | Applying it to every index "for safety" — more pages everywhere |
| Sequential keys — few splits | Extreme concurrent inserts → last-page latch contention |
| Random keys (GUID) — spread contention | Large tables on slow storage — constant splits, low density |
| Reorganize — small fragmentation, online, interruptible | Low density by design — reorganize compacts it away |
| Rebuild — large fragmentation, reset density | Heaps with many nonclustered indexes; big offline windows |

### 9. Common mistakes
- **Setting a server-wide fill factor of 80** → every index 25% larger, more I/O, little benefit for append-only indexes → set per index where splits are measured.
- **Lowering fill factor on an IDENTITY clustered index** → space reserved that inserts never use → keep 100.
- **Judging fragmentation by `avg_fragmentation_in_percent` alone** → missing half-empty pages → check density too, using SAMPLED/DETAILED mode.
- **Updating rows to grow after insert (e.g. empty `VARCHAR` filled later)** → mid-index splits even with sequential keys → size or default columns appropriately, or use fill factor on that index.
- **Using `Page Splits/sec` raw** → alarm on harmless end-of-index splits → use ratios and per-index allocation counters.
- **Rebuilding everything nightly regardless of need** → huge log, AG traffic, little gain → base maintenance on measurements.

### 10. 🎯 Interview kit
**30-second answer:** In a B-tree a row must go on a specific page; if that page is full, SQL Server splits it: allocates a new page, moves about half the rows, fixes the leaf linked list and adds an entry to the parent, possibly cascading upward. Mid-index splits are expensive — lots of log, latches — and leave half-empty, out-of-order pages (internal and external fragmentation). Fill factor leaves free space at build/rebuild time to absorb inserts and growth, at the cost of more pages. Use it selectively on indexes with measured splits.

**Structure a longer answer:** why splits exist (ordered storage) → step-by-step mechanics → costs (log, latches, fragmentation) → causes (random keys, growing updates) → measure (`leaf_allocation_count`, `LOP_DELETE_SPLIT`, density) → mitigate (key choice, fill factor, maintenance) → scenario: GUID clustered key on a busy orders table.

**Likely follow-ups:**
- *Does fill factor apply to new inserts?* → no, only at create/rebuild/reorganize.
- *Are all page splits bad?* → end-of-index splits move no data and are cheap.
- *Which matters more on SSDs: logical fragmentation or density?* → density (more pages, more memory).
- *How do heaps differ?* → forwarded records instead of splits.

**Signals interviewers look for:** precise split mechanics, distinguishing good vs bad splits and internal vs external fragmentation, measurement before tuning, and selective fill factor use.

### 11. Check yourself
- [ ] Draw the before/after state of a page split, including linked-list and parent updates.
- [ ] Reproduce splits with the demo and read density and fragmentation before and after.
- [ ] Explain why fill factor 80 on an IDENTITY index wastes space.
- [ ] Find the indexes with the most splits on a server using operational stats.
- [ ] Decide fill factor for: an orders table keyed by IDENTITY; a sessions table keyed by GUID; a customer table whose `Notes` column is filled after creation.
<!-- explainer:end -->

## References

- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 12 Dealing With Index Fragmentation
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Splits per Second
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — T-SQL Scripts › Fill Factor
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Balance Index Count › Specify Fill Factors
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Fragmentation Statistics
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Index to Your Environment › Fragmentation Operations
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Indexes › 11.3 #69 Reorganizing indexes to fix page density
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Splits
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Delete Operations
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Indexes › 11.1 #67 Assuming internal fragmentation is always bad

