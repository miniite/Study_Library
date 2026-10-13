---
id: "TOOLS/SQL_Server/02_Indexing/Columnstore/Columnstore_Architecture"
level: 5
type: "curriculum"
curriculum: "sql_server"
---

# Columnstore Architecture

**L5 Expert** · [↑ Columnstore Indexes](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A columnstore is a set of read-only compressed rowgroups (one segment per column), plus two writable side structures: the delta store for new rows and the delete bitmap for removed rows. Encodings, dictionaries and the tuple mover tie them together.

### 1. The big picture
The parent lesson showed the shape: rowgroups of up to 1,048,576 rows, each split into per-column segments. This lesson opens the box. Compressed segments are **immutable**: SQL Server never edits a compressed segment in place, because recompressing a million-row segment for every single-row change would be ruinous. Every write therefore goes somewhere else. **Inserts** go to the delta store, **deletes** go to the delete bitmap, and **updates** use both. Later, background processes and maintenance fold those side structures back into clean compressed rowgroups.

Understanding this design explains almost every columnstore behaviour you will meet in practice. It explains why trickle inserts produce small rowgroups, why deleted rows still cost reads, why updates are slow, and why `REORGANIZE` matters. The other child lessons (elimination, loading/maintenance, batch mode) all build on this one.

### 2. Core concepts and vocabulary
- **Compressed rowgroup**: a rowgroup whose columns have been encoded and compressed into segments. It is read-only.
- **Segment**: one column of one compressed rowgroup. It is stored as a LOB chain of 8 KB pages and carries metadata: row count, size, encoding, min/max data IDs, null flag.
- **Delta rowgroup / delta store**: an uncompressed rowgroup stored as an ordinary clustered B-tree (rowstore). New rows land here when they are not bulk-loaded.
- **Rowgroup states**: `INVISIBLE` (being compressed, not yet visible), `OPEN` (delta rowgroup accepting rows), `CLOSED` (delta rowgroup full, waiting for compression), `COMPRESSED`, `TOMBSTONE` (an old delta rowgroup whose rows have been compressed, waiting for garbage collection).
- **Tuple mover**: a background task (by default it checks about every 5 minutes) that compresses `CLOSED` delta rowgroups. From SQL Server 2019 it also runs a background *merge* that combines small compressed rowgroups and purges deleted rows.
- **Delete bitmap**: for a CCI, a hidden B-tree listing the row locators (rowgroup id + row position) of deleted rows in compressed rowgroups.
- **Delete buffer**: for an NCCI, an extra B-tree that buffers deletes first. They are moved later into the delete bitmap, so OLTP deletes stay cheap.
- **Dictionary**: a lookup table mapping small integer IDs to actual values. A **global (primary) dictionary** is shared by all segments of a column. A **local (secondary) dictionary** belongs to one rowgroup. A dictionary is capped at 16 MB.
- **Trimmed rowgroup**: a compressed rowgroup closed before 1,048,576 rows. Causes include dictionary size, memory pressure, the bulk-load batch size, a build thread finishing, or a reorganize. The reason is shown as `trim_reason_desc`.
- **Mapping index**: a hidden index used when a CCI also has nonclustered B-trees. It maps old row locators to new ones when the tuple mover moves rows.

### 3. How it works, step by step
**Compression of a rowgroup (build, bulk load or tuple mover):**
1. Collect up to 1,048,576 rows in memory, either from the input stream or by reading a closed delta rowgroup.
2. For each column, choose an encoding:
   - **Value encoding** for numbers: subtract a base and divide by a power of 10, so values become small integers. For example, 1,200, 1,300 and 1,500 become 12, 13 and 15 with magnitude 10² and need fewer bits.
   - **Dictionary encoding** for strings and repetitive values: each distinct value gets an ID, and the segment stores IDs.
3. **Reorder rows (VertiPaq optimization).** SQL Server chooses a row order within the rowgroup that maximizes runs of repeated values across columns. Every column of a rowgroup uses the same row order. That is why row position N in segment A belongs to the same row as position N in segment B.
4. **Run-length encoding (RLE).** A run like `ACTIVE ×40,000` is stored as one pair.
5. **Bit-packing.** Each remaining value is stored in the minimum bits needed (an ID range 0–200 needs 8 bits, not 32).
6. **Optional archive layer.** With `COLUMNSTORE_ARCHIVE`, the result is further compressed with the Xpress algorithm.
7. Write segments and dictionaries as LOB pages, record metadata, and mark the rowgroup `COMPRESSED`.

**Write path for inserts:**
1. An `INSERT` of fewer than 102,400 rows (or a trickle of single-row inserts) goes into the current `OPEN` delta rowgroup, a normal B-tree.
2. When that delta rowgroup reaches 1,048,576 rows it becomes `CLOSED`, and a new `OPEN` one is created.
3. The tuple mover compresses the closed rowgroup into a new `COMPRESSED` rowgroup. The old delta rowgroup becomes `TOMBSTONE` and is later removed.
4. A bulk insert of ≥ 102,400 rows per batch skips steps 1–3 and creates compressed rowgroups directly. Any remainder under 102,400 rows goes to a delta store.

**Write path for deletes and updates:**
1. A `DELETE` of a row in the **delta store** removes it physically, as in any B-tree.
2. A `DELETE` of a row in a **compressed** rowgroup inserts its locator into the **delete bitmap** (for an NCCI, into the delete buffer first). The segment is untouched.
3. Every scan of that rowgroup must decompress the deleted row too and then filter it out by consulting the bitmap. Dead rows therefore still cost CPU and I/O.
4. An `UPDATE` is a delete plus an insert. The old version is flagged in the bitmap, and the new version goes to the delta store. A columnstore that is updated often therefore turns into a mix of compressed rowgroups with many deleted rows plus large delta stores.

| State | Storage | Writable? | Who moves it on |
|---|---|---|---|
| OPEN | B-tree delta store | yes (inserts) | becomes CLOSED at 1,048,576 rows (or by REORGANIZE with COMPRESS_ALL_ROW_GROUPS) |
| CLOSED | B-tree delta store | no new rows | tuple mover / REORGANIZE |
| COMPRESSED | columnar segments | no (deletes go to the bitmap) | REORGANIZE merge / REBUILD |
| TOMBSTONE | leftover delta store | no | garbage collector |

### 4. Worked example
```sql
-- Small inserts land in the delta store
INSERT Fact.Sale_CCI (...) SELECT TOP (50000) ... FROM Fact.Sale;

SELECT row_group_id, state_desc, total_rows, deleted_rows, trim_reason_desc,
       transition_to_compressed_state_desc
FROM sys.dm_db_column_store_row_group_physical_stats
WHERE object_id = OBJECT_ID('Fact.Sale_CCI')
ORDER BY row_group_id;
-- New row: state_desc = OPEN, total_rows = 50000  (a delta rowgroup)

-- Delete 100,000 rows that live in compressed rowgroups
DELETE TOP (100000) FROM Fact.Sale_CCI WHERE [Invoice Date Key] < '2014-01-01';
-- Compressed rowgroups now show deleted_rows > 0; their size_in_bytes is unchanged

-- Force compression of the open delta rowgroup and purge deleted rows
ALTER INDEX CCI_fact_Sale_CCI ON Fact.Sale_CCI
    REORGANIZE WITH (COMPRESS_ALL_ROW_GROUPS = ON);
-- OPEN -> COMPRESSED (small, trim_reason = REORG), old one -> TOMBSTONE

-- Segment-level detail: encodings, dictionaries, min/max
SELECT c.name, s.segment_id, s.encoding_type, s.row_count,
       s.on_disk_size, s.min_data_id, s.max_data_id,
       s.primary_dictionary_id, s.secondary_dictionary_id
FROM sys.column_store_segments AS s
JOIN sys.partitions AS p ON p.hobt_id = s.hobt_id
JOIN sys.columns   AS c ON c.object_id = p.object_id AND c.column_id = s.column_id
WHERE p.object_id = OBJECT_ID('Fact.Sale_CCI')
ORDER BY c.name, s.segment_id;
```
Reading `encoding_type`: **1** = numeric, value-based, no dictionary. **2** = numeric with dictionary. **3** = string/binary with dictionary. **4** = numeric stored as is. **5** = string/binary without dictionary. High-cardinality numeric columns typically show 1. Low-cardinality columns, and most strings, show 2 or 3. Compare `on_disk_size` across columns to see which ones dominate storage. A wide, unique string column (like a GUID or a free-text comment) is usually the worst.

### 5. Under the hood
- **Same row order across segments.** A row has no row ID stored inside a compressed rowgroup. Its identity is (rowgroup id, position). That is exactly what the delete bitmap stores, and what a nonclustered B-tree on a CCI uses as its 8-byte row locator.
- **Mapping index.** When the tuple mover compresses a delta rowgroup, rows change from (delta rowgroup, key) to (compressed rowgroup, position). Nonclustered B-trees would hold stale locators. Rather than updating every NC index, SQL Server keeps a hidden mapping index from old to new locators. This is part of why NC B-trees make CCI loads slower.
- **Dictionaries and trimming.** A dictionary is limited to 16 MB. A wide string column with many distinct values can fill it before 1,048,576 rows, and the rowgroup is then closed early (`DICTIONARY_SIZE`). Other trim reasons: `MEMORY_LIMITATION` (not enough memory grant to compress a full rowgroup), `BULKLOAD` (the batch size was smaller than a rowgroup), `REORG` (forced compression), and `RESIDUAL_ROW_GROUP` (leftover rows at the end of a parallel build).
- **Delete bitmap cost.** The bitmap is itself a B-tree. Scans join against it, so many deleted rows cost both reads of dead data and bitmap lookups. Above about 10% deleted rows in a rowgroup, `REORGANIZE` (2016+) rewrites the rowgroup without them.
- **Delta store is not batch-friendly.** Rows in the delta store are read in row mode from a B-tree with no compression and no elimination. A table with millions of rows sitting in open or closed delta stores loses much of the columnstore advantage.
- **Locking.** Inserts into an open delta rowgroup take locks on that rowgroup. Parallel trickle loaders can contend on it, and you can see this in `row_group_lock_wait_count` in `sys.dm_db_column_store_row_group_operational_stats`.

### 6. Performance and design implications
- Healthy columnstore: almost all rowgroups `COMPRESSED`, close to 1,048,576 rows, `deleted_rows` small, no long-lived `CLOSED` rowgroups.
- Unhealthy signs: many rowgroups of 10K–100K rows, a high `deleted_rows / total_rows` ratio, millions of rows in delta stores, and trim reasons of `MEMORY_LIMITATION` or `DICTIONARY_SIZE`.
- Keep wide unique strings out of a columnstore when possible, or move them to a separate table. They hurt compression, cause dictionary trimming, and are rarely aggregated.
- Columns with low cardinality and data loaded in a sorted order compress best, because of long RLE runs.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sys.column_store_row_groups` | State, total/deleted rows, size per rowgroup | Quick health check |
| `sys.dm_db_column_store_row_group_physical_stats` | Adds trim reason, transition reason, delta store details | Why are rowgroups small? |
| `sys.column_store_segments` | Encoding, size, min/max, dictionary IDs per segment | Compression and elimination analysis |
| `sys.column_store_dictionaries` | Dictionary type, entry count, size | Diagnosing dictionary pressure |
| `sys.dm_db_column_store_row_group_operational_stats` | Scans, delete-buffer scans, lock waits per rowgroup | Hot/cold usage, contention |
| `ALTER INDEX ... REORGANIZE WITH (COMPRESS_ALL_ROW_GROUPS = ON)` | Compress OPEN delta rowgroups now | After a final small load |
| `COMPRESSION_DELAY = n` | Rows stay in the delta store at least n minutes | NCCI on hot OLTP rows |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Rely on the tuple mover for steady append loads | Loads are bursty and small: delta stores pile up |
| Force `COMPRESS_ALL_ROW_GROUPS` after loading | Doing it after every tiny load: creates many tiny rowgroups |
| `COMPRESSION_DELAY` when rows get updated soon after insert | Pure append data: the delay only keeps rows uncompressed longer |
| Archive compression for cold rowgroups/partitions | Rowgroups that are scanned constantly |

### 9. Common mistakes
- Thinking deletes free space immediately → the size stays the same and scans still read the dead rows → monitor `deleted_rows` and reorganize.
- Running heavy `UPDATE`s on a CCI → each one becomes a bitmap entry plus a delta row, and fragmentation explodes → batch-rewrite the affected partition, or keep volatile data in rowstore.
- Misreading `CLOSED` as broken → it is just waiting for the tuple mover → wait, or `REORGANIZE`.
- Putting GUID/comment columns in a CCI without thinking → dictionary trimming and poor compression → exclude or normalize them.

### 10. 🎯 Interview kit
**30-second answer:** "Compressed rowgroups are immutable. New rows go to a delta store, a normal B-tree, until it reaches about a million rows. Then the tuple mover compresses it, using value or dictionary encoding, run-length encoding and bit-packing. Deletes are logical, recorded in a delete bitmap. Updates are a delete plus an insert. Reorganize or rebuild cleans up deleted rows and small rowgroups."
**Structure a longer answer:** rowgroup/segment layout → encodings → write path (delta store, tuple mover, 102,400 bulk threshold) → delete bitmap / buffer → consequences (trimmed rowgroups, dead rows) → how to monitor.
**Likely follow-ups:** *What is a TOMBSTONE rowgroup?* → a delta store already compressed, awaiting cleanup. *Delete bitmap vs delete buffer?* → bitmap for CCI; NCCI buffers deletes first to keep OLTP deletes fast. *Why might rowgroups be small?* → trim reasons: dictionary, memory, bulk batch size, reorg.
**Signals interviewers look for:** immutability as the core design idea, correct rowgroup states, knowing that updates are delete+insert, and knowing which DMVs to use.

### 11. Check yourself
- [ ] Trace a single-row insert, a 200,000-row bulk insert, a delete, and an update through the structures.
- [ ] List the five rowgroup states and what moves a rowgroup between them.
- [ ] Explain value encoding vs dictionary encoding with an example.
- [ ] Query the trim reasons of a columnstore and interpret `DICTIONARY_SIZE`.
- [ ] Explain why a CCI with nonclustered B-trees loads more slowly.
<!-- explainer:end -->

## In this section (study in order)

1. [Columnstore Compression and Encoding](<Compression_and_Encoding/_index.md>) (L6 Internals)

## References

- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — What are Columnstore Indexes RE COLUMNSTORE INDEXES › The Delete Bitmap
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Delete anD UpDate OperatiOns UP ATE OPERATI › Delete Operations Delete Operations
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Index Maintenance › Reorganize to Remove Delta Rowgroups
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore metadata › Rowgroup Physical Metadata
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Index arChIteCture R HTE TURE
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — What are Columnstore Indexes RE COLUMNSTORE INDEXES › The Delta Store

