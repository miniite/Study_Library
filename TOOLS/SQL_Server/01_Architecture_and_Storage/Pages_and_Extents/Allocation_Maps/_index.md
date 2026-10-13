---
id: "TOOLS/SQL_Server/01_Architecture_and_Storage/Pages_and_Extents/Allocation_Maps"
level: 5
type: "curriculum"
curriculum: "sql_server"
---

# Allocation Maps (GAM SGAM PFS IAM)

**L5 Expert** · [↑ Pages and Extents](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** SQL Server tracks space with bitmap pages at fixed positions in every data file — GAM and SGAM for extents, PFS for individual pages, IAM for which extents belong to which object, plus DCM/BCM for backups — and every allocation reads and latches them.

### 1. The big picture
When an insert needs a new page, or a scan needs to find every page of a heap, SQL Server does not search the file. It consults small bitmap pages that summarise the state of thousands of pages or extents at once. These **allocation maps** answer four questions:

1. *Which extents are free?* → **GAM** (Global Allocation Map).
2. *Which mixed extents still have a free page?* → **SGAM** (Shared GAM).
3. *Is this page allocated, how full is it, does it have ghost records, is it an IAM page?* → **PFS** (Page Free Space).
4. *Which extents belong to this table/index allocation unit?* → **IAM** (Index Allocation Map).

Two more maps serve backup: **DCM** (Differential Changed Map) records extents changed since the last full backup, and **BCM** (Bulk Changed Map) records extents changed by minimally logged operations since the last log backup.

Understanding them explains tempdb allocation contention (`PAGELATCH_UP` on `2:1:1` / `2:1:3`), why heaps rely on PFS for inserts, why differential backups are fast, and why log backups in BULK_LOGGED include data extents. Note that these structures exist only in **data files** — the log file has its own (VLF) architecture.

### 2. Core concepts and vocabulary
- **GAM interval** — the range of extents one GAM/SGAM/DCM/BCM page maps: about 64,000 extents ≈ **4 GB** of file. One bit per extent.
- **PFS interval** — one PFS page covers **8,088 pages** (~64 MB), one **byte** per page.
- **GAM bit** — `1` = extent is free; `0` = extent is allocated (uniform or mixed).
- **SGAM bit** — `1` = extent is a mixed extent *with at least one free page*; `0` = not mixed, or mixed but full.
- **PFS byte** — free-space band (bits 0–2: empty, 1–50%, 51–80%, 81–95%, 96–100%), ghost-records flag (bit 3), IAM-page flag (bit 4), mixed-page flag (bit 5), allocated flag (bit 6).
- **IAM page** — belongs to one allocation unit and one GAM interval; a bitmap of the extents in that interval owned by the allocation unit, plus a header with up to 8 single-page (mixed) allocations. IAM pages for an allocation unit form an **IAM chain**.
- **DCM bit** — extent changed since last full backup (`1`).
- **BCM bit** — extent changed by a minimally logged operation since last log backup (`1`).
- **Allocation unit** — IN_ROW_DATA, ROW_OVERFLOW_DATA or LOB_DATA for one partition of one index/heap; each has its own IAM chain.

### 3. How it works, step by step
**Fixed layout of every data file**

| Page | Content |
|---|---|
| 0 | File header (file ID, filegroup ID, size, max size, growth…) |
| 1 | First PFS (then every 8,088 pages) |
| 2 | First GAM (then every GAM interval, ~511,232 pages) |
| 3 | First SGAM (then every GAM interval) |
| 4–5 | Unused/reserved |
| 6 | First DCM |
| 7 | First BCM |
| 9 | Boot page (primary file only: database-wide metadata) |
| 10+ | IAM, data, index, LOB pages… |

**Combined meaning of GAM + SGAM bits for an extent**

| GAM | SGAM | Meaning |
|---|---|---|
| 1 | 0 | Free extent |
| 0 | 0 | Uniform extent, or mixed extent with no free pages |
| 0 | 1 | Mixed extent with free pages |

**Allocating a uniform extent to an index**
1. Find a GAM bit = 1 in a suitable file/interval (proportional fill chooses the file).
2. Flip it to 0 (the extent is now allocated) — latching the GAM page.
3. Set the corresponding bit in the allocation unit's IAM page for that interval (creating a new IAM page if the object has none for this interval).
4. Update PFS bytes as pages within the extent get used.

**Allocating a single page from a mixed extent** (when mixed allocation is on, for the first eight pages):
1. Look in SGAM for an extent with bit = 1 (mixed with free page). If none, take a free extent from GAM (set GAM 0) and mark it mixed with free space (SGAM 1).
2. Check PFS bytes to find an unallocated page in that extent; mark it allocated.
3. Record the page in the IAM header's single-page slots.
4. If the mixed extent is now full, set its SGAM bit to 0.

**Heap insert**
1. Read the heap's IAM chain to know its extents.
2. Read PFS bytes for those pages to find one in a free-space band that can hold the row.
3. Insert; update the PFS byte if the band changes. If none fits, allocate a new page/extent as above. Each such search shows up in the `FreeSpace Scans/sec` counter.

B-tree inserts don't need PFS to find space — the key decides the page — but PFS still tracks allocation status and ghost records.

**Ghost cleanup**: deleted rows in indexes become *ghost records*; the PFS ghost bit lets the background cleanup task visit only pages with ghosts instead of every page.

**Differential and log backups**
- A **differential backup** reads DCM pages and copies only extents with bit = 1; a full backup resets the bits.
- In **BULK_LOGGED**, a **log backup** reads BCM pages and also copies the data extents changed by minimally logged operations (so the backup can redo them); then it resets the BCM bits.

### 4. Worked example
```sql
-- 1. Allocation pages of the current database (2019+): page types at fixed positions
SELECT page_id, page_type_desc
FROM sys.dm_db_page_info(DB_ID(), 1, 1, 'LIMITED')   -- PFS
UNION ALL SELECT page_id, page_type_desc FROM sys.dm_db_page_info(DB_ID(), 1, 2, 'LIMITED') -- GAM
UNION ALL SELECT page_id, page_type_desc FROM sys.dm_db_page_info(DB_ID(), 1, 3, 'LIMITED'); -- SGAM

-- 2. A table's IAM chain and owned extents
SELECT allocation_unit_type_desc, page_type_desc, allocated_page_page_id, extent_page_id,
       is_iam_page, is_mixed_page_allocation, page_free_space_percent
FROM sys.dm_db_database_page_allocations(DB_ID(), OBJECT_ID('dbo.PageDemo'), NULL, NULL, 'DETAILED')
ORDER BY is_iam_page DESC, allocated_page_page_id;

-- 3. tempdb allocation contention right now
SELECT wt.session_id, wt.wait_type, wt.resource_description,
       pi.page_type_desc
FROM sys.dm_os_waiting_tasks wt
CROSS APPLY sys.fn_PageResCracker(wt.resource_address) prc
CROSS APPLY sys.dm_db_page_info(prc.db_id, prc.file_id, prc.page_id, 'LIMITED') pi
WHERE wt.wait_type LIKE 'PAGELATCH%';

-- 4. How much a differential backup would copy (2016 SP2+)
SELECT file_id, modified_extent_page_count, allocated_extent_page_count,
       100.0 * modified_extent_page_count / NULLIF(allocated_extent_page_count, 0) AS pct_changed
FROM sys.dm_db_file_space_usage;
```
How to read it:
- Query 1 confirms page 1 is `PFS_PAGE`, 2 is `GAM_PAGE`, 3 is `SGAM_PAGE`.
- Query 2 shows the IAM page first (`is_iam_page = 1`) and the extents it maps; `page_free_space_percent` is the PFS band for each heap/LOB page.
- Query 3 turns wait resources like `2:1:1` into page types. Many sessions on `PFS_PAGE`, `GAM_PAGE` or `SGAM_PAGE` in tempdb = allocation contention → add equally sized tempdb files (and, from 2019, memory-optimized tempdb metadata for system-table contention).
- Query 4 reads the DCM-derived change counts: if `pct_changed` is high, a full backup may be better than a differential.

### 5. Under the hood
- **Why bitmaps?** One 8 KB GAM page has ~64,000 usable bits → 64,000 extents × 64 KB = 4 GB. A single page read answers "where is free space in 4 GB?". PFS uses a byte per page because it stores more state.
- **Latching.** Changing an allocation bit requires an exclusive (`PAGELATCH_UP`/`EX`) latch on that map page. With only one data file, every allocation in tempdb serialises on the same PFS/GAM/SGAM pages. More files → more map pages → parallel allocation.
- **PFS contention fix (2019).** SQL Server 2019 updates PFS pages with a shared latch ("concurrent PFS updates"), easing PFS contention in all databases.
- **Mixed extents by default off (2016+).** User databases and tempdb allocate uniform extents from the first page (`MIXED_PAGE_ALLOCATION OFF`), so SGAM is rarely touched — eliminating a classic tempdb hotspot (previously trace flag 1118).
- **IAM ordering.** IAM pages are linked in allocation order, not file order; an allocation-order scan (heaps, or `NOLOCK`/`TABLOCK` scans of B-trees) follows IAM bitmaps in file order within each interval, which is fast but can miss/duplicate rows under concurrent splits — one reason `NOLOCK` scans are unsafe.
- **Coarse PFS bands.** A page in the 51–80% band might have 1.6 KB or 3.9 KB free; a 2 KB row may or may not fit. The engine tries and moves on, which is why heaps with wide rows can show high `FreeSpace Scans/sec`.
- **Corruption.** Allocation-map corruption is detected by `DBCC CHECKDB` (`CHECKALLOC` part); errors such as "page is allocated to two objects" or "extent marked allocated in GAM but not owned by any IAM" come from cross-checking these bitmaps.

### 6. Performance and design implications
- **tempdb:** multiple equally sized data files (up to 8 to start, matching cores), uniform growth, and SQL Server 2019+ features (concurrent PFS updates, memory-optimized metadata).
- **Heaps with heavy inserts:** watch `FreeSpace Scans/sec` relative to `Batch Requests/sec`; find the heaps with the most `leaf_insert_count` in `sys.dm_db_index_operational_stats`. A clustered index (or bulk loads with `TABLOCK`) removes the PFS search.
- **Backups:** differential size depends on DCM-marked extents; a few row changes spread across many extents mark many extents.
- **BULK_LOGGED log backups** can be large (they include bulk-changed extents) even though the log itself stayed small.
- **Large user databases:** multiple data files spread allocation work and I/O, provided they're equally sized so proportional fill stays balanced.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sys.dm_db_page_info(db, file, page, mode)` | Page header info incl. type | Identify a page from a wait resource (2019+) |
| `sys.fn_PageResCracker` | Converts `page_resource` to db/file/page | With `dm_os_waiting_tasks` |
| `sys.dm_db_database_page_allocations` | Pages, IAM, extents, PFS free % | Learning/diagnostics (undocumented) |
| `sys.dm_db_file_space_usage` (`modified_extent_page_count`) | DCM-based change tracking | Choose full vs differential |
| `DBCC PAGE (db, file, page, 3)` + TF 3604 | Dump allocation bitmaps | Internals on test systems |
| `ALTER DATABASE … SET MIXED_PAGE_ALLOCATION ON/OFF` | Mixed extent behaviour | Rarely changed |
| `FreeSpace Scans/sec` counter | Heap free-space searches | Heap insert overhead |
| `DBCC CHECKALLOC` / `CHECKDB` | Validates allocation structures | Corruption checks |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Uniform extents from the start (default) — no SGAM hot spot | Databases with thousands of tiny objects (64 KB minimum each) |
| More data files to spread allocation | Unequal file sizes/growth — proportional fill skews to the biggest |
| Differential backups when few extents change | High `pct_changed` — differential approaches full size |
| BULK_LOGGED for big loads | Need point-in-time restore inside a log backup containing bulk changes |

### 9. Common mistakes
- **Treating `PAGELATCH` waits as disk I/O** → buying storage → they're in-memory latch contention; identify the page type (PFS/GAM/SGAM vs data page).
- **Adding one big extra tempdb file** → proportional fill sends most allocations to it, contention remains → add equally sized files.
- **Expecting differentials to stay small forever** → they grow until the next full backup → schedule fulls based on change rate.
- **Assuming log backups in BULK_LOGGED are small because the log is** → they include bulk-changed data extents.
- **Relying on allocation-order (`NOLOCK`) scans for correctness** → rows missed or read twice during splits → use proper isolation.

### 10. 🎯 Interview kit
**30-second answer:** Each data file has fixed allocation bitmaps: GAM marks free extents, SGAM marks mixed extents with free pages, each covering about 4 GB; PFS, every 8,088 pages, stores per-page allocation status and free-space band; IAM pages map which extents in each 4 GB interval belong to an allocation unit. DCM and BCM track extents changed for differential and bulk-logged log backups. Allocations latch these pages, which is why tempdb with a single file suffers `PAGELATCH_UP` contention.

**Structure a longer answer:** layout of a data file → each map and its bit semantics → walk through an extent allocation and a heap insert → backup maps → contention and fixes → scenario: tempdb contention diagnosed via `2:1:1` waits.

**Likely follow-ups:**
- *What does GAM=0, SGAM=1 mean?* → mixed extent with free pages.
- *Why do heaps use PFS but B-trees don't for placement?* → key order decides B-tree page.
- *How does a differential backup know what to copy?* → DCM bits.
- *Why multiple tempdb files?* → more allocation map pages to spread latching.

**Signals interviewers look for:** precise bit semantics, linking structures to real symptoms (contention, backup size), and modern-version awareness (uniform extents default, 2019 PFS improvements).

### 11. Check yourself
- [ ] Draw the first 10 pages of a data file and name each.
- [ ] Explain the GAM/SGAM bit combinations.
- [ ] Walk through allocating the ninth page of a heap and record which maps change.
- [ ] Decode a `PAGELATCH_UP` wait on `2:1:1` and propose a fix.
- [ ] Explain why a BULK_LOGGED log backup can be larger than the log.
<!-- explainer:end -->

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Free Space Page › Global Allocation Map Page
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Free Space Page › Shared Global Allocation Map Page
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Extents › Page Types
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Free Space Page
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — FreeSpace Scans per Second
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Free Space Page › Bulk Changed Map Page
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Cost Allocation and Chargeback

