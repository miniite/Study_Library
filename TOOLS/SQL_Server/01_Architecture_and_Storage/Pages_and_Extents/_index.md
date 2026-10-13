---
id: "TOOLS/SQL_Server/01_Architecture_and_Storage/Pages_and_Extents"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Pages and Extents

**L4 Advanced** · [↑ Architecture and Storage](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** SQL Server stores everything in 8 KB pages, groups them into 64 KB extents of eight contiguous pages, and allocates space to tables and indexes extent by extent.

### 1. The big picture
Pages and extents are the physical vocabulary of SQL Server. A data file is just a long array of 8 KB pages numbered from 0. Every row, index entry, allocation bitmap and LOB fragment lives on some page; every read and write to a data file moves whole pages; and the buffer pool caches whole pages. "Logical reads" in `STATISTICS IO` is a page count. Fragmentation, fill factor, page splits, compression and allocation contention are all about how rows are packed into pages and how pages are packed into extents.

This lesson is the parent of two deeper ones: **Page Anatomy** (what's inside a page) and **Allocation Maps** (how the engine tracks which pages and extents are used).

### 2. Core concepts and vocabulary
- **Page** — 8,192 bytes: a 96-byte header, row data, and a slot array at the end. Up to 8,060 bytes are available for one in-row record.
- **Page ID** — `(file_id : page_number)`, e.g. `1:312`. Page *n* starts at byte offset *n* × 8,192 of the file.
- **Extent** — eight physically contiguous pages (64 KB). The unit of space allocation.
- **Mixed extent** — an extent whose pages may belong to up to eight different objects.
- **Uniform extent** — an extent whose eight pages all belong to one object (one allocation unit).
- **Allocation unit** — the set of pages of one type for one partition: `IN_ROW_DATA`, `ROW_OVERFLOW_DATA` or `LOB_DATA`.
- **Data page** — holds table rows (heap pages or clustered-index leaf pages).
- **Index page** — non-leaf B-tree pages, and nonclustered index leaf pages.
- **Text/Image (LOB) page** — holds large values (`VARCHAR(MAX)`, `VARBINARY(MAX)`, XML) and row-overflow data.
- **Allocation pages** — GAM, SGAM, PFS, IAM, DCM, BCM: bitmaps that track space (covered in *Allocation Maps*).

### 3. How it works, step by step
**Page types you'll meet**

| Type | Holds |
|---|---|
| 1 Data | Rows of heaps and clustered index leaves |
| 2 Index | Non-leaf pages; nonclustered index leaf pages |
| 3 Text mix / 4 Text tree | LOB and row-overflow data |
| 8 GAM / 9 SGAM | Extent allocation bitmaps |
| 10 IAM | Extents belonging to one allocation unit |
| 11 PFS | Page free space and allocation status |
| 16 DCM / 17 BCM | Changed extents since last full backup / minimally logged extents |

**Allocating space to a new table** (classic behaviour):
1. A new table has no pages. The first insert needs one data page and one IAM page.
2. While the object is small (under eight pages), pages are taken one at a time from **mixed extents** shared with other small objects — saving space for the many tiny tables in a typical database.
3. Once the object reaches eight pages, further space is allocated as **uniform extents** — whole 64 KB extents owned by this object. Its pages are now contiguous, so a scan can read an extent in a single larger I/O.
4. The IAM page records which extents the object owns.

**Modern default:** since SQL Server 2016, user databases and tempdb allocate **uniform extents from the first page** (`MIXED_PAGE_ALLOCATION OFF`; trace flag 1118 behaviour is the default). This avoids contention on SGAM pages, at the cost of up to 64 KB per tiny object. You can turn mixed allocation back on per database; `master`, `model` and `msdb` keep it on.

**Row placement:** rows must fit in-row (≤ 8,060 bytes). Variable-length values that push a row over the limit move to **row-overflow** pages, leaving a 24-byte pointer in-row. `MAX` types store large values in **LOB** pages.

**Why pages matter for columnstore:** a rowstore page holds whole rows, so aggregating one column over 40 rows of a 20-column table reads all 20 columns' bytes. A columnstore segment stores one column's values together, so the same aggregate touches far fewer pages — the reason columnstore wins for analytics.

### 4. Worked example
```sql
CREATE TABLE dbo.PageDemo (Id INT IDENTITY PRIMARY KEY, Payload CHAR(2000) NOT NULL DEFAULT 'x');
INSERT dbo.PageDemo DEFAULT VALUES;  -- 1 row
GO
-- Pages and extents used
SELECT allocation_unit_type_desc, page_type_desc, allocated_page_file_id AS file_id,
       allocated_page_page_id AS page_id, extent_page_id, is_mixed_page_allocation
FROM sys.dm_db_database_page_allocations(DB_ID(), OBJECT_ID('dbo.PageDemo'), NULL, NULL, 'DETAILED');

INSERT dbo.PageDemo DEFAULT VALUES;
GO 100
-- Rows per page: 2,000-byte rows → about 4 per page
SELECT index_level, page_count, record_count, avg_page_space_used_in_percent
FROM sys.dm_db_index_physical_stats(DB_ID(), OBJECT_ID('dbo.PageDemo'), 1, NULL, 'DETAILED');

-- Reserved vs used space
EXEC sp_spaceused 'dbo.PageDemo';
```
What to notice:
- After one row you see one `IAM_PAGE` and one `DATA_PAGE`. `is_mixed_page_allocation` is 0 in a modern user database, and `extent_page_id` shows the data page sits in an extent reserved entirely for this table.
- With 2,000-byte rows, four fit per page (4 × ~2,011 bytes ≈ 8,044) — page_count ≈ 26 for 101 rows; `avg_page_space_used_in_percent` is high.
- `sp_spaceused` shows *reserved* (whole extents, multiples of 64 KB) larger than *data* — allocated but not-yet-used pages in owned extents.

### 5. Under the hood
- **Fixed geometry.** Page *n* of file *f* is at offset `n × 8192`. Extents start at page numbers divisible by 8. Page 0 of each file is the file header; page 1 is the first PFS; page 2 the first GAM; page 3 the first SGAM; pages 6 and 7 are DCM and BCM. In the primary file, page 9 is the boot page.
- **Torn-page protection.** A page is 16 disk sectors of 512 bytes. `PAGE_VERIFY CHECKSUM` (default) stores a checksum in the header on write and verifies it on read, catching I/O corruption (error 824).
- **I/O sizes.** Single-page reads are 8 KB; read-ahead and scans read up to whole extents and beyond (up to 512 KB–several MB in Enterprise); checkpoint writes coalesce adjacent dirty pages.
- **Allocation units per partition.** Each index partition can own up to three allocation units, each with its own IAM chain: in-row, row-overflow, LOB.
- **Space reuse.** Deleted rows leave space on the page (ghost records are cleaned up later). Empty pages are deallocated; empty extents return to the free pool (GAM bit set).

### 6. Performance and design implications
- **Row width drives pages.** Rows per page = ~8,060 / row size. A 4,100-byte row fits once per page, wasting nearly half of it. Design widths deliberately and prefer appropriate data types.
- **Logical reads are pages.** Halving row size roughly halves pages scanned and memory used.
- **Contiguity** helps large scans (fewer, larger I/Os). Uniform extents give each object contiguous 64 KB chunks.
- **Allocation contention** (`PAGELATCH_UP` on PFS/GAM/SGAM pages) shows up in tempdb under heavy create/drop of temp objects; fix with multiple equally sized tempdb files.
- **Columnstore** sidesteps row-per-page limits for analytic queries by storing columns separately.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sys.dm_db_database_page_allocations` | Lists an object's pages, types, extents | Learning, diagnostics (undocumented) |
| `sys.dm_db_page_info` (2019+) | Header info for one page | Resolving a page ID from a wait resource |
| `sys.dm_db_index_physical_stats` | Pages, records, space used per level | Density and fragmentation |
| `sp_spaceused` | Reserved vs data vs index vs unused | Capacity checks |
| `ALTER DATABASE … SET MIXED_PAGE_ALLOCATION ON/OFF` | Mixed-extent behaviour | Rarely changed |
| `ALTER DATABASE … SET PAGE_VERIFY CHECKSUM` | Page corruption detection | Always on |
| `DBCC PAGE` (with TF 3604) | Dumps a page | Deep internals on test systems |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Uniform extents from the start (default) — less SGAM contention | Thousands of tiny tables where 64 KB each adds up |
| Narrow rows → more rows per page | Rows just over half a page (≈4,030 bytes) — one row per page |
| `MAX` types for genuinely large values | Using `MAX` "just in case" — off-row LOB pages, no index keys |
| Columnstore for scans over few columns | OLTP singleton access patterns |

### 9. Common mistakes
- **Thinking in rows instead of pages** → misjudging query cost → read `logical reads` and page counts.
- **Very wide fixed-length columns (`CHAR(2000)`) for short data** → few rows per page, bloated cache → use `VARCHAR` sized to the data.
- **Designing rows just over 4,030 bytes** → only one row fits per page → split rarely used wide columns into another table or use LOB storage.
- **Turning off `PAGE_VERIFY CHECKSUM`** → silent corruption goes undetected → keep it on and run `DBCC CHECKDB`.

### 10. 🎯 Interview kit
**30-second answer:** SQL Server's unit of storage and I/O is the 8 KB page — 96-byte header, rows, and a slot array, with up to 8,060 bytes per row. Eight contiguous pages form a 64 KB extent, the unit of allocation. Extents are uniform (one object) or mixed (shared by small objects); since 2016 user databases use uniform extents from the start. Allocation bitmaps (GAM, SGAM, PFS, IAM) track which extents and pages are in use.

**Structure a longer answer:** page structure → page types → extents and allocation → row-overflow and LOB → implications: row width, logical reads, tempdb contention.

**Likely follow-ups:**
- *Why 8,060 bytes and not 8,192?* → header (96 bytes) and slot array/overhead.
- *What happens if a row exceeds 8,060 bytes?* → variable columns move to row-overflow pages with a 24-byte pointer.
- *Why did mixed extents cause problems?* → SGAM contention in tempdb.
- *What's a logical read?* → one page read from the buffer pool.

**Signals interviewers look for:** you translate rows into pages, know allocation basics, and connect them to performance (row width, tempdb).

### 11. Check yourself
- [ ] Compute how many 600-byte rows fit on one page.
- [ ] Explain mixed vs uniform extents and what changed in SQL Server 2016.
- [ ] List the page types in a database and what each holds.
- [ ] Use `sys.dm_db_database_page_allocations` to find a table's IAM and data pages.
- [ ] Explain why `sp_spaceused` reports reserved space larger than data.
<!-- explainer:end -->

## In this section (study in order)

1. [Page Anatomy](<Page_Anatomy/_index.md>) (L5 Expert)
2. [Allocation Maps (GAM SGAM PFS IAM)](<Allocation_Maps/_index.md>) (L5 Expert)

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Extents
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Index Allocation Map Page › Data Page
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Index arChIteCture R HTE TURE › Physical Data on Pages
- *SQL Server Integration Services Design Patterns* — Tim Mitchell et al. — SQL Server Integration Services Design Patterns › Contents

