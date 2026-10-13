---
id: "02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Parquet_Internals"
level: 4
type: "concept"
topic: "02_Data_Engineering/Storage_and_Lakehouse/File_Formats"
tools: ["Apache_Spark"]
---

# Parquet Internals

**L4 Advanced** · [↑ File Formats](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A Parquet file is a set of row groups, each holding one compressed, encoded chunk per column, plus a footer of statistics that lets engines read only the columns and chunks a query needs.

### 1. The big picture
Parquet is the default storage format for analytics on data lakes and lakehouses. Its value is simple: an analytical query such as "total sales by region last quarter" needs two columns out of dozens, over millions of rows. Row formats like CSV force the engine to read and parse everything. Parquet stores each column's values together, compresses them aggressively (similar values sit side by side), and keeps statistics so whole chunks can be skipped.

Published comparisons illustrate the scale: the same 1 TB dataset stored as CSV versus Parquet shrank to roughly 130 GB (about 87% smaller), ran a representative query dozens of times faster and scanned about 99% less data. Compression ratios of 10:1 on repetitive columns are common, and moving from other formats often saves around 75% of storage depending on the codec.

Parquet is also the physical layer under Delta Lake and Apache Iceberg tables: those table formats add transaction logs and metadata on top of Parquet data files. Understanding Parquet's internals is therefore how you reason about file sizing, skipping, partitioning and cost across the whole lakehouse.

### 2. Core concepts and vocabulary
- **Row group** — a horizontal slice of the table (commonly 64–512 MB uncompressed, ~128 MB default in many writers) containing all columns for a set of rows; the unit of parallelism and skipping.
- **Column chunk** — the data for one column within one row group.
- **Page** — the unit inside a column chunk (typically ~1 MB): data pages, dictionary page, optional index pages. Compression and encoding apply per page.
- **Footer (file metadata)** — at the end of the file: schema, row-group locations, column-chunk offsets, statistics (min/max/null count), encodings, codec.
- **Encoding** — how values are represented: PLAIN, dictionary (RLE_DICTIONARY), RLE/bit-packing, delta encodings.
- **Compression codec** — Snappy (fast, moderate ratio), ZSTD (better ratio, tunable), GZIP (slow, high ratio), LZ4.
- **Definition and repetition levels** — small integers that encode nulls and nesting so arrays/maps/structs can be stored columnarly (the Dremel model).
- **Predicate pushdown** — using footer/page statistics to skip row groups or pages.
- **Column pruning / projection pushdown** — reading only requested column chunks.
- **Bloom filter** — optional per-column filter for equality lookups on high-cardinality columns.

### 3. How it works, step by step
**File layout**
```
[ magic "PAR1" ]
[ Row group 1 ]  → Column chunk A (pages…) | Column chunk B (pages…) | …
[ Row group 2 ]  → …
[ Footer: schema + row-group/column-chunk metadata + stats ]
[ footer length ][ magic "PAR1" ]
```

**Reading a query** — `SELECT region, SUM(amount) FROM sales WHERE sale_date >= '2026-01-01' GROUP BY region`:
1. Engine reads the last bytes of the file to find the footer; parses schema and row-group stats (one small read).
2. **Column pruning:** only `region`, `amount`, `sale_date` chunks are candidates; the other 47 columns are never read.
3. **Row-group skipping:** for each row group, compare the `sale_date` min/max with the predicate; skip groups whose max < `2026-01-01`.
4. **Page skipping** (with page indexes): skip pages within a chunk similarly.
5. Fetch needed chunks (range requests on object storage), decompress, decode (dictionary → values, RLE → runs).
6. Filter remaining rows, aggregate, return.

**Writing**
1. Buffer rows; when the buffer reaches the row-group size, transpose into column chunks.
2. For each column choose encoding (dictionary if cardinality is low; fall back to plain if the dictionary grows too large).
3. Split into pages, compress each, record statistics.
4. Write chunks; at close, write the footer.

### 4. Worked example
```python
import pyarrow.parquet as pq

pf = pq.ParquetFile("sales_2026_01.parquet")
print(pf.metadata)                    # rows, row groups, created_by
print(pf.schema_arrow)
rg = pf.metadata.row_group(0)
for i in range(rg.num_columns):
    c = rg.column(i)
    print(c.path_in_schema, c.compression, c.encodings,
          c.total_compressed_size, c.statistics.min, c.statistics.max)

# column pruning + predicate pushdown
tbl = pq.read_table("sales_2026_01.parquet",
                    columns=["region","amount"],
                    filters=[("sale_date", ">=", "2026-01-15")])
```
Reading output: `compression=SNAPPY`, `encodings=(RLE_DICTIONARY, PLAIN, RLE)` for `region` means dictionary encoding is working; `statistics.min/max` on `sale_date` show whether the file is sorted by date (narrow ranges → good skipping). If every row group has min=Jan 1 and max=Jan 31, skipping can't help — a sign data wasn't clustered by that column.

In Spark/SQL:
```sql
-- Write sorted & partitioned to maximize skipping
INSERT OVERWRITE TABLE sales PARTITION (sale_month)
SELECT ... FROM staging ORDER BY sale_date;
```

### 5. Under the hood
- **Dictionary encoding:** each distinct value gets an integer ID; the column stores IDs bit-packed, plus one dictionary page. For a column with 20 distinct countries across 100M rows, IDs need 5 bits each. If the dictionary exceeds a threshold (~1 MB by default), the writer falls back to plain for later pages.
- **RLE / bit-packing hybrid:** runs of identical IDs collapse to (value, count); varied values are bit-packed. Sorted or clustered data yields long runs and massive savings.
- **Delta encodings** (DELTA_BINARY_PACKED) suit timestamps/IDs; byte-stream-split helps floating-point.
- **Nested data:** repetition/definition levels flatten nested structures into columns, so reading `address.city` doesn't require reading the whole struct.
- **Statistics are per chunk/page.** Min/max/null_count enable skipping; they are weak for unsorted data and for strings with long prefixes (truncated). Bloom filters help equality on IDs.
- **Immutability.** Parquet files aren't updated in place; an UPDATE rewrites files. This is why Delta/Iceberg add metadata/transaction layers, and why many small writes produce many small files.
- **Small files problem.** Each file has a footer and each read has request overhead on object storage; thousands of 1 MB files waste more on listing/opening than scanning. Target ~128 MB–1 GB files.
- **Row-group size trade-off:** larger groups → better compression and fewer metadata reads, but more memory per writer/reader and coarser skipping.
- **Type pitfalls:** timestamp units (INT96 legacy vs INT64 micros), decimal precision, and schema-merging differences between engines.

### 6. Performance and design implications
- Cost ≈ (bytes read from chunks you need) + (metadata/IO overhead). Optimize both: select needed columns, cluster by filter columns, right-size files.
- Partition by low-cardinality columns used in filters (date); don't over-partition (millions of tiny directories).
- Use ZSTD or Snappy: Snappy for speed-sensitive hot data, ZSTD for cold/cost-sensitive.
- Sort within files by the main filter column(s) to tighten min/max and improve RLE.
- Compact regularly after streaming/incremental writes.
- Inspect with `parquet-tools`/`pyarrow` before tuning; check row-group counts and stats.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `parquet-tools meta/schema` or `pyarrow.parquet.ParquetFile` | Inspect footer and stats | Diagnose skipping/size |
| `spark.sql.parquet.compression.codec` | Choose codec | Tune cost/speed |
| `parquet.block.size` / row group size | Row-group size | Tuning memory vs skipping |
| `spark.sql.files.maxPartitionBytes` | Input split size | Balance read parallelism |
| `OPTIMIZE` (Delta) / `rewrite_data_files` (Iceberg) | Compact small files | After many small writes |
| Bloom filter options | Per-column equality filter | High-cardinality lookups |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Analytics scans, aggregations, wide tables | Single-record lookups/updates (use OLTP store) |
| Interoperating across Spark, Presto/Trino, warehouses | Streaming record-at-a-time append of tiny messages (Avro first) |
| Cheap object storage + data skipping | Highly random, unsorted data where stats don't help |
| ZSTD for cold data | Latency-critical CPU-bound reads where decompression dominates |

### 9. Common mistakes
- Millions of tiny files → slow listing and planning → compact to hundreds of MB.
- Selecting `*` → defeats column pruning → project needed columns.
- Unsorted data → wide min/max → no skipping → sort/cluster on filter columns.
- Over-partitioning by high-cardinality columns → metadata explosion → partition by date, cluster others.
- Treating Parquet as updatable → expensive rewrites → use Delta/Iceberg/Hudi for upserts.
- Mixed schemas across files without schema evolution handling → read errors → enforce schema or use table format.

### 10. 🎯 Interview kit
**30-second answer:** Parquet is a columnar file format: a file contains row groups, each with a column chunk per column, split into compressed, encoded pages; a footer holds the schema and per-chunk min/max statistics. Engines read only needed columns and skip row groups/pages whose stats can't match the filter, and columnar compression (dictionary, RLE, delta) shrinks data dramatically.

**Structure a longer answer:** layout → read path (prune columns, skip groups, decode) → encodings/compression → write path and file sizing → trade-offs (immutability, small files) → relation to Delta/Iceberg.

**Likely follow-ups:**
- *Why smaller than CSV?* → Columnar grouping + dictionary/RLE/delta + codec + binary types.
- *Why is predicate pushdown sometimes ineffective?* → Unsorted data (min/max overlap), non-sargable expressions, or stats missing/truncated.
- *How does Parquet store nested data?* → Dremel repetition/definition levels.
- *Optimal file size?* → ~128 MB–1 GB, balancing parallelism and overhead.
- *How do you update data?* → Rewrite files; table formats manage this with metadata/transaction logs.

**Signals interviewers look for:** layout understanding (row group/chunk/page/footer), knowing why skipping works, and operational points (small files, sorting).

### 11. Check yourself
- [ ] Draw the file layout from magic bytes to footer.
- [ ] Walk a filtered aggregate through pruning, skipping and decoding.
- [ ] Explain dictionary + RLE benefit with sorted vs unsorted data.
- [ ] Choose file size and partition scheme for 5 TB of daily events.
- [ ] Explain why Parquet needs Delta/Iceberg for ACID updates.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)

## Practice questions

- [Basic] [How does Parquet store its schema and why is it called self-describing?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Q-09468_How_does_Parquet_store_its_schema_and_why_is_it_called_self-.md>)
- [Intermediate] [What are the key differences between Parquet and ORC?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Q-09553_What_are_the_key_differences_between_Parquet_and_ORC.md>)
- [Intermediate] [How would you verify that a Parquet file is not corrupted?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Q-09629_How_would_you_verify_that_a_Parquet_file_is_not_corrupted.md>)
- [Deep] [What are repetition and definition levels in Parquet and roughly what do they encode?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Q-09580_What_are_repetition_and_definition_levels_in_Parquet_and_rou.md>)
- [Deep] [Your Athena bill spiked after a schema migration even though data volume was flat -- how would you trace it back to Parquet layout?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Q-09438_Your_Athena_bill_spiked_after_a_schema_migration_even_though.md>)
- [Deep] [How do min/max stats become useless after an UPDATE-heavy pattern of rewrites, and how would you restore effective pruning?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Q-09482_How_do_min_max_stats_become_useless_after_an_UPDATE-heavy_pa.md>)
- [Deep] [How do you weigh the cost of maintaining ORC indexes and bloom filters against their query benefit on a Hive-heavy platform?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Q-09510_How_do_you_weigh_the_cost_of_maintaining_ORC_indexes_and_blo.md>)
- [Deep] [How do you evaluate whether enabling Parquet column indexes helps your selective range queries or just adds write overhead?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Q-09547_How_do_you_evaluate_whether_enabling_Parquet_column_indexes_.md>)

## Related concepts

- [Columnar Formats in the Lake](<../Lakehouse/Columnar_Formats_in_the_Lake.md>) *(Lakehouse)*
- [Min Max Statistics and Data Skipping](<../../../01_Databases_and_SQL/Query_Performance/Indexing/Min_Max_Statistics_and_Data_Skipping.md>) *(Indexing)*
- [Partitioning and Data Skipping in Lakes](<../../../01_Databases_and_SQL/Query_Performance/Indexing/Partitioning_and_Data_Skipping_in_Lakes.md>) *(Indexing)*
- [Pandas vs Polars vs PySpark](<../../../TOOLS/Python/Python_Language/Pandas_vs_Polars_vs_PySpark.md>) *(Python Language)*
- [Large Values and TOAST](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Large_Values_and_TOAST.md>) *(SQL Fundamentals)*
- [Lakehouse Fundamentals](<../Lakehouse/Lakehouse_Fundamentals.md>) *(Lakehouse)*

## References

- *Azure Data Fundamentals* — Michael John Pena — Data Storage Fundamentals › Why columnar storage matters
- *Azure Data Fundamentals* — Michael John Pena — Data Storage Fundamentals › When Parquet excels
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Getting Started with Delta Lake › Advantages of Parquet files
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Getting Started with Delta Lake › Writing a Parquet file
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Schema Handling › Display the data file(s)\
- *Hadoop: The Definitive Guide* — Tom White — Parquet › Parquet File Format
- *Hadoop: The Definitive Guide* — Tom White — Parquet › Parquet Configuration
- *Hadoop: The Definitive Guide* — Tom White — Parquet › Writing and Reading Parquet Files
- *Hadoop: The Definitive Guide* — Tom White — Parquet › Projection and read schemas
- *Building Medallion Architectures* — Piethein Strengholt — Demystifying the Medallion Architecture › Conclusion
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Storage The Heart of the Lakehouse › Key features
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Portfolio data engineering project › Step 3: Provision a Blob storage container on Azure and upload your extracted data to it as a Parquet file.

