---
id: "TOOLS/SQL_Server/01_Architecture_and_Storage/Pages_and_Extents/Page_Anatomy"
level: 5
type: "curriculum"
curriculum: "sql_server"
---

# Page Anatomy

**L5 Expert** · [↑ Pages and Extents](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Every 8,192-byte page is a 96-byte header, a record area that grows from the top, and a slot (offset) array that grows from the bottom — and that layout explains row limits, row order, ghost records and how SQL Server finds a row inside a page.

### 1. The big picture
The page is the smallest thing SQL Server reads, writes, caches, latches and checksums. Whether it's a data page of a heap, a leaf page of a clustered index, a non-leaf index page or a LOB page, the same three-part anatomy applies: **header**, **records**, **offset array**. Knowing the anatomy lets you reason about:

- why a row is limited to 8,060 bytes;
- why rows on a page are not necessarily stored in key order, yet seeks still binary-search them;
- what `DBCC PAGE` output, page IDs in wait resources and corruption errors mean;
- how deletes leave ghost records and free space, and when a page needs compaction.

This lesson zooms into the page; the parent lesson covers extents and the sibling lesson covers allocation maps.

### 2. Core concepts and vocabulary
- **Page number / page ID** — pages in a file are numbered from 0; page *n* lives at byte offset *n* × 8,192. A page is addressed as `file_id:page_id`.
- **Page header (96 bytes)** — metadata: page ID, type, level in the index, owning allocation unit ID, previous/next page IDs, slot count, free bytes, free-data offset, LSN of last change, ghost-record count, checksum/torn bits, flags.
- **Records (rows)** — data rows, index rows, ghost rows, forwarding stubs, LOB fragments, stored from byte 96 upward.
- **Offset array (slot array)** — 2-byte entries at the end of the page, one per slot, storing where each record starts. Slot 0 is the last 2 bytes of the page; the array grows backwards toward the records.
- **Slot** — a logical position on the page; `file:page:slot` is a heap RID.
- **Free space** — the gap between the end of the records area and the start of the offset array (plus any holes left by deleted rows).
- **Ghost record** — a deleted index row kept temporarily (marked as ghost) until the background ghost-cleanup task removes it.
- **Page LSN** — the log sequence number of the last change to the page; used by recovery and by WAL (page can't be flushed before log up to that LSN).
- **Page checksum** — computed on write, verified on read when `PAGE_VERIFY CHECKSUM` is set; mismatch → error 824.
- **Compaction** — rewriting a page's records contiguously to merge holes into one free area.

### 3. How it works, step by step
**Layout**
```
byte 0      ┌──────────────────────────────┐
            │ Page header (96 bytes)       │
byte 96     ├──────────────────────────────┤
            │ Row 0 │ Row 1 │ Row 2 │ ...  │  ← records grow downward
            │                              │
            │        free space            │
            │                              │
            │ ... │ off[2] │ off[1] │off[0]│  ← offset array grows upward
byte 8191   └──────────────────────────────┘
```
8,192 − 96 header = 8,096 bytes for records and slots. One record can be at most **8,060 bytes** (8,096 minus slot entry and internal overhead reserved per row/page), which is the in-row size limit you hit when creating tables.

**Inserting a row onto a page**
1. Check the header's free-byte count. If there isn't room for the row + a 2-byte slot, the page is full (B-tree → split; heap → choose another page).
2. If there is free space in total but not contiguous (holes from deletes), **compact** the page first.
3. Write the record at the free-data offset (the first open position after existing records).
4. Add its starting offset to the slot array.
5. For an index page, the slot array is kept in **key order**: SQL Server shifts slot entries (2 bytes each — cheap) instead of moving rows. So the physical position of a row on the page can be "random", while slot 0, 1, 2… are in key order.
6. Update header fields: slot count, free bytes, free-data offset, page LSN; mark the page dirty.

**Finding a row in an index page:** binary search over the slot array (which is in key order), comparing keys at the offsets it points to. A page with 300 rows needs about 9 comparisons.

**Deleting a row**
1. In an index (clustered or nonclustered), the row is marked as a **ghost** (record type changed, header ghost count incremented, PFS ghost bit set). This keeps rollback simple and lets key-range locks work.
2. After the transaction commits, the ghost-cleanup task (every few seconds) physically removes ghosts, freeing space. Under snapshot isolation, ghosts stay while versions may be needed.
3. In a heap, deleted rows are removed directly: the slot entry is zeroed but the slot numbers of the other rows don't change (nonclustered indexes point to them by RID), leaving holes until the page is compacted.

**Updating a row**
- Same size or smaller: in place.
- Larger with room on the page: rewritten on the page (possibly after compaction).
- Larger without room: heap → forwarded record; B-tree → page split.
- Updating a clustered-index *key* is a delete + insert (the row moves to its new key position).

**Record format (data row, FixedVar):** status bits (2 bytes) → fixed-data length (2) → fixed columns → column count (2) → null bitmap → variable column count (2) → variable offsets (2 each) → variable data. Index rows on non-leaf pages hold the key plus a 6-byte child page pointer.

### 4. Worked example
```sql
CREATE TABLE dbo.Anatomy (Id INT PRIMARY KEY, Name VARCHAR(20) NOT NULL);
INSERT dbo.Anatomy VALUES (30,'thirty'), (10,'ten'), (20,'twenty');

-- Find the data page (2019+)
SELECT allocated_page_file_id, allocated_page_page_id, page_type_desc
FROM sys.dm_db_database_page_allocations(DB_ID(), OBJECT_ID('dbo.Anatomy'), 1, NULL, 'DETAILED')
WHERE page_type_desc = 'DATA_PAGE';

-- Header fields of that page (replace 1, 352 with your file/page)
SELECT page_type_desc, slot_count, free_bytes, free_data_offset, ghost_rec_count,
       previous_page_page_id, next_page_page_id, page_lsn, is_iam_page
FROM sys.dm_db_page_info(DB_ID(), 1, 352, 'DETAILED');

-- Full dump including the offset table (test systems only)
DBCC TRACEON(3604);
DBCC PAGE ('YourDb', 1, 352, 1);   -- style 1: hex per row + OFFSET TABLE

-- Ghosts after delete
DELETE dbo.Anatomy WHERE Id = 20;
SELECT ghost_rec_count FROM sys.dm_db_page_info(DB_ID(), 1, 352, 'DETAILED');
```
What to look for:
- The header shows `slot_count = 3`, `free_bytes` ≈ 8,096 − rows − 6 bytes of slots, and `free_data_offset` just after the third row.
- In `DBCC PAGE` style 1, rows appear in the order they were *inserted* (30, 10, 20 at increasing offsets), but the **OFFSET TABLE** lists slot 0 → row 10, slot 1 → row 20, slot 2 → row 30: the slot array, not the physical position, holds the key order.
- Immediately after the delete, `ghost_rec_count` may be 1; a few seconds later it returns to 0 when ghost cleanup has run.
- `page_lsn` changes every time the page is modified.

### 5. Under the hood
- **Header field highlights:** `m_type` (1 data, 2 index, 3/4 text, 8 GAM, 9 SGAM, 10 IAM, 11 PFS, 13 boot, 15 file header, 16 DCM, 17 BCM), `m_level` (0 = leaf), `m_prevPage`/`m_nextPage` (B-tree linked list; zero for heaps), `m_slotCnt`, `m_freeCnt`, `m_freeData`, `m_lsn`, `m_ghostRecCnt`, `m_tornBits` (checksum or torn-page bits), and the allocation unit ID (`Metadata: AllocUnitId`) that ties the page to its object.
- **Why 8,060?** 8,096 bytes after the header, minus the 2-byte slot entry, minus space SQL Server reserves per row for versioning tags (14 bytes) and internal overhead. The exact limit applies to the row's in-row portion; variable columns beyond it move to row-overflow pages.
- **Torn-page detection vs checksum.** A page is written as 16 × 512-byte sectors. `TORN_PAGE_DETECTION` stores 2 bits per sector to detect partial writes; `CHECKSUM` (default since 2005) checksums the whole page and also catches bit flips. Checksums are only added when a page is next written, so after enabling, older pages remain unprotected until modified or rebuilt.
- **Page LSN and WAL.** Before the buffer manager writes a dirty page, it ensures the log is flushed at least up to the page LSN. During recovery, redo applies a log record only if its LSN is greater than the page LSN.
- **Versioning tag.** With RCSI/snapshot, modified rows carry a 14-byte tag (version pointer + transaction sequence number). Enabling versioning on a busy table can therefore grow rows and cause splits.
- **Compression changes records, not the anatomy.** A page-compressed page adds a compression information (CI) record right after the header (prefix values and dictionary), but header + records + offset array still apply.
- **Columnstore** segments are stored as LOB data on ordinary LOB pages; the page anatomy is the same, the content is a compressed blob.

### 6. Performance and design implications
- **Rows per page = (8,096) / (row size + 2)** — the core of all capacity maths. Keep rows narrow; beware rows just over 4,030 bytes (one per page).
- **Cheap reordering:** inserting into the middle of an index page only shifts 2-byte slot entries, so in-page ordering costs little; page *splits* are the expensive part.
- **Ghost cleanup lag:** heavy delete workloads (queue tables) can leave many ghosts; scans still have to skip them. Watch `ghost_record_count` in `sys.dm_db_index_physical_stats` and the `ghost cleanup` task; avoid patterns that delete and re-read the same range constantly.
- **Latching unit = page:** hot pages (last page of an IDENTITY index, a single-row counter table) cause `PAGELATCH_EX` waits regardless of disk speed.
- **Corruption detection:** keep `PAGE_VERIFY CHECKSUM`, run `DBCC CHECKDB`, and monitor `msdb.dbo.suspect_pages` and error 823/824/825.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sys.dm_db_page_info(db, file, page, 'DETAILED')` | Header fields as columns | Inspect any page (2019+) |
| `DBCC PAGE (db, file, page, style)` + `DBCC TRACEON(3604)` | Dump header, rows, offset table | Learning, deep diagnostics (test) |
| `sys.dm_db_database_page_allocations` | Find an object's pages | Locate pages to inspect |
| `sys.dm_db_index_physical_stats` (`ghost_record_count`, `avg_record_size_in_bytes`) | Ghosts and row sizes | Delete-heavy tables, sizing |
| `ALTER DATABASE … SET PAGE_VERIFY CHECKSUM` | Enable page checksums | Always |
| `msdb.dbo.suspect_pages` | Pages that failed checksum/I/O | Corruption response |
| `%%physloc%%` + `sys.fn_PhysLocFormatter` | Shows file:page:slot of each row | Seeing where rows live |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Narrow rows → many rows per page | Wide fixed columns that are mostly empty |
| `CHECKSUM` page verification | `NONE` or only torn-page detection |
| Enabling RCSI for concurrency | Forgetting the 14-byte tag can trigger splits on full pages |
| Inspecting pages with DMVs | Running `DBCC PAGE` on production at scale (heavy, undocumented) |

### 9. Common mistakes
- **Assuming rows on a page are physically sorted** → wrong conclusions from `DBCC PAGE` → the offset array provides order.
- **Expecting deleted index rows to free space immediately** → confusion about page counts → ghosts are removed asynchronously.
- **Designing rows near the 8,060 limit** → off-row overflow reads, one row per page → split wide rarely-used columns into another table.
- **Turning off checksums for "performance"** → silent corruption → negligible CPU cost; keep it on.
- **Reading `PAGELATCH` waits as disk problems** → they're in-memory page contention; look at which page and why it's hot.

### 10. 🎯 Interview kit
**30-second answer:** A SQL Server page is 8,192 bytes: a 96-byte header with metadata such as page type, owning allocation unit, previous/next page, free space, slot count and the last LSN; then records written from the top; and a 2-byte-per-row offset array written from the bottom. For indexes the offset array is kept in key order, so rows can sit anywhere physically while seeks binary-search the slots. A row can use at most 8,060 bytes in-row. Deleted index rows become ghosts and are cleaned up asynchronously.

**Structure a longer answer:** three parts of a page → header fields and why they matter (LSN, prev/next, checksum) → insert/delete/update mechanics within a page → limits (8,060) → tools to inspect → scenario: diagnosing error 824 or a hot `PAGELATCH_EX` page.

**Likely follow-ups:**
- *How does SQL Server find a row in a page?* → binary search through the key-ordered offset array.
- *What is a ghost record?* → logically deleted index row awaiting cleanup.
- *What does page LSN do?* → enforces WAL and makes redo idempotent.
- *Why 8,060 bytes?* → header, slot entry and reserved overhead.

**Signals interviewers look for:** exact layout knowledge, linking header fields to recovery, linked lists and corruption detection, and practical inspection skills.

### 11. Check yourself
- [ ] Sketch a page with header, three records and the offset array, labelling byte offsets.
- [ ] Insert rows 30, 10, 20 and show with `DBCC PAGE` that physical and slot order differ.
- [ ] Explain what happens on the page when a row is deleted from a clustered index.
- [ ] Read `slot_count`, `free_bytes` and `page_lsn` for a page with `sys.dm_db_page_info`.
- [ ] Compute rows per page for a 250-byte row.
<!-- explainer:end -->

## In this section (study in order)

1. [Inspecting Pages with DBCC](<Inspecting_Pages_with_DBCC/_index.md>) (L6 Internals)

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Hex Rows Print Option
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Hex Data Print Option
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Row Data Print Option
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Header Only Print Option
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Choosing between TOP, OFFSET-FETCH, and ROWNUMBER()
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Pagination with ROWNUMBER()
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Pagination with OFFSET-FETCH
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Pages
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Pagination: TOP, OFFSET-FETCH, ROWNUMBER
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Extents › File Header Page

