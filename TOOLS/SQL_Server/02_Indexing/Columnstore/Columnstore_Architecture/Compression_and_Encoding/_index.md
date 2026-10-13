---
id: "TOOLS/SQL_Server/02_Indexing/Columnstore/Columnstore_Architecture/Compression_and_Encoding"
level: 6
type: "curriculum"
curriculum: "sql_server"
---

# Columnstore Compression and Encoding

**L6 Internals** · [↑ Columnstore Architecture](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A columnstore shrinks each column segment in three stages: it encodes values into small integers (value or dictionary encoding), reorders rows so equal values sit together (VertiPaq), then compresses the result (run-length encoding, bit packing, bit arrays, and optionally Xpress for archive).

### 1. The big picture
The architecture lesson showed that a compressed rowgroup is a set of read-only segments, one per column. This lesson explains *what is inside a segment* and why it is often 10× smaller than the same data in a rowstore. Compression is the single biggest reason columnstore indexes are fast. Less data on disk means less I/O. Less data in memory means more of the table fits in the buffer pool. Batch mode can also operate directly on some encoded forms, so the CPU work drops too.

Columns compress well because values *within one column* repeat far more than values across a row. A `Status` column in a million-row rowgroup may hold only five distinct values. A `Date` column may hold a few hundred. Row and page compression in a rowstore must work across a whole row or page that mixes strings, money and dates, so they find far fewer repeats. Columnstore compression works on one data type and one business meaning at a time, which is the ideal case.

Note one exception: the **delta store is not compressed**. It is an ordinary B-tree built for fast writes. It stays small (at most about a million rows per open rowgroup), so its uncompressed cost is constant and does not grow with the table.

### 2. Core concepts and vocabulary
- **Encoding**: turning each raw value into a compact integer before compression. Two families: value encoding and dictionary encoding.
- **Value encoding**: for numbers. Shrinks integers by dividing by a common power of 10 and subtracting a common base, so fewer bits are needed per value.
- **Dictionary encoding**: each distinct value is stored once in a *dictionary*; the segment stores only small integer IDs that point to it. Used for strings and for low-cardinality numbers.
- **Primary (global) dictionary**: a dictionary shared by all segments of a column across rowgroups.
- **Secondary (local) dictionary**: a dictionary that belongs to one or a few segments, holding values the global dictionary does not cover.
- **Cardinality**: the number of distinct values in a column. Low cardinality = small dictionary and small IDs = excellent compression.
- **VertiPaq optimization**: the engine reorders rows inside a rowgroup so that equal encoded values sit next to each other. The same reordering applies to all columns, so rows stay aligned.
- **Run-length encoding (RLE)**: storing "value × count" instead of repeating the value. Works only when equal values are adjacent.
- **Bit packing**: storing each value in the minimum number of bits it needs, not in its full declared width.
- **Bit array**: for very few distinct values, one bit column per value, with a 1 marking which value each row has.
- **COLUMNSTORE_ARCHIVE**: an extra compression layer (Microsoft Xpress = LZ77 + Huffman) on top of normal columnstore compression. Smaller, but more CPU to read and write.
- **Dictionary pressure**: a dictionary reaching its 16 MB size limit, which forces the current rowgroup to close early.

### 3. How it works, step by step
When the tuple mover or a bulk load compresses a rowgroup, SQL Server runs these steps per rowgroup:

1. **Encode every column.** Each value becomes an integer: either a rebased number (value encoding) or a dictionary ID (dictionary encoding). The engine picks per column, based on data type and cardinality.
2. **Find the best row order (VertiPaq).** The engine looks for a row order that creates the longest runs of equal values across the columns, and reorders all columns the same way.
3. **Compress each segment individually.** It applies run-length encoding, bit packing and bit arrays, whichever gives the smallest result for that segment.
4. **Optionally apply archive compression.** If the index or partition uses `COLUMNSTORE_ARCHIVE`, the finished segment is compressed again with Xpress.
5. **Write the segment** as a LOB chain of pages and record its metadata (encoding type, min/max data IDs, size, dictionary IDs).

#### 3a. Value encoding, worked
Take six integers. Store each in the fewest bits it needs:

| Value | Bits needed | ÷ 10 | Bits | − 6 (smallest) | Bits |
|---|---|---|---|---|---|
| 60 | 6 | 6 | 3 | 0 | 0 |
| 150 | 8 | 15 | 4 | 9 | 4 |
| 90 | 7 | 9 | 4 | 3 | 2 |
| 9000 | 13 | 900 | 10 | 894 | 10 |
| 630 | 10 | 63 | 6 | 57 | 6 |
| 300000 | 19 | 30000 | 15 | 29994 | 15 |
| **Total** | **63** | | **42** | | **37** |

- **Step 1: divide by a common power of 10.** All values end in 0, so the engine stores value × 10⁻¹ and remembers the exponent once for the segment.
- **Step 2: rebase.** It subtracts the smallest value (6) from all values and remembers the base once.
- Result: 63 bits → 37 bits, about 38 % smaller, before any further compression. A column where every value is the same (say 1,700,000) rebases to all zeros, which costs almost nothing.

Decimals and money work similarly: the engine scales them into integers first, then applies the same tricks.

#### 3b. Dictionary encoding, worked
A `VARCHAR(50)` column holds ten food names:

| Row | Value | Bytes |
|---|---|---|
| 1 | taco | 4 |
| 2 | hamburger | 9 |
| 3 | Fish Fry | 8 |
| 4 | taco | 4 |
| 5 | hamburger | 9 |
| 6 | taco | 4 |
| 7–9 | triple-layer Chocolate Cake ×3 | 27 each |
| 10 | Cup of Coffee | 13 |

Raw size: 132 bytes. There are 5 distinct values, so the engine builds a dictionary with IDs 0–4. Five IDs need **3 bits** each. The segment becomes ten 3-bit IDs (30 bits ≈ 4 bytes), plus one copy of each distinct string in the dictionary. On a million-row segment with five distinct strings, the dictionary cost becomes negligible and the segment shrinks enormously.

Rules of thumb:
- **Cardinality is everything.** A `CHAR(100)` with 5 distinct values compresses far better than a `CHAR(10)` with 1,000. A rough dictionary size estimate is *distinct values × average value length*.
- Dictionaries **can be shared** across rowgroups. A table with 100 rowgroups may reuse one global dictionary per column.
- **Numbers can use dictionaries too.** A low-cardinality integer (for example a customer key with only 3 values) is often dictionary-encoded; a high-cardinality key (a surrogate primary key) is value-encoded.

#### 3c. VertiPaq reordering + run-length encoding
After encoding, the ten food IDs might be `2,1,0,2,1,2,3,3,3,4`. VertiPaq reorders the rows to `0,1,1,2,2,2,3,3,3,4`. RLE then stores five pairs: `0(1) 1(2) 2(3) 3(3) 4(1)`. On real data with a million rows and a handful of values, a whole segment may collapse into a few hundred runs.

The reordering is applied to the whole rowgroup, so the row at position *n* is still the same row in every column. The engine chooses an order that benefits the columns where it helps most; it cannot sort every column perfectly at once.

#### 3d. Bit packing and bit arrays
- **Bit packing.** A 4-byte `INT` holding 30 needs only 5 bits; 950,000 needs 20 bits. The engine stores each value in a fixed number of bits big enough for the largest value in the segment, not 32 bits. Most values shrink by half or more. This works for any fixed-width type (`INT`, `BIGINT`, `DECIMAL`, `CHAR`).
- Why is this safe? Segments are **never updated in place**. An `UPDATE` is a delete (mark in the delete bitmap) plus an insert (into the delta store). So a 3-bit slot can never be asked to hold a 12-bit value.
- **Bit arrays.** When a segment has very few distinct values but they are scattered (so RLE does not help much), the engine can store one bit column per distinct value. Row *n* gets a 1 in the column for its value and 0 elsewhere.

#### 3e. Archive compression
`DATA_COMPRESSION = COLUMNSTORE_ARCHIVE` runs Xpress over each finished segment. Xpress combines **LZ77** (replace repeated byte sequences with back-references) and **Huffman coding** (shorter codes for frequent symbols). It typically saves a further large chunk of space, but every read must undo it, so it costs CPU on every scan. You can set it per partition, so old partitions can be archive-compressed while current ones stay normal.

#### 3f. Compressed in memory, too
Columnstore segments stay compressed on disk **and** in memory. They are decompressed only when a query needs the values (and batch mode can often work on encoded data directly). Every byte you save through better compression is a byte of buffer pool you get back.

### 4. Worked example
```sql
-- 1) Compare standard and archive compression on the same data
CREATE TABLE dbo.Sale_CCI (
    SaleKey BIGINT NOT NULL, CustomerKey INT NOT NULL,
    Description NVARCHAR(100) NOT NULL, Quantity INT NOT NULL,
    UnitPrice DECIMAL(18,2) NOT NULL, InvoiceDate DATE NOT NULL);
CREATE CLUSTERED COLUMNSTORE INDEX CCI_Sale ON dbo.Sale_CCI;

SELECT * INTO dbo.Sale_CCI_Archive FROM dbo.Sale_CCI WHERE 1 = 0;
CREATE CLUSTERED COLUMNSTORE INDEX CCI_Sale_Arch ON dbo.Sale_CCI_Archive
    WITH (DATA_COMPRESSION = COLUMNSTORE_ARCHIVE);
-- load both with the same rows (bulk insert, >= 102,400 rows per batch)

EXEC sp_spaceused 'dbo.Sale_CCI';
EXEC sp_spaceused 'dbo.Sale_CCI_Archive';

-- 2) How is each column encoded, and how big is each segment?
SELECT c.name AS column_name, s.segment_id, s.row_count,
       s.encoding_type,                 -- see table in section 5
       s.primary_dictionary_id, s.secondary_dictionary_id,
       s.on_disk_size / 1024 AS kb
FROM sys.column_store_segments s
JOIN sys.partitions p ON p.hobt_id = s.hobt_id
JOIN sys.columns   c ON c.object_id = p.object_id AND c.column_id = s.column_id
WHERE p.object_id = OBJECT_ID('dbo.Sale_CCI')
ORDER BY c.column_id, s.segment_id;

-- 3) Dictionaries: how many, how big
SELECT c.name AS column_name, d.dictionary_id, d.type, d.entry_count,
       d.on_disk_size / 1024 AS kb
FROM sys.column_store_dictionaries d
JOIN sys.partitions p ON p.hobt_id = d.hobt_id
JOIN sys.columns   c ON c.object_id = p.object_id AND c.column_id = d.column_id
WHERE p.object_id = OBJECT_ID('dbo.Sale_CCI');

-- 4) Did any rowgroup close early because a dictionary filled up?
SELECT row_group_id, state_desc, total_rows, trim_reason_desc
FROM sys.dm_db_column_store_row_group_physical_stats
WHERE object_id = OBJECT_ID('dbo.Sale_CCI')
ORDER BY row_group_id;
```
**How to read it:**
- In query 2, `SaleKey` should show value encoding (no dictionary IDs, `-1` or 0) and a large segment size, because every value is distinct. `CustomerKey` with few values will usually be dictionary-encoded and tiny. `Description` will use a string dictionary.
- In query 3, a column with a huge `entry_count` and a dictionary near 16 MB is a dictionary-pressure risk.
- In query 4, `trim_reason_desc = DICTIONARY_SIZE` with `total_rows` far below 1,048,576 confirms dictionary pressure. Healthy rowgroups show `NO_TRIM` and ~1,048,576 rows.

### 5. Under the hood
**Encoding types** (`sys.column_store_segments.encoding_type`):

| Value | Meaning |
|---|---|
| 1 | Value-based: non-string, no dictionary (rebasing + scaling) |
| 2 | Value-hash-based: non-string, uses a dictionary |
| 3 | String-hash-based: string, uses a dictionary |
| 4 | Store-by-value: non-string, no dictionary, values stored as-is |
| 5 | String store-by-value: string, no dictionary |

**Segment metadata.** Each segment stores `min_data_id` and `max_data_id`. For value-encoded columns these are encoded numbers; for dictionary columns they are dictionary IDs. This is what makes segment elimination possible (next lessons). Segments also record `base_id` and `magnitude`, which are the rebasing base and the power-of-10 scale from section 3a.

**Global vs local dictionaries.** `primary_dictionary_id` points to the global dictionary for the column (shared across rowgroups in the partition). `secondary_dictionary_id` points to a local dictionary used by that segment for values not in the global one. `-1` means "none".

**Dictionary limit and pressure.** A dictionary is capped at **16 MB**. The engine checks how big the dictionary *would* get, so pressure often appears well before 16 MB is used. When a column has wide, rarely repeated strings (free-text notes, GUIDs as strings, long URLs), the dictionary fills before the rowgroup reaches 1,048,576 rows. SQL Server then closes the rowgroup early (trim reason `DICTIONARY_SIZE`) and starts a new one with a new dictionary. The result is many undersized rowgroups, more dictionaries, more metadata and slower scans.

**Why compressed segments are immutable.** Bit packing, RLE and reordering all depend on the *whole* segment. Changing one value could change the bit width, break a run, or require a different order. So the engine never edits a segment; it uses the delete bitmap and delta store instead, and rebuilds segments during `REORGANIZE` / `REBUILD`.

**String normalization test.** The book tests replacing an `NVARCHAR(100)` `Description` column with a `SMALLINT` key into a lookup table. The fact table shrank by roughly 15:1 for that column. Dictionary encoding of a small integer beats dictionary encoding of a long string, because the dictionary itself is tiny and the IDs fit in fewer bits.

### 6. Performance and design implications
- **Lower cardinality per column = better compression.** Avoid storing high-cardinality strings in fact tables when a key would do.
- **Narrow data types help.** `DATE` instead of `DATETIME2(7)` when you do not need time; `INT` instead of `BIGINT` when the range allows; `DECIMAL(9,2)` instead of `DECIMAL(38,10)`.
- **Order matters.** VertiPaq can only reorder *within* a rowgroup. Loading data pre-sorted on a common filter column (or using an ordered columnstore, SQL Server 2022+) gives longer runs and better segment elimination.
- **Watch for dictionary pressure.** Check `trim_reason_desc`. Fix by moving wide text columns out of the columnstore table, normalizing them to keys, or splitting very wide string data into a separate rowstore table.
- **Archive only cold data.** Archive saves storage but adds CPU on every read. Put it on old partitions that are rarely queried.
- **Measure:** `sp_spaceused`, `sys.column_store_segments.on_disk_size`, `sys.dm_db_column_store_row_group_physical_stats`, and `SET STATISTICS IO ON` (look at `lob logical reads` and `segment reads / skipped`).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `WITH (DATA_COMPRESSION = COLUMNSTORE)` | Default columnstore compression | Hot / current data |
| `WITH (DATA_COMPRESSION = COLUMNSTORE_ARCHIVE)` | Adds Xpress compression | Cold, rarely read data |
| `ALTER INDEX ... REBUILD PARTITION = n WITH (DATA_COMPRESSION = COLUMNSTORE_ARCHIVE)` | Changes compression of one partition | Ageing out old partitions |
| `sys.column_store_segments` | Per-segment encoding, size, min/max, dictionary IDs | Check encoding choice and segment size |
| `sys.column_store_dictionaries` | Per-dictionary type, entries, size | Find large dictionaries |
| `sys.dm_db_column_store_row_group_physical_stats` | Rowgroup state, rows, `trim_reason_desc` | Detect dictionary pressure |
| `sp_estimate_data_compression_savings` | Estimates savings for a compression setting (columnstore supported from 2019) | Before switching to archive |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Standard columnstore compression for anything queried regularly | — |
| Archive compression for old partitions read a few times a month | Data scanned often; CPU-bound servers |
| Normalizing a long, low-cardinality string to a small key | Loads become slower (lookup step) and queries need a join; small strings gain little |
| Keeping a string column inline | High-cardinality wide strings that cause dictionary pressure |
| Pre-sorting data before load | Sorting costs time and tempdb on huge loads; benefit is mostly for common filter columns |

### 9. Common mistakes
- Putting free-text or long unique strings into a clustered columnstore → dictionary pressure, small rowgroups, poor compression → move them to a side table or normalize.
- Using archive compression on hot data "to save space" → every scan pays Xpress decompression CPU → use it per partition on cold data only.
- Assuming compression is fixed after load → trickle inserts produce small delta-compressed rowgroups → bulk load in batches ≥ 102,400 rows and reorganize.
- Oversized data types (`NVARCHAR(MAX)`, `DATETIME2(7)`, `DECIMAL(38,x)`) → wider values and dictionaries → use the narrowest correct type.
- Expecting `UPDATE` to change values in place → it is delete + insert → plan for the delete bitmap and delta store growth.

### 10. 🎯 Interview kit
**30-second answer:** Columnstore compresses each column segment separately. First it encodes values: numbers are rebased and scaled (value encoding), strings and low-cardinality numbers go through a dictionary so the segment stores tiny IDs. Then VertiPaq reorders rows so equal values sit together, and run-length encoding, bit packing and bit arrays shrink the result. Archive compression can add Xpress on top for cold data. Segments stay compressed in memory, so the savings also reduce buffer pool use.

**Structure a longer answer:** clarify (columnstore vs row/page compression) → encoding (value vs dictionary, cardinality) → VertiPaq + RLE + bit packing → dictionary pressure and archive trade-offs → close with a real case (e.g. a wide comment column that caused 400k-row rowgroups, fixed by moving it out).

**Likely follow-ups:**
- *Why does columnstore compress better than page compression?* → one data type and one meaning per segment; many repeats; no per-row overhead.
- *What is dictionary pressure and how do you detect it?* → dictionary hits 16 MB limit; rowgroup closes early; `trim_reason_desc = DICTIONARY_SIZE`.
- *Is the delta store compressed?* → no; it is a plain B-tree for fast writes.
- *Why are segments read-only?* → bit widths, runs and row order depend on the whole segment; updates are delete + insert.
- *When would you use COLUMNSTORE_ARCHIVE?* → cold partitions; trade CPU for storage.

**Signals interviewers look for:** knows cardinality drives compression; mentions VertiPaq reordering as a prerequisite for RLE; can name the DMV and trim reason for dictionary pressure; understands archive is a CPU/storage trade-off, set per partition.

### 11. Check yourself
- [ ] Rebase the values 120, 80, 400 by hand (scale, then subtract the minimum) and count bits saved.
- [ ] Explain why ten strings with 5 distinct values need 3-bit IDs.
- [ ] Explain why VertiPaq reordering must happen before run-length encoding.
- [ ] Query `sys.column_store_segments` on a table and say which columns are value- vs dictionary-encoded.
- [ ] Find a rowgroup with `trim_reason_desc = DICTIONARY_SIZE` and propose two fixes.
- [ ] Argue for and against archive compression on a 3-year-old partition.
<!-- explainer:end -->

## References

- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Compression › Columnstore Compression Algorithms
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Compression › Other Compression Algorithms
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Index arChIteCture R HTE TURE › CHAPTER 5
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Index Performance › Dictionary Size and Dictionary Pressure
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Compression › Columnstore Archive Compression
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Compression
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Compression › The Compression Life Cycle
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Dropping an Index › sys.columnstoredictionaries
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Compression › Should String Data Be Normalized?
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Index Maintenance › The No-Maintenance Columnstore Index

