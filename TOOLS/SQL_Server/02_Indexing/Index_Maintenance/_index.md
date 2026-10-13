---
id: "TOOLS/SQL_Server/02_Indexing/Index_Maintenance"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Index Maintenance

**L4 Advanced** · [↑ Indexing](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Data changes leave indexes with half-empty pages (low page density) and pages out of order (logical fragmentation), heaps with forwarded records, columnstores with deleted rows and open deltastores, and statistics that go stale — maintenance means measuring each of these and fixing only what hurts, with the cheapest operation that actually fixes it.

### 1. The big picture
Indexes are built tidy: pages full to the fill factor, in key order, statistics fresh. Then inserts, updates and deletes go to work:

- **Page splits** leave pages ~50% full and new pages allocated out of sequence.
- **Deletes** leave free space and ghost records.
- **Heaps** grow forwarded records when rows outgrow their page.
- **Columnstores** accumulate deleted-row markers, small or open rowgroups.
- **Shrinking** a data file moves pages backward and scrambles their order.
- **Statistics** describe yesterday's data distribution.

Maintenance has costs too: CPU, I/O, transaction log, blocking (unless online), replicas falling behind, and disrupted page caches. So the modern view is nuanced: *page density* (internal fragmentation) matters for nearly every workload because it means more pages to read and cache; *logical order* (external fragmentation) matters mostly for large range scans from disk. Statistics freshness often matters more than either. Maintain selectively, based on measurements.

### 2. Core concepts and vocabulary
- **Internal fragmentation / low page density** — pages hold less data than they could (`avg_page_space_used_in_percent`). Costs reads, memory and disk.
- **External (logical) fragmentation** — leaf pages whose next-page pointer points to a page that is not the physically next one (`avg_fragmentation_in_percent`). Costs read-ahead efficiency on range scans from disk.
- **Fill factor** — % of each leaf page filled at build/rebuild time (0 or 100 = full). Leaves room for future inserts to reduce splits. `PAD_INDEX = ON` applies it to intermediate levels too.
- **Rebuild** (`ALTER INDEX … REBUILD`) — builds a new copy of the index from scratch, then drops the old one. Removes both kinds of fragmentation, restores fill factor, updates the index's statistics with a full scan.
- **Reorganize** (`ALTER INDEX … REORGANIZE`) — in-place, always online, interruptible: compacts leaf pages *up to* the fill factor and reorders leaf pages by swapping them. Doesn't update statistics.
- **Online rebuild** (`ONLINE = ON`) — keeps the index available during the rebuild; brief locks at start and end only. Enterprise edition (and Azure SQL).
- **Resumable** (`RESUMABLE = ON`, 2017+ rebuild, 2019+ create) — online operation that can be paused, resumed or auto-paused with `MAX_DURATION`.
- **Forwarded record** — in a heap, a stub pointing to a row moved elsewhere; fixed by `ALTER TABLE … REBUILD`.
- **Deltastore / tombstone** — columnstore rowstore buffer for new rows; tombstone = a rowgroup emptied by reorganize, awaiting removal.
- **Statistics** — histograms and density data used by the optimizer for row estimates; maintained by auto-update and by `UPDATE STATISTICS`.

### 3. How it works, step by step
**Step 1 — measure with `sys.dm_db_index_physical_stats(db, object, index, partition, mode)`**
- Modes: `LIMITED` (fastest; reads only the parent level, gives logical fragmentation but not page density), `SAMPLED` (reads ~1% of leaf pages for large indexes; gives density), `DETAILED` (reads all pages; one row per index level).
- Key columns: `index_level`, `page_count`, `record_count`, `avg_fragmentation_in_percent`, `avg_page_space_used_in_percent`, `fragment_count`, `forwarded_record_count` (heaps), `ghost_record_count`.
- Read the **leaf level** (`index_level = 0`). Averaging density across all levels gives misleading numbers: the root and intermediate pages are naturally sparse (a root page with 3 entries is "5% full") — this is why people wrongly conclude that "rebuild didn't remove fragmentation".
- Ignore small indexes: below ~1,000 pages the effect is negligible, and below 8 pages (mixed extents in old versions) fragmentation numbers won't even change after a rebuild.

**Step 2 — decide**

| Finding (leaf level, index ≥ ~1,000 pages) | Typical action |
|---|---|
| Page density well below fill factor (e.g. < 70–80% when FF is 100) | Rebuild, or reorganize (compacts up to FF) |
| Page density *above* target fill factor and many splits | Rebuild (only a rebuild can *lower* density back to the FF) |
| Logical fragmentation high, large range scans from disk | Reorganize (moderate) or rebuild (high) |
| Logical fragmentation high, data mostly in memory / SSD, singleton access | Often nothing — the effect is small |
| Heap with many forwarded records | `ALTER TABLE … REBUILD` (or add a clustered index) |
| Columnstore with > 10–20% deleted rows or many small/open rowgroups | `REORGANIZE` (optionally `COMPRESS_ALL_ROW_GROUPS = ON`) or rebuild |
| Stale statistics on changing tables | `UPDATE STATISTICS` — often the real fix |

The classic thresholds — reorganize at 5–30% logical fragmentation, rebuild above 30% — are a starting point from old guidance. Microsoft's current guidance says to decide by measured benefit for your workload and to consider page density first. Treat thresholds as configurable, not as laws.

**Step 3 — execute**
1. *Rebuild offline*: fastest, takes a schema-modification lock on the table (clustered) or a shared lock (nonclustered) for the duration → blocking.
2. *Rebuild online*: takes a short shared schema lock at the start and a short Sch-M lock at the end; in between, readers and writers continue (changes are applied to both old and new structures). Longer than offline, needs space for both copies plus a mapping index for clustered rebuilds. Caveat: the final Sch-M lock request queues behind long transactions and blocks everyone queued behind it — use `WAIT_AT_LOW_PRIORITY`. Parallel online rebuilds can leave some fragmentation behind, so `MAXDOP = 1` is sometimes recommended for perfect ordering, at the cost of time.
3. *Resumable*: `ALTER INDEX … REBUILD WITH (ONLINE = ON, RESUMABLE = ON, MAX_DURATION = 60 MINUTES)`; `PAUSE`, `RESUME`, `ABORT` from another session; monitor in `sys.index_resumable_operations`. Not compatible with `SORT_IN_TEMPDB` or an explicit transaction.
4. *Reorganize*: single-threaded, online, minimal locks, each small step is its own transaction — can be stopped any time without losing work done. Generates lots of log for heavily fragmented indexes (may exceed rebuild's log). Doesn't update statistics.
5. *Avoid drop and re-create*: the index is missing meanwhile, constraints and FKs get in the way, and dropping a clustered index rebuilds all NC indexes twice. If you must redefine, use `CREATE INDEX … WITH (DROP_EXISTING = ON)`.

**Step 4 — statistics**
- Rebuild updates the rebuilt index's statistics with full scan; column statistics are not touched.
- Reorganize updates none. Add `UPDATE STATISTICS` (or rely on auto-update, possibly with a higher sample rate) as a separate maintenance step.
- Don't run `UPDATE STATISTICS … WITH SAMPLE` right after a rebuild on the same index — it replaces the full-scan statistics with sampled ones.

**Step 5 — automate selectively**
- Maintenance-plan tasks (Reorganize Index, Rebuild Index) can't filter properly by size and fragmentation in older versions, so they often rebuild everything — heavy, mostly wasted work.
- A good job: loop over indexes from `sys.dm_db_index_physical_stats` (`SAMPLED`), filter by page count, density and fragmentation, choose reorganize/rebuild/online per index, log each action, and run statistics updates separately. Community solutions do exactly this; write your own only if you need to.

### 4. Worked example
```sql
-- 1) Measure the leaf level of indexes over 1,000 pages
SELECT OBJECT_NAME(ips.object_id) AS table_name, i.name AS index_name, i.fill_factor,
       ips.page_count, ips.avg_page_space_used_in_percent AS density,
       ips.avg_fragmentation_in_percent AS logical_frag, ips.forwarded_record_count
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'SAMPLED') ips
JOIN sys.indexes i ON i.object_id = ips.object_id AND i.index_id = ips.index_id
WHERE ips.index_level = 0 AND ips.page_count >= 1000 AND ips.alloc_unit_type_desc = 'IN_ROW_DATA'
ORDER BY ips.page_count DESC;

-- Suppose the result shows:
-- Orders.IX_Orders_CustomerGuid  FF 0  pages 120,000  density 58%  logical_frag 97%
-- Orders.PK_Orders               FF 0  pages 410,000  density 99%  logical_frag 0.6%

-- 2) Fix the GUID-keyed index online, leaving room for random inserts
ALTER INDEX IX_Orders_CustomerGuid ON dbo.Orders
REBUILD WITH (ONLINE = ON (WAIT_AT_LOW_PRIORITY (MAX_DURATION = 5 MINUTES, ABORT_AFTER_WAIT = SELF)),
              FILLFACTOR = 80, RESUMABLE = OFF);

-- 3) Or a resumable rebuild for a huge index in a short window
ALTER INDEX PK_BigTable ON dbo.BigTable
REBUILD WITH (ONLINE = ON, RESUMABLE = ON, MAX_DURATION = 60 MINUTES);
SELECT index_id, state_desc, percent_complete FROM sys.index_resumable_operations;
-- next window: ALTER INDEX PK_BigTable ON dbo.BigTable RESUME;

-- 4) Columnstore: check deleted rows per rowgroup, then reorganize
SELECT row_group_id, state_desc, total_rows, deleted_rows,
       100 * (total_rows - ISNULL(deleted_rows,0)) / NULLIF(total_rows,0) AS pct_live
FROM sys.dm_db_column_store_row_group_physical_stats
WHERE object_id = OBJECT_ID('dbo.SalesFact');
ALTER INDEX CCI_SalesFact ON dbo.SalesFact REORGANIZE WITH (COMPRESS_ALL_ROW_GROUPS = ON);
```
**Reading it.** The GUID index is the real problem: 58% density means ~70% more pages than necessary, in cache and on disk, and it will keep splitting. Rebuilding with fill factor 80 restores density to ~80% and leaves room for random inserts, so splits slow down — re-measure after a week to tune the fill factor. The 410,000-page PK at 99% density and 0.6% fragmentation needs nothing; rebuilding it nightly would just burn I/O and log. For the columnstore, rowgroups with more than ~10% deleted rows get merged by reorganize; emptied rowgroups appear as `TOMBSTONE` and are removed later by the background tuple mover.

### 5. Under the hood
- **Rebuild mechanics.** A rebuild reads the existing index in key order (or the base data), sorts if needed (in the destination filegroup or in tempdb with `SORT_IN_TEMPDB = ON`), writes brand-new pages sequentially into fresh extents and then swaps metadata. Space needed: roughly the size of the index again (plus sort space). In FULL recovery it is fully logged; in BULK_LOGGED/SIMPLE an offline rebuild is minimally logged.
- **Online rebuild internals.** During the build, concurrent DML is applied to both the old and the new structure (versioning is used to read a consistent snapshot); for a clustered index a temporary mapping index tracks old→new row locations. The short Sch-M lock at the end swaps the structures.
- **Reorganize mechanics.** It walks the leaf level in logical order. Compaction moves rows from later pages into earlier ones up to the fill factor (freeing emptied pages). Defragmentation swaps page contents so the logical order matches allocation order within the existing pages — it never allocates new extents, so it cannot fix interleaving with other objects' extents, and it cannot *lower* density below current levels. It also doesn't move pages between files.
- **Why density above the fill factor matters.** If an index has FF 70 to absorb inserts, but pages have since filled to 100%, reorganize won't help (it only compacts up to the FF); pages stay full, and bad splits resume immediately. Only a rebuild redistributes rows back to 70%.
- **Shrink is the enemy.** `DBCC SHRINKFILE/SHRINKDATABASE` moves pages from the end of the file into free space at the front, one by one, reversing their order — producing near-100% logical fragmentation. If you must shrink, rebuild afterwards (which may grow the file again) — better, avoid shrinking.
- **Columnstore (2025 note).** Through SQL Server 2022 a columnstore rebuild was offline; SQL Server 2025 adds online rebuild for columnstore indexes. Reorganize remains the normal tool: it compresses closed deltastores (tuple mover) and merges/removes rowgroups with many deleted rows.

### 6. Performance and design implications
- Low page density = more logical reads for everything (scans and range seeks) and more memory per useful row; it's the most important fragmentation metric.
- Logical fragmentation hurts read-ahead on spinning disks for large ordered scans; on SSD/memory-resident data its effect is much smaller.
- Rebuilds generate log equal to roughly the index size (FULL recovery): watch log backups, availability-group redo queues, and log-shipping latency.
- Frequent rebuild of indexes that re-fragment immediately (random keys) is a cycle of waste; fix with fill factor or a better key.
- Measure benefit: compare query reads/durations before and after maintenance; if nothing changes, reduce maintenance.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sys.dm_db_index_physical_stats(…, 'LIMITED'/'SAMPLED'/'DETAILED')` | Fragmentation, density, forwarded records | Deciding what to maintain |
| `ALTER INDEX … REBUILD [PARTITION = n] WITH (ONLINE = ON, FILLFACTOR = n, MAXDOP = n, SORT_IN_TEMPDB = ON)` | Full rebuild | Low density, high fragmentation |
| `WITH (RESUMABLE = ON, MAX_DURATION = n MINUTES)` + `PAUSE/RESUME/ABORT` | Pausable online rebuild | Large indexes, short windows |
| `WAIT_AT_LOW_PRIORITY (MAX_DURATION, ABORT_AFTER_WAIT)` | Controls the final lock wait | Busy OLTP systems |
| `ALTER INDEX … REORGANIZE [WITH (LOB_COMPACTION = ON)]` | In-place compaction and reordering | Moderate fragmentation, no Enterprise online |
| `ALTER INDEX ALL ON t REBUILD` | All indexes of one table | Table-level maintenance |
| `ALTER TABLE t REBUILD` | Rebuild a heap | Forwarded records |
| `REORGANIZE WITH (COMPRESS_ALL_ROW_GROUPS = ON)` | Compress open/closed deltastores | Columnstore |
| `sys.dm_db_column_store_row_group_physical_stats` | Rowgroup state and deleted rows | Columnstore health |
| `UPDATE STATISTICS t [stat] WITH FULLSCAN / SAMPLE n PERCENT` | Refresh statistics | Separate from index maintenance |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Rebuild — density far below target, very high fragmentation, need fresh index stats | Huge index, no online option, tight log/replica capacity |
| Online rebuild — 24/7 systems on Enterprise | Long-running transactions holding locks (final Sch-M wait) |
| Resumable — rebuild doesn't fit one window | Need `SORT_IN_TEMPDB`, or inside explicit transactions |
| Reorganize — moderate fragmentation, Standard edition, must be interruptible | Density above fill factor (can't fix), very high fragmentation (log heavy) |
| Lower fill factor — random-key indexes that split constantly | Ever-increasing keys (wastes space, no benefit) |
| Doing nothing — small indexes, memory-resident data, singleton access | Large range scans from disk on badly fragmented indexes |

### 9. Common mistakes
- Rebuilding everything every night → hours of I/O, huge log, AG lag, no measurable gain → filter by size, density and fragmentation.
- Averaging `avg_page_space_used_in_percent` across all levels → "rebuild didn't work" → read the leaf level only.
- Reorganizing an index whose pages are fuller than its fill factor → no change, splits continue → rebuild.
- No maintenance at all on 24/7 systems "because rebuild blocks" → bloated indexes → use online/resumable rebuilds.
- Shrinking the database after maintenance → fragmentation is back → don't shrink routinely.
- Updating statistics with a sample right after a rebuild → replaces full-scan statistics → skip stats for just-rebuilt indexes.
- Treating 5%/30% as hard rules → wasted work or missed problems → decide by measured impact.

### 10. 🎯 Interview kit
**30-second answer:** "I measure with `sys.dm_db_index_physical_stats` at the leaf level, for indexes over about a thousand pages. Page density matters most — half-empty pages mean more reads and memory; logical fragmentation matters mainly for big scans from disk. Rebuild creates a fresh copy, fixes both and updates the index's statistics; it can be online and resumable on Enterprise. Reorganize is in-place, always online and interruptible, but only compacts up to the fill factor and doesn't update stats. I don't rebuild everything nightly; I target what's degraded, tune fill factor for random-key indexes, and handle statistics separately."

**Structure a longer answer:** causes (splits, deletes, shrink, heaps, columnstore deletes) → two metrics and their costs → rebuild vs reorganize mechanics → online/resumable and locking caveats → statistics → selective automation → scenario: nightly full rebuild replaced by a targeted job, maintenance time from 4 h to 20 min, AG lag gone.

**Likely follow-ups:**
- *Does rebuild update statistics?* → yes, index stats with full scan; not column stats. Reorganize doesn't.
- *What locks does an online rebuild take?* → short S/Sch-S at start, Sch-M at end; IS during build; low-priority waits.
- *Why might fragmentation return immediately?* → random keys + full pages; lower fill factor or change key.
- *What does shrink do to indexes?* → reverses page order → heavy logical fragmentation.
- *Columnstore maintenance?* → deleted rows and open deltastores → reorganize, `COMPRESS_ALL_ROW_GROUPS`.

**Signals interviewers look for:** distinguishes density from logical fragmentation; knows the mechanics and limits of reorganize; uses online/resumable options; ties maintenance to measured benefit; remembers statistics.

### 11. Check yourself
- [ ] Query leaf-level density and fragmentation for indexes above 1,000 pages.
- [ ] Explain why reorganize can't fix an index with fill factor 70 whose pages are now 100% full.
- [ ] Run a resumable online rebuild, pause it, check progress, resume it.
- [ ] Show the effect of `DBCC SHRINKFILE` on fragmentation in a test database.
- [ ] Inspect a columnstore's rowgroups and decide whether to reorganize.
- [ ] Design a nightly job: filters, actions, logging, statistics step.
<!-- explainer:end -->

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Index Maintenance (Rebuild and Reorganize)
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Maintenance Plans
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Shrink Operations › Fragmentation Variants
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Index to Your Environment › Chapter 9
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Indexes › 11.6 #72 Rebuilding all indexes indiscriminately
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Indexes › 11.5 #71 Not rebuilding indexes
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Indexes › 11.3 #69 Reorganizing indexes to fix page density
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 12 Dealing With Index Fragmentation
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Indexes › 11.4 #70 Misinterpreting fragmentation statistics
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 10 Index Behaviors

