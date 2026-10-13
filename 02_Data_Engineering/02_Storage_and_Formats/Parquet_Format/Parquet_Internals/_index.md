---
id: "02_Data_Engineering/02_Storage_and_Formats/Parquet_Format/Parquet_Internals"
level: 6
type: "curriculum"
curriculum: "data_engineering"
---

# Parquet File Internals

**L6 Internals** · [↑ Parquet Columnar Format](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A Parquet file is a `PAR1` magic number, a series of row groups (each holding per-column chunks made of encoded, compressed pages), and a footer that stores the schema and the location of every block — which is why readers start at the end of the file.

### 1. The big picture

*Parquet Format* explained why columnar storage helps. This lesson opens the file: the exact order of bytes, why metadata sits in the footer, and how pages, encodings and compression stack. Understanding the layout lets you reason about read cost (two seeks, then targeted reads), splittability, small-file overhead, and writer memory.

It also borrows a general idea from database storage: files are divided into fixed-size units (pages) and structures are designed to minimise the number of block fetches. Parquet applies that idea to analytical, read-mostly, write-once data.

### 2. Core concepts and vocabulary

- **Magic number `PAR1`** — 4-byte marker at both the start and the very end of a Parquet file.
- **Header** — only the 4-byte magic number; all real metadata is elsewhere.
- **Footer** — file metadata: format version, schema, extra key-value pairs, and metadata for each row group (and column chunk). Followed by a 4-byte footer length and the closing `PAR1`.
- **Row group (block)** — a set of rows stored as column chunks.
- **Column chunk** — all values of one column in a row group.
- **Page** — the unit of encoding, compression and decoding inside a column chunk.
- **Plain encoding** — values written in full (e.g., an `int32` as 4 bytes little-endian); no compression by itself.
- **Delta encoding** — store differences between successive values.
- **Run-length encoding (RLE)** — a repeated value stored once with its count.
- **Dictionary encoding** — build a dictionary of distinct values, then store small integer indexes.
- **Bit packing** — store several small values in one byte/word.
- **Sync marker** — a delimiter used by other formats (SequenceFile, Avro) to find block boundaries; Parquet does not need them.
- **Page (database sense)** — fixed-size file unit (typically 4–16 KiB) used by row stores; slotted pages and fixed-size record layouts show the design pressures Parquet avoids by being immutable.

### 3. How it works, step by step

**File layout (left to right)**

```
+------+---------------------+---------------------+-----+-----------------+--------+------+
| PAR1 | Row group 1         | Row group 2         | ... | Footer metadata | length | PAR1 |
|      |  col chunk A (pages)|  col chunk A (pages)|     | schema, groups, | 4 B    |      |
|      |  col chunk B (pages)|  col chunk B (pages)|     | chunk offsets   |        |      |
+------+---------------------+---------------------+-----+-----------------+--------+------+
```

![The internal structure of a Parquet file](<../../../../_assets/Hadoop_The_Definitive_Guide_Tom_White/hddg_1301.png>)

**Writing**

1. Write the 4-byte `PAR1` header.
2. Buffer incoming rows until the row group reaches the configured size.
3. For each column, split the buffered values into pages, encode each page, compress it, and write the column chunk. The writer remembers each block's offset and statistics in memory.
4. Repeat for further row groups.
5. After the last block, write the footer (with all recorded positions), then its 4-byte length, then `PAR1` again.

**Reading**

1. Seek to `end − 8 bytes`; read the footer length and confirm `PAR1`.
2. Seek backward by that length; read the footer metadata.
3. From the footer pick the row groups and column chunks needed.
4. Read those byte ranges, decompress pages and decode values.

### 4. Worked example

Inspecting the structure of a small file produced by a MapReduce job writing a two-column (`offset`, `line`) schema:

```bash
parquet-tools dump output/part-m-00000.parquet
```

```text
INT64 offset
--------------------------------------------------------------------------------
*** row group 1 of 1, values 1 to 4 ***
value 1: R:0 D:0 V:0
value 2: R:0 D:0 V:33
value 3: R:0 D:0 V:57
value 4: R:0 D:0 V:89

BINARY line
--------------------------------------------------------------------------------
*** row group 1 of 1, values 1 to 4 ***
value 1: R:0 D:0 V:On the top of the Crumpetty Tree
...
```

Reading it: there is one row group; within it each column is dumped as a contiguous run (`offset` values together, then `line` values together), confirming column-major layout. `R` and `D` are repetition and definition levels; both 0 because the fields are `required` and flat. With a nested or optional field you would see non-zero levels describing nesting depth and nulls.

Writing from Java with explicit sizing:

```java
MessageType schema = MessageTypeParser.parseMessageType(
    "message Pair { required binary left (UTF8); required binary right (UTF8); }");
GroupWriteSupport.setSchema(schema, conf);
ParquetWriter<Group> writer = new ParquetWriter<Group>(path, new GroupWriteSupport(),
    ParquetWriter.DEFAULT_COMPRESSION_CODEC_NAME,
    ParquetWriter.DEFAULT_BLOCK_SIZE,     // row group
    ParquetWriter.DEFAULT_PAGE_SIZE,      // page
    ParquetWriter.DEFAULT_PAGE_SIZE,      // dictionary page
    ParquetWriter.DEFAULT_IS_DICTIONARY_ENABLED,
    ParquetWriter.DEFAULT_IS_VALIDATING_ENABLED,
    ParquetProperties.WriterVersion.PARQUET_1_0, conf);
writer.write(group);
writer.close();   // close() writes the footer — a file without it is unreadable
```

Reading needs no schema or properties because both are stored in the file; you may supply a *read schema* to project a subset of columns.

### 5. Under the hood

**Why metadata is at the end.** Sequence files and Avro data files put metadata in a header and use sync markers to delimit blocks. Parquet's writer streams blocks first and keeps their offsets in memory; only after the last block does it know every position, so the metadata is written last. Consequences:

- Block boundaries are explicit, so no sync markers are needed and files are **splittable**: after reading the footer, each row group can go to a different task.
- Readers need two seeks before any data (end − 8, then back by footer length).
- A crashed writer leaves a file with no footer — unreadable. Writers should write to a temporary location and rename on success.

**Page stack.** Inside a column chunk, each page holds values of one column, so adjacent values are similar and compress well. Two levels of compression apply: first *encoding* (plain/delta/RLE/dictionary/bit-packing, chosen automatically by type), then a general *codec* (Snappy, Gzip, LZO). Booleans use RLE plus bit packing; most other types default to dictionary encoding, falling back to plain when the dictionary page would exceed the limit.

**Page size is the read granularity.** Fetching one row of one column means decompressing and decoding the entire containing page; smaller pages shorten that scan but add per-page metadata (offsets, dictionaries). Row group size governs sequential-read efficiency (less per-chunk setup) against writer/reader memory, since a whole group is buffered.

**Contrast with row-store pages.** Database engines divide files into fixed-size pages (4–16 KiB) with slotted layouts for variable-size records, handling in-place updates, balancing and pointer fix-ups. A Parquet file is immutable and written sequentially, so none of that machinery is needed: no free-space management, no pointer rebalancing — updates mean writing new files.

**Nested columns.** Each primitive leaf is a separate column chunk with definition and repetition levels stored per value, so structure can be reconstructed without ever interleaving leaf data.

### 6. Performance and design implications

- **Align sizes.** Keep row group ≤ HDFS block (both 128 MB by default) so a group is read from one datanode. On object stores, use group sizes in the 128 MB–1 GB range and consider file sizes in the hundreds of MB.
- **Footer cost.** Each file costs two seeks plus a footer read; thousands of tiny files multiply this and bloat listings. Compact small files.
- **Memory.** Writers hold a full row group per open file; many concurrently open partitions × 128 MB can exhaust executor memory → lower `parquet.block.size` or reduce open writers.
- **Selective reads.** Select only needed columns; footer statistics let engines skip row groups that cannot match a filter.
- **Codec choice.** Snappy: fast, modest ratio; Gzip: smaller, slower. Pick according to whether you are I/O-bound or CPU-bound.
- **Inspection:** `parquet-tools` (schema, dump, meta) shows row group sizes, encodings and compression per column chunk — use it before tuning.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `parquet-tools dump <file>` | Prints values per row group and column with R/D levels | Verifying layout and nesting |
| `parquet-tools meta` / `schema` | Shows footer metadata, row groups, encodings | Tuning sizes, finding small groups |
| `parquet.block.size` | Row group size (128 MB default) | Balance scan efficiency and memory |
| `parquet.page.size` | Page size (1 MB default) | Point-lookup vs scan trade-off |
| `parquet.dictionary.page.size` | Dictionary cap before plain fallback | High-cardinality columns |
| `parquet.enable.dictionary` | Toggle dictionary encoding | Disable where it doesn't help |
| `parquet.compression` | Page codec | Always set explicitly |
| Memory settings of YARN/MapReduce/Spark executors | Heap available for buffering | Fix writer out-of-memory failures |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| Large row groups for long sequential scans | Executors have little memory or many partitions are written at once |
| Small pages for point lookups | Mostly full scans (metadata overhead outweighs benefits) |
| Dictionary encoding for low/medium-cardinality columns | Near-unique values (dictionary grows, falls back to plain) |
| Snappy for speed | Network or storage is the bottleneck and CPU is idle (use heavier codec) |
| Footer-at-end design (batch, write-once) | Streaming append with no clean close — use a table format with a log |

### 9. Common mistakes

- Reading a file still being written → no footer → "not a Parquet file" errors → publish via rename/commit after `close()`.
- Hundreds of thousands of 1 MB files → footer reads dominate → compact into larger files.
- Raising `parquet.block.size` beyond task memory → OOM → size memory to row group × concurrent writers.
- Assuming sorted data automatically gives pruning → statistics only help if values cluster per group → sort/partition by filter columns before writing.
- Mismatching row group and HDFS block sizes → remote reads → align or keep row group ≤ block.

### 10. 🎯 Interview kit

**30-second answer:** A Parquet file starts and ends with the `PAR1` magic number; data is stored as row groups of column chunks made of encoded, compressed pages; the footer holds the schema and offsets for every block plus a 4-byte length, so readers seek to the end, read the footer, and then fetch only the column chunks they need. Footer-based block boundaries make files splittable without sync markers.

**Structure a longer answer:** draw the layout → write vs read path → encodings and compression → sizing (row group, page, HDFS block) → failure modes (missing footer, small files, OOM).

**Likely follow-ups:**
- *Why is the footer last?* → Writer only knows block offsets after writing them; enables splittability without sync markers.
- *What happens if the writer crashes?* → No footer, file unreadable — write to temp and rename.
- *Which encodings does Parquet use?* → Plain, delta, RLE, dictionary, bit packing, then a compression codec.
- *Row group vs page?* → Row group is a horizontal slice buffered in memory; page is the decode/compress unit within a column chunk.

**Signals interviewers look for:** accurate layout, ability to relate parameters to memory and I/O behaviour, and awareness of small-file and write-atomicity issues.

### 11. Check yourself

- [ ] Draw a Parquet file from the first byte to the last, naming every region.
- [ ] Describe the exact steps a reader takes to fetch one column from the third row group.
- [ ] Explain why Parquet needs no sync markers while Avro does.
- [ ] Give two memory-related failure modes for writers and their fixes.
- [ ] Contrast Parquet's immutable layout with a slotted B-tree page.
<!-- explainer:end -->

## References

- *Hadoop: The Definitive Guide* — Tom White — Parquet › Parquet File Format
- *Hadoop: The Definitive Guide* — Tom White — Parquet › Writing and Reading Parquet Files
- *Database Internals* — Alex Petrov — File Formats › Page Structure
- *Database Internals* — Alex Petrov — F

