---
id: "TOOLS/SQL_Server/01_Architecture_and_Storage/Heaps_and_Forwarded_Records"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Heaps and Forwarded Records

**L4 Advanced** · [↑ Architecture and Storage](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A heap is a table with no clustered index — an unordered set of pages tracked only by IAM pages — and when a heap row grows too big for its page it moves and leaves a forwarding pointer behind, adding hidden I/O.

### 1. The big picture
Every table starts life as a **heap** and stays one until you create a clustered index (rowstore or columnstore). Heaps and clustered indexes are mutually exclusive: a table is one or the other. Think of a mobile library van with a few shelves: books are just numbered as they arrive, with no catalogue order. Finding one book means checking every shelf unless you keep a separate list (a nonclustered index).

Heaps are not "wrong" — they load fast and suit staging tables — but they have two distinctive behaviours you must understand: **forwarded records**, which add extra page reads after updates, and **space management** quirks after deletes (heap bloat). Both are invisible in the query text and show up only in DMVs and I/O statistics.

### 2. Core concepts and vocabulary
- **Heap** — table storage without a clustered index; rows have no defined order.
- **IAM page** — Index Allocation Map; lists the extents a heap (or index) owns within a 4 GB GAM interval. A heap is reached *only* through its IAM chain — there is no root page.
- **RID (row identifier)** — `file:page:slot`, the physical address of a heap row. Nonclustered indexes on a heap store the RID as the row locator.
- **PFS page** — Page Free Space; records roughly how full each page is (0%, 1–50%, 51–80%, 81–95%, 96–100%). Heap inserts use it to find a page with room.
- **Forwarded record** — when an updated heap row no longer fits on its page, the row moves to another page and a small **forwarding stub** (pointer) is left at the original RID.
- **Forwarded fetch** — the extra read needed to follow a forwarding stub to the real row.
- **Heap bloat** — pages that stay allocated (or half-empty) after deletes, inflating scans.
- **RID lookup** — the plan operator that fetches a heap row using a RID from a nonclustered index.

### 3. How it works, step by step
**Inserting into a heap**
1. The engine checks the PFS bytes of pages in the heap's IAM-owned extents for one with enough free space.
2. If found, the row goes there (anywhere — not at the "end"); otherwise a new page (and possibly a new extent) is allocated and recorded in the IAM.
3. The row's RID becomes its permanent address — nonclustered indexes point to it.

**Scanning a heap:** the engine reads the IAM pages and scans allocated extents in *physical file order* (an *allocation-order scan*). No linked list between heap pages exists.

**How forwarding happens** (the books' example): page 100 holds four rows of about 2 KB each, so it is full.
1. Row 2 is updated and grows to 2.5 KB. It no longer fits on page 100.
2. SQL Server moves the full row to a page with room, say page 101.
3. At row 2's original slot on page 100 it leaves a forwarding stub (about 9 bytes) holding the new RID. The moved row carries a back-pointer to its original location.
4. Why not just change the RID? Because every nonclustered index on the heap points at the original RID. Forwarding avoids updating all of them.
5. If the row moves again, the original stub is updated to point to the newest location — there is never a chain of more than one hop.
6. If the row shrinks enough later, it can move back home and the stub disappears.

**Reading a forwarded row**
- A RID lookup or singleton seek goes to page 100, finds the stub, and does a *forwarded fetch* to page 101: an extra logical read per forwarded row.
- A scan reads pages in allocation order; when it meets a stub, it jumps to the forwarded row immediately, and skips that row when it later reaches page 101, so it isn't returned twice. Each jump is an extra page access — and they add up.

**Deleting from a heap**
- Space freed by deletes is reusable by later inserts (PFS shows the page has room) — the myth that heap space is never reused is false.
- However, pages that become *completely* empty are not always deallocated: unless the delete takes a table lock (`TABLOCK`), or lock escalation occurs, empty pages may stay allocated to the heap. Scans still read them, so a heap that was 100 pages, then half-deleted, may still scan 100 pages.

### 4. Worked example
```sql
CREATE TABLE dbo.HeapForwardedRecords (RowId INT IDENTITY(1,1), FillerData VARCHAR(2500));
INSERT dbo.HeapForwardedRecords (FillerData)
SELECT TOP 24 REPLICATE('X', 2000) FROM sys.objects;

SELECT page_count, record_count, forwarded_record_count
FROM sys.dm_db_index_physical_stats(DB_ID(), OBJECT_ID('dbo.HeapForwardedRecords'), NULL, NULL, 'DETAILED');
-- page_count 6, record_count 24, forwarded_record_count 0  (4 rows per page)

UPDATE dbo.HeapForwardedRecords SET FillerData = REPLICATE('X', 2500) WHERE RowId % 2 = 0;

SELECT page_count, record_count, forwarded_record_count
FROM sys.dm_db_index_physical_stats(DB_ID(), OBJECT_ID('dbo.HeapForwardedRecords'), NULL, NULL, 'DETAILED');
-- more pages, and a non-zero forwarded_record_count

SET STATISTICS IO ON;
SELECT * FROM dbo.HeapForwardedRecords;              -- reads > page_count
SET STATISTICS IO OFF;

SELECT forwarded_fetch_count
FROM sys.dm_db_index_operational_stats(DB_ID(), OBJECT_ID('dbo.HeapForwardedRecords'), 0, NULL);

ALTER TABLE dbo.HeapForwardedRecords REBUILD;        -- removes forwarding and empty pages
```
How to read it:
- Before the update, six full pages hold 24 rows. After it, grown rows don't fit, so some are moved to new pages and `forwarded_record_count` becomes non-zero.
- In `record_count` (DETAILED mode) forwarded rows may be counted with their stubs; focus on `forwarded_record_count`.
- `STATISTICS IO` for the scan reports more logical reads than the heap's page count — one extra per forwarded row. In the books' larger example a full scan went from **26 to 144 reads** after every third row was widened, and range lookups through an index also needed extra reads.
- `forwarded_fetch_count` in the operational stats increases by one each time a query follows a stub.
- After `ALTER TABLE … REBUILD`, `forwarded_record_count` is 0 and empty pages are released.

### 5. Under the hood
- **IAM-only navigation.** A heap's first IAM page is recorded in `sys.system_internals_allocation_units.first_iam_page`. When the heap grows beyond one GAM interval (~4 GB), another IAM page is added, forming an IAM chain.
- **PFS granularity is coarse.** PFS tracks free space in bands, not exact bytes. An insert may check a page the PFS says is "51–80% full" and still find the row doesn't fit; the engine tries another page. Very wide rows plus coarse PFS can leave unusable gaps.
- **Stub structure.** The stub is a special record type (forwarding stub) holding the 8-byte RID of the new location; the forwarded record has a back-pointer so the engine can update the stub if the row moves again.
- **Rebuilds and nonclustered indexes.** `ALTER TABLE … REBUILD` on a heap rewrites all rows to new RIDs — and therefore must rebuild *every* nonclustered index too. On big tables, that's a heavy operation.
- **Clustered indexes don't forward.** In a B-tree, a growing row that doesn't fit causes a *page split* instead; nonclustered indexes use the clustering key, not a physical address, so nothing needs a pointer.
- **Fast inserts.** Bulk loads into an empty heap with `TABLOCK` can be minimally logged and run in parallel — a key reason heaps are popular for staging.

### 6. Performance and design implications
- **Scans** of heaps with forwarded rows read extra pages; **RID lookups** pay one extra read per forwarded row.
- **Monitor** `forwarded_record_count` (physical stats), `forwarded_fetch_count` (operational stats), and the counter `SQLServer:Access Methods\Forwarded Records/sec`, ideally as a ratio to `Batch Requests/sec`. A sustained high ratio means heaps are costing I/O.
- **Prevent forwarding:** avoid heaps for tables whose variable-length columns grow through updates; or add a clustered index.
- **Heaps shine** for: staging/ETL tables loaded in bulk and read once; temporary data where index build time exceeds query time; tables whose keys change often; tables with non-ascending keys (e.g. GUIDs) where a clustered index would split constantly. Data warehouse fast-track designs have used heaps for exactly these reasons.
- **Clustered indexes shine** for: unique ever-increasing keys; highly selective key columns; range queries; tables with heavy insert/delete churn.
- Neither "cluster everything" nor "heap everything" is a rule — test.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sys.dm_db_index_physical_stats(..., 'DETAILED')` → `forwarded_record_count` | Counts forwarded rows | Health check of heaps |
| `sys.dm_db_index_operational_stats` → `forwarded_fetch_count` | Forwarded fetches since cached | Measure runtime impact |
| `Forwarded Records/sec` (perfmon) | Rate of forwarded fetches instance-wide | Trend monitoring |
| `ALTER TABLE t REBUILD` | Rewrites heap, removes forwarding and empty pages | Maintenance (rebuilds NC indexes too) |
| `DELETE … WITH (TABLOCK)` | Lets emptied pages be deallocated | Purging heap data |
| `TRUNCATE TABLE` | Deallocates all pages | Clearing staging heaps |
| `CREATE CLUSTERED INDEX` | Converts heap to B-tree | Permanent fix for OLTP tables |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Heap: bulk-loaded staging, read-once, `TRUNCATE` between loads | Rows grow through updates (forwarding) |
| Heap: non-ascending keys where splits would hurt | Many point lookups without supporting nonclustered indexes |
| Clustered index: range queries, ordered access, steady churn | Very wide or frequently changing clustering keys |
| Rebuild heap to clean up | Large heap with many nonclustered indexes — rebuild cost multiplies |

### 9. Common mistakes
- **Leaving OLTP tables as heaps by accident** (no primary key, or PK declared nonclustered) → forwarding, RID lookups, bloat → choose a clustered key deliberately.
- **Deleting most of a heap without `TABLOCK`** → empty pages stay allocated; scans stay slow → use `TABLOCK`, `TRUNCATE`, or rebuild.
- **Assuming heap space is never reused** → unnecessary rebuilds → freed space is reused by inserts; rebuild only for empty-page bloat or forwarding.
- **Ignoring forwarded-record counters** → slowly rising I/O with no plan change → check them in routine index maintenance.
- **Rebuilding a big heap during business hours** → all nonclustered indexes rebuild too → schedule or convert to clustered.

### 10. 🎯 Interview kit
**30-second answer:** A heap is a table without a clustered index: pages in no order, found through IAM pages, rows addressed by RID. If an update makes a row too big for its page, SQL Server moves it and leaves a forwarding pointer so nonclustered indexes don't need updating; reads then take an extra hop. Deletes free space for reuse, but empty pages may stay allocated unless you use a table lock or rebuild. Heaps are great for bulk-loaded staging, poor for updated OLTP tables.

**Structure a longer answer:** define heap and IAM navigation → explain forwarding and why it exists → measure it (DMVs, counter) → fix it (clustered index, rebuild) → trade-offs: staging vs OLTP.

**Likely follow-ups:**
- *Why not just update the RID in nonclustered indexes?* → would require updating every index; forwarding is cheaper at write time.
- *Can a forwarded row be forwarded again?* → the original stub is repointed; never more than one hop.
- *Do clustered indexes have forwarded records?* → no; they split pages instead.
- *How do you remove forwarded records?* → `ALTER TABLE … REBUILD` or create a clustered index.

**Signals interviewers look for:** you know heaps are a choice (not a bug), you can quantify forwarding with DMVs, and you weigh rebuild cost.

### 11. Check yourself
- [ ] Explain how a heap scan finds its pages without a root page.
- [ ] Walk through what happens when a 2 KB row on a full page grows to 2.5 KB.
- [ ] Reproduce forwarding with the demo and compare scan reads before and after.
- [ ] Explain why emptied heap pages may stay allocated and how to release them.
- [ ] Name three workloads where a heap beats a clustered index.
<!-- explainer:end -->

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Forwarded Records
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Heap Bloat and Forwarding
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Forwarded Records per Second
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Splits › Heap
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Forwarded Fetch
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Heap Tables
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Heap Structure
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Myth 9 Every Table Should Have a HeapClustered Index › Myth 9: Every Table Should Have a Heap/Clustered Index
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Identify Heaps
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Myth 8 Deleting Form Heaps Results in Unrecoverable Space › Myth 8: Deleting Form Heaps Results in Unrecoverable Space

