---
id: "TOOLS/SQL_Server/01_Architecture_and_Storage/Pages_and_Extents/Page_Anatomy/Inspecting_Pages_with_DBCC"
level: 6
type: "curriculum"
curriculum: "sql_server"
---

# Inspecting Pages with DBCC

**L6 Internals** · [↑ Page Anatomy](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A small toolkit — `DBCC EXTENTINFO`, `DBCC IND` / `sys.dm_db_database_page_allocations`, `DBCC PAGE` / `sys.dm_db_page_info` and `sys.dm_os_buffer_descriptors` — lets you list which pages and extents an object owns, open any single page and read its header, rows and slot array, and see which of those pages are sitting in memory.

### 1. The big picture
The earlier lessons described pages, extents, allocation maps (GAM, SGAM, PFS, IAM) and the anatomy of a single page in theory. This lesson gives you the instruments to *verify* that theory on a real database. Once you can list an object's pages and open one, you can answer questions that otherwise stay abstract:

- Is this table a heap or a B-tree, how many levels does the index have, and which page is the root?
- Did that insert pattern cause page splits? Are pages half empty?
- How big is a row *really* on disk (null bitmap, variable-length offsets, versioning tag, forwarding stub)?
- A wait resource or error message says `PAGE: 7:1:4512` — what object is that and what is on it?
- Which tables are occupying the buffer pool right now?

Two cautions frame everything. First, the classic commands (`DBCC IND`, `DBCC PAGE`, `DBCC EXTENTINFO`) are **undocumented and unsupported**: they have existed for decades and are widely used, but their output format can change and Microsoft gives no guarantees. Second, some of them do real I/O; use them on test copies or on a handful of pages, not as a production monitoring loop. Newer, documented replacements exist for most of them (`sys.dm_db_database_page_allocations` since 2012, `sys.dm_db_page_info` since 2019), and in practice you mix the two.

### 2. Core concepts and vocabulary
- **Page ID** — `file_id:page_id`, e.g. `1:312`. Page *n* starts at byte *n* × 8,192 of the file. In waits and errors you also see `db_id:file_id:page_id`.
- **Allocation unit** — the container of pages for one partition of one index, of one type: IN_ROW_DATA, ROW_OVERFLOW_DATA or LOB_DATA. Each has its own IAM chain.
- **IAM page (Index Allocation Map)** — the page that records which extents in a 4 GB file range belong to one allocation unit. Listing pages starts from it.
- **Mixed vs uniform extent** — a mixed extent's 8 pages can belong to different objects (single-page allocations); a uniform extent belongs entirely to one allocation unit.
- **Index level** — 0 for the leaf (or for every page of a heap), rising to N at the root.
- **Page type** — a number in the header: 1 data, 2 index, 3 text-mix (LOB), 4 text-tree, 8 GAM, 9 SGAM, 10 IAM, 11 PFS, 13 boot, 15 file header, 16 DCM (differential changed map), 17 BCM (bulk changed map).
- **Print option** — the level of detail `DBCC PAGE` produces: header only, hex per row, one hex dump, or decoded column values.
- **Trace flag 3604** — redirects DBCC output to the client (Messages tab) instead of the error log.
- **Buffer descriptor** — the in-memory metadata the buffer manager keeps for each 8 KB buffer; exposed by `sys.dm_os_buffer_descriptors`.
- **Page merge / page allocation counters** — cumulative counters in `sys.dm_db_index_operational_stats` showing how an index grows and shrinks between maintenance runs.

### 3. How it works, step by step
The tools form a funnel: from *which extents*, to *which pages*, to *what is on one page*, to *what is in memory*.

| Question | Tool | Returns |
|---|---|---|
| How were extents allocated (mixed or uniform)? | `DBCC EXTENTINFO` | One row per allocation: file, first page, pages allocated, extent size |
| Which pages does this object own, and how are they linked? | `DBCC IND` or `sys.dm_db_database_page_allocations` | One row per page: type, level, prev/next page, IAM page |
| What exactly is on page X? | `DBCC PAGE` or `sys.dm_db_page_info` | Header fields; optionally rows, slot array, decoded columns |
| What is cached in memory? | `sys.dm_os_buffer_descriptors` | One row per buffered page |
| How much has the index split and merged? | `sys.dm_db_index_operational_stats` | Cumulative allocation and merge counts |

**Step 1 — `DBCC EXTENTINFO (db, table, index, partition)`.** Parameters: database name/ID (0 = current), table name/ID, index name/ID (-1 = all), partition number (0 = all). It returns one row per *allocation*, with `file_id`, `page_id` (first page), `pg_alloc` (pages used from that extent), `ext_size` (1 for a single-page allocation, 8 for a full extent), `object_id`, `index_id`, `partition_id`, `iam_chain_type` (in-row, LOB, overflow) and `pfs_bytes` (the PFS status bytes for those pages). A mixed-extent allocation shows up as `ext_size = 1, pg_alloc = 1`; a uniform extent as `ext_size = 8` with `pg_alloc` between 1 and 8.

The classic demonstration: create a heap whose rows are about 8 KB each (`CHAR(8000)`), so every insert needs a new page. With mixed allocation in effect, the first eight pages arrive one at a time from mixed extents (eight rows with `ext_size = 1`). From the ninth page on, allocations come as uniform extents (a row with `ext_size = 8`, `pg_alloc = 2` after ten pages). Earlier single-page allocations are *not* moved; a `ALTER TABLE … REBUILD` (or index rebuild) rewrites the object into uniform extents.

> **Version note.** Since SQL Server 2016, user databases default to `MIXED_PAGE_ALLOCATION OFF`: even the first page of a new table gets a uniform extent. You will only reproduce the "first eight pages mixed" behavior in `tempdb`/system databases on older versions, with trace flag 1118 off on ≤2014, or after `ALTER DATABASE … SET MIXED_PAGE_ALLOCATION ON`. Check with `SELECT name, is_mixed_page_allocation_on FROM sys.databases`.

**Step 2 — `DBCC IND (db, table, index)`.** Same first three parameters. One row per page that holds data for the object, *including the IAM page* (which EXTENTINFO omits). Key columns: `PageFID`/`PagePID`, `IAMFID`/`IAMPID` (which IAM page tracks it; NULL for the IAM page itself), `ObjectID`, `IndexID`, `PartitionNumber`, `PartitionID`, `iam_chain_type`, `PageType`, `IndexLevel`, and `NextPageFID/PID`, `PrevPageFID/PID`.

How to read it:
- A **heap** shows PageType 10 (IAM) plus PageType 1 pages, all at IndexLevel 0, with next/prev = 0 — heap pages are not linked to each other.
- A **clustered index** with a few full rows shows PageType 1 leaf pages at level 0 linked through next/prev, plus one PageType 2 page at level 1 — the root. The root is the page with the highest IndexLevel.
- More rows → more levels; the number of distinct IndexLevel values is the index depth.

**Step 2b — `sys.dm_db_database_page_allocations (db_id, object_id, index_id, partition_id, mode)`.** The documented-ish replacement (it is still marked undocumented, but it is a regular function you can query). All five parameters are required; NULL means "all" for object/index/partition. Mode `LIMITED` returns allocation metadata only; `DETAILED` additionally reads the pages to return type, level, linkage and free space. Advantages over `DBCC IND`:
1. It's a rowset: filter, join to `sys.indexes`/`sys.objects`, aggregate.
2. It reports *allocated but empty* pages too (`is_allocated`), while DBCC IND lists only pages with data.
3. Extra columns: `is_mixed_page_allocation`, `page_free_space_percent`, `page_type_desc`, `is_page_compressed`, `has_ghost_records`, `extent_page_id`.

Column mapping: `allocated_page_file_id`/`allocated_page_page_id` ↔ PageFID/PagePID; `allocated_page_iam_file_id`/`allocated_page_iam_page_id` ↔ IAMFID/IAMPID; `page_type`, `page_level`, `next_page_*`, `previous_page_*` ↔ the corresponding DBCC IND columns.

**Step 3 — `DBCC PAGE (db, file, page, print_option)`.** Enable `DBCC TRACEON(3604)` first, or the output goes to the error log. Print options:

| Option | Output |
|---|---|
| 0 (default) | Buffer info + page header only |
| 1 | Header + each record as a separate hex dump + the offset (slot) array |
| 2 | Header + the whole page as one hex dump + offset array |
| 3 | Header + each record, with every column decoded by name and value (best for learning) |

The header section is where most diagnostic value lies:
- `m_pageId`, `m_type`, `m_level`, `m_indexId` (internal), `m_prevPage` / `m_nextPage`
- `m_slotCnt` (rows/slots), `m_freeCnt` (free bytes), `m_freeData` (offset where free space begins), `m_ghostRecCnt`
- `m_lsn` (last change), `m_tornBits` (checksum or torn-page bits)
- `Metadata: AllocUnitId`, `Metadata: PartitionId`, `Metadata: ObjectId`, `Metadata: IndexId` — which object owns the page
- allocation status lines: is the page allocated in GAM/SGAM, PFS fullness band (`0_PCT_FULL`, `50_PCT_FULL`, `80_PCT_FULL`…), and DIFF/ML (changed since last full backup / minimally logged)

With option 1 or 3 each record also shows `Record Type` (PRIMARY_RECORD, INDEX_RECORD, GHOST_DATA_RECORD, FORWARDING_STUB, FORWARDED_RECORD), `Record Attributes` (NULL_BITMAP, VARIABLE_COLUMNS, VERSIONING_INFO) and `Record Size` — the true byte size, including overhead. Option 3 on an index page shows child page IDs and key values; that is how you walk a B-tree by hand from root to leaf.

**Step 3b — `sys.dm_db_page_info (db_id, file_id, page_id, mode)`** (SQL Server 2019+). A documented function returning the header fields as columns (`page_type_desc`, `slot_count`, `free_bytes`, `ghost_rec_count`, `object_id`, `index_id`, `page_lsn`, …). Pair it with `sys.fn_PageResCracker(page_resource)` to decode the `page_resource` column in `sys.dm_exec_requests`, so you can jump from a waiting session straight to the page and its owning table.

**Step 4 — `sys.dm_os_buffer_descriptors`.** One row per page currently in the buffer pool: `database_id`, `file_id`, `page_id`, `page_type`, `allocation_unit_id`, `row_count`, `free_space_in_bytes`, `is_modified` (dirty). Grouping by database tells you how memory is split; joining `allocation_unit_id` to `sys.allocation_units` → `sys.partitions` tells you which tables and indexes are cached. A practical lesson from the books: unindexed logging tables or unpruned `msdb` backup history can occupy a large share of memory and push out the data the business actually needs.

**Step 5 — allocation cycle counters.** `sys.dm_db_index_operational_stats` exposes `leaf_allocation_count`, `nonleaf_allocation_count`, `leaf_page_merge_count`, `nonleaf_page_merge_count`. They describe how the index "breathes": pages added by inserts and splits, pages removed by merges after deletes. Note that a freshly loaded table already shows hundreds of leaf allocations from plain inserts — allocation ≠ bad split. Interpret them together with `sys.dm_db_index_physical_stats` and the page-split counters covered in the page-split lesson.

### 4. Worked example
```sql
-- Setup: a small clustered table with fat rows so pages fill quickly
CREATE DATABASE PageLab;
GO
USE PageLab;
CREATE TABLE dbo.Fat
( RowID  INT IDENTITY(1,1) CONSTRAINT PK_Fat PRIMARY KEY CLUSTERED,
  Filler CHAR(2000) NOT NULL DEFAULT ('x'),
  Note   VARCHAR(100) NULL );
INSERT dbo.Fat (Note) SELECT TOP (40) 'row ' + CAST(ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS varchar(10))
FROM sys.all_objects;

-- 1. Which pages does it own? (documented-ish rowset)
SELECT allocated_page_file_id AS f, allocated_page_page_id AS p,
       page_type_desc, page_level, previous_page_page_id AS prev,
       next_page_page_id AS nxt, page_free_space_percent, is_mixed_page_allocation
FROM sys.dm_db_database_page_allocations(DB_ID(), OBJECT_ID('dbo.Fat'), 1, NULL, 'DETAILED')
WHERE is_allocated = 1
ORDER BY page_level DESC, p;

-- 2. Open the root (the page with the highest page_level), decoded
DBCC TRACEON(3604);
DBCC PAGE ('PageLab', 1, 328, 3);     -- replace 328 with your root page id

-- 3. Open the first leaf page and look at one record
DBCC PAGE ('PageLab', 1, 320, 3);     -- replace with a leaf page id

-- 4. Same header, documented way (2019+)
SELECT page_type_desc, slot_count, free_bytes, ghost_rec_count, page_lsn
FROM sys.dm_db_page_info(DB_ID(), 1, 320, 'DETAILED');

-- 5. What is in cache for this database?
SELECT o.name, i.name AS index_name, COUNT(*) AS cached_pages,
       SUM(CASE WHEN bd.is_modified = 1 THEN 1 ELSE 0 END) AS dirty_pages
FROM sys.dm_os_buffer_descriptors bd
JOIN sys.allocation_units au ON au.allocation_unit_id = bd.allocation_unit_id
JOIN sys.partitions pa ON pa.hobt_id = au.container_id AND au.type IN (1,3)
JOIN sys.objects o ON o.object_id = pa.object_id
JOIN sys.indexes i ON i.object_id = pa.object_id AND i.index_id = pa.index_id
WHERE bd.database_id = DB_ID() AND o.is_ms_shipped = 0
GROUP BY o.name, i.name;
```
**Reading the results.**
- Query 1: each row is ~2,015 bytes on disk (2,000 fixed + 4 INT + small header, null bitmap, variable-column offset and the VARCHAR), so 4 rows fit per page → 10 leaf pages at `page_level 0`, linked by prev/next in key order, plus one `INDEX_PAGE` at `page_level 1`: the root. You'll also see the `IAM_PAGE`. On SQL Server 2016+ `is_mixed_page_allocation` is 0 everywhere.
- Query 2 (root, option 3): output is a small grid with one row per child — `ChildPageId` and the low key `RowID (key)` for each leaf page. The first entry's key is NULL: the leftmost pointer covers "everything below the next key". This is the navigation the seek follows.
- Query 3 (leaf, option 3): header shows `m_type = 1`, `m_level = 0`, `m_slotCnt = 4`, `m_freeCnt` ≈ 100 bytes. Each slot prints `Record Type = PRIMARY_RECORD`, `Record Attributes = NULL_BITMAP VARIABLE_COLUMNS`, `Record Size = 2019` (or similar), then `RowID = 1`, `Filler = x…`, `Note = row 1`. At the bottom, the `OFFSET TABLE` lists slots and their byte offsets: slot 0 at offset 96 (right after the header).
- Now `DELETE dbo.Fat WHERE RowID = 2;` and immediately re-run query 3: `m_ghostRecCnt = 1` and slot 1 shows `GHOST_DATA_RECORD` until the ghost-cleanup task runs (a few seconds later the slot disappears).
- Query 5 shows that all 12 pages are cached and, until the next checkpoint, dirty.

### 5. Under the hood
**Where DBCC PAGE reads from.** `DBCC PAGE` asks the buffer manager for the page — if it is not in memory it is read from disk into the buffer pool (the "BUFFER:" section of option 0 shows the `BUF` address and whether it is dirty). So the output reflects the latest in-memory state, including changes not yet checkpointed. It takes only a short latch, but on a busy page it still competes like any reader.

**How the page list is produced.** Both `DBCC IND` and the DMF start at the allocation unit's first IAM page (stored in system metadata — `sys.system_internals_allocation_units.first_iam_page`), walk the IAM chain, and for every extent bit set in each IAM page enumerate its pages. In `LIMITED` mode the function stops there (allocation info only, cheap). `DETAILED` additionally reads each page header to report type, level and links, which costs one page read per page — on a 100 GB table that's 13 million reads. Always pass the object and index IDs.

**Mapping a page back to an object.** The page header contains the `AllocUnitId`. Join it to `sys.allocation_units` (`allocation_unit_id`), then `container_id` → `sys.partitions.hobt_id` (for in-row and row-overflow units) or `partition_id` (for LOB units) → `object_id`, `index_id`. This is how `sys.dm_db_page_info`, wait-resource decoding and corruption triage all identify the owner.

**What the offset table shows.** Slot entries are stored from the end of the page backwards; DBCC prints them in slot order. In a B-tree page, slot order is logical key order even though physical offsets can be anywhere — after an update moves a variable-length row within the page, its offset changes but its slot does not. That's why seeks can binary-search a page via the slot array.

**Version notes.**
- 2012: `sys.dm_db_database_page_allocations` arrives (undocumented but stable).
- 2016: mixed extents off by default for user databases; tempdb also uniform by default (replacing TF 1118).
- 2019: `sys.dm_db_page_info` and `sys.fn_PageResCracker` — the documented path for header inspection and for wait resources.

### 6. Performance and design implications
- Use these tools to confirm row size and rows-per-page assumptions: a column you thought was 20 bytes may be stored with 14 bytes of versioning tag (row versioning on), or the row may have a forwarding stub. Rows per page drive pages read, cache footprint and I/O.
- `page_free_space_percent` from the DMF, averaged per index, is a quick density check on small objects; for big ones prefer `sys.dm_db_index_physical_stats` in `SAMPLED` mode.
- Buffer-pool composition is an indexing signal: a large table that is mostly scanned will dominate the cache; a narrower index might let the same queries run with a fraction of the memory.
- Cost: `LIMITED` mode and `DBCC EXTENTINFO` read only allocation pages. `DETAILED` mode and `DBCC PAGE` on uncached pages cause physical reads. `sys.dm_os_buffer_descriptors` walks every buffer descriptor — on a server with 1 TB of RAM that's ~130 million rows, so run it rarely and filter by `database_id`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `DBCC TRACEON(3604)` | Sends DBCC output to the client session | Before any `DBCC PAGE` |
| `DBCC EXTENTINFO(0, 'tbl', -1)` | Lists extent allocations (mixed vs uniform) | Studying allocation patterns, excessive extents |
| `DBCC IND(0, 'tbl', -1)` | Lists pages with data, incl. IAM, level, links | Quick page list on older versions |
| `sys.dm_db_database_page_allocations(db, obj, idx, part, 'LIMITED'/'DETAILED')` | Page list as a rowset | Filtering/joining; empty pages; free space per page |
| `DBCC PAGE(db, file, page, 0/1/2/3)` | Dumps a page | Header, record sizes, ghosts, walking a B-tree |
| `sys.dm_db_page_info(db, file, page, 'DETAILED')` | Documented page header | 2019+, production-friendly lookups |
| `sys.fn_PageResCracker(page_resource)` | Decodes the binary page resource of a request | Turning PAGELATCH waits into objects |
| `sys.dm_os_buffer_descriptors` | One row per cached page | Memory usage by database / table |
| `sys.dm_db_index_operational_stats` (allocation & merge columns) | Cumulative page allocations/merges | Seeing how an index grows between maintenance |
| `ALTER DATABASE … SET MIXED_PAGE_ALLOCATION ON/OFF` | Mixed-extent behavior for a database | Reproducing classic allocation demos |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `DBCC PAGE` option 3 to learn row formats and walk B-trees | Production automation — undocumented, output format may change |
| `sys.dm_db_page_info` on 2019+ for header questions | You need row contents — it returns only header fields |
| DMF `LIMITED` for allocation counts on large objects | You need type/level/links — those require `DETAILED` (reads every page) |
| `DBCC IND` on old versions or for familiarity | You want empty-but-allocated pages or joins — use the DMF |
| `sys.dm_os_buffer_descriptors` to diagnose cache pollution | Huge-memory servers at peak time; aggregate once, store results |

### 9. Common mistakes
- Running `DBCC PAGE` without trace flag 3604 → "DBCC execution completed" and no output (it went to the error log) → `DBCC TRACEON(3604)` first, or use `WITH TABLERESULTS`.
- Calling the DMF with `NULL, NULL, NULL, NULL, 'DETAILED'` on a big database → reads every page, long run, I/O spike → always pass object and index IDs; prefer `LIMITED`.
- Expecting mixed extents in a demo on 2016+ → results look "wrong" → check `is_mixed_page_allocation_on`; the default changed.
- Assuming heap pages are linked → next/prev are 0 in a heap; only B-tree levels form doubly linked lists.
- Reading PageType 2 as "data" → 2 is an index (non-leaf or nonclustered leaf) page; data pages are type 1.
- Forgetting that pages can be dirty → `DBCC PAGE` shows the in-memory version, which may differ from disk until the next checkpoint.
- Using the decoded `m_objId` / internal `m_indexId` from the header directly as object_id → use the `Metadata: ObjectId` line or map via `AllocUnitId`.

### 10. 🎯 Interview kit
**30-second answer:** "To see how a table is physically stored I list its pages with `sys.dm_db_database_page_allocations` (or the older `DBCC IND`) — that shows page types, index levels, the root and the leaf chain. Then I open a page with `DBCC PAGE` and trace flag 3604, option 3, which shows the header — slot count, free bytes, ghost count, LSN, owning object — and every row decoded with its true size. On 2019+ `sys.dm_db_page_info` gives the header in a documented way and decodes page wait resources. To see what's in memory I aggregate `sys.dm_os_buffer_descriptors`."

**Structure a longer answer:** clarify why (learning internals, latch contention, corruption, memory usage) → describe the funnel extents → pages → page contents → cache → mention documented vs undocumented tools and cost (`LIMITED` vs `DETAILED`) → close with a scenario such as decoding `PAGELATCH_EX` on `2:1:1` (tempdb PFS) or finding which table dominates the buffer pool.

**Likely follow-ups:**
- *How do you find the root page of an index?* → highest `page_level` in the page list; or `root_page` in `sys.system_internals_allocation_units`.
- *What does print option 3 add?* → decoded column values per record, record type and size.
- *Why might DBCC IND show fewer pages than the DMF?* → it only lists pages with data; the DMF also reports allocated empty pages.
- *A session waits on `PAGELATCH_EX` on 2:1:3. What is it?* → tempdb (db 2), file 1, page 3 = PFS page → allocation contention, add tempdb files / version-specific fixes.
- *What changed in 2016 around extents?* → uniform extents by default; TF 1118 no longer needed.

**Signals interviewers look for:** knows the tools are undocumented and when a documented alternative exists; understands cost of `DETAILED`; can map a page ID to an object; links page-level evidence to real problems (row size, splits, latch contention, cache pollution).

### 11. Check yourself
- [ ] List the pages of a clustered index, identify the root and count the levels.
- [ ] Open a leaf page with `DBCC PAGE … 3` and explain `m_slotCnt`, `m_freeCnt`, `m_ghostRecCnt` and `Record Size`.
- [ ] Delete a row and catch its ghost record before cleanup.
- [ ] Explain why `DBCC IND` and the DMF can return different counts, and when `LIMITED` is enough.
- [ ] Decode a `page_resource` from `sys.dm_exec_requests` to a table name.
- [ ] Write the query that shows how many MB each database occupies in the buffer pool.
<!-- explainer:end -->

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — DBCC IND
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — DBCC EXTENTINFO
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — sys.dmdbdatabasepageallocations
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — DBCC PAGE
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — � P
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Allocation Cycle
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Columnstore Structure › Examining Pages
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Buffer Allocation
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Optimization › 10.1 #54 Turning on TF1117 and TF1118
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Free Space Page › Differential Changed Map Page

