---
id: "02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Row_vs_Columnar_Formats"
level: 4
type: "concept"
topic: "02_Data_Engineering/Storage_and_Lakehouse/File_Formats"
tools: []
---

# Row vs Columnar Formats

**L4 Advanced** · [↑ File Formats](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Row-oriented file formats (CSV, JSON, Avro) are built to write and exchange whole records; columnar file formats (Parquet, ORC) are built to read a few columns from many records — pick by access pattern and pipeline stage.

### 1. The big picture
Where the previous storage lesson compared row and column layouts inside a database engine, this one compares **file formats** in a lake or pipeline. Files are how data moves between systems (ingestion, exchange, archival) and how lakehouses store tables. The format determines what you can read cheaply, how well data compresses, whether schema travels with the data, and whether engines can skip data.

Row-oriented serialization encodes complete relations or documents sequentially: CSV rows, JSON documents, Avro records. Columnar serialization pivots the data so each column is stored on its own, which allows reading a subset of fields and encoding repetitive values efficiently. Most real platforms use both: row formats at the edges (ingest, streaming, raw landing), columnar formats in the middle and at serving (curated, analytics).

### 2. Core concepts and vocabulary
- **Row-oriented format** — records written one after another (CSV, JSON, Avro, protobuf).
- **Columnar format** — values grouped by column (Parquet, ORC, Arrow in memory).
- **Self-describing** — the file carries its schema (Avro, Parquet, ORC; not CSV).
- **Splittable** — a large file can be divided so multiple workers read it in parallel.
- **Schema-on-read vs schema-on-write** — interpreting structure when reading (CSV/JSON) vs enforcing at write (Avro/Parquet).
- **Nested data support** — storing arrays/structs/maps natively.
- **Compression codec** — Snappy, Gzip, ZSTD, LZ4 applied inside blocks/pages.
- **Apache Arrow** — an in-memory columnar format for zero-copy exchange between engines (not a storage format).
- **ORC** — Optimized Row Columnar; Hive-era columnar format similar to Parquet.
- **Binary vs text** — text is human-readable but large and slow to parse.

### 3. How it works, step by step
Same table, four formats:

| Property | CSV | JSON (lines) | Avro | Parquet / ORC |
|---|---|---|---|---|
| Orientation | Row | Row | Row | Column |
| Schema in file | No | Implicit (per record) | Yes (header/registry) | Yes (footer) |
| Types | Strings, inferred | Limited | Rich + logical | Rich + logical |
| Nested data | No | Yes | Yes | Yes |
| Compression | External (gzip) | External | Block codec | Per-column encodings + codec |
| Column pruning | No | No | No | Yes |
| Predicate skipping | No | No | No (some sync-block) | Yes (min/max stats) |
| Splittable | Only if uncompressed / splittable codec | Same | Yes (blocks) | Yes (row groups) |
| Append record-at-a-time | Easy | Easy | Easy (blocks) | Poor (needs batching) |
| Typical role | Exchange, legacy | Logs, APIs | Streaming, raw landing | Analytics tables |

**Why the columnar layout is smaller:** values in a column are the same type and often similar (country codes, statuses, sorted dates). Encodings such as dictionary and run-length compress them far better than interleaved rows of mixed types. A column like `status` with 5 distinct values over 100M rows becomes a tiny dictionary plus bit-packed IDs.

**Decision flow**
1. Is it the first landing of data from a source or a stream? → Row/self-describing (Avro, or raw JSON/CSV kept as-is for fidelity).
2. Will analysts or engines scan it? → Convert to Parquet (or ORC in Hive-centric stacks).
3. Do you need ACID updates/time travel? → Parquet data files inside Delta/Iceberg/Hudi.
4. Is it for in-memory exchange between engines? → Arrow.

### 4. Worked example
Convert raw CSV into analytic Parquet and compare:

```python
import pandas as pd, pyarrow as pa, pyarrow.parquet as pq, os

df = pd.read_csv("events.csv", parse_dates=["event_time"])
df["country"] = df["country"].astype("category")

df.to_csv("events_copy.csv", index=False)
pq.write_table(pa.Table.from_pandas(df), "events.parquet",
               compression="zstd", row_group_size=1_000_000)

for f in ["events.csv", "events.parquet"]:
    print(f, os.path.getsize(f)/1e6, "MB")

# read two columns only
pq.read_table("events.parquet", columns=["country","amount"])
```
Typical outcome: the Parquet file is 70–90% smaller; reading two columns touches a fraction of the bytes. The published CSV-vs-Parquet benchmark for a 1 TB dataset shows ~130 GB on disk, ~34× faster query, ~99% less data scanned and ~99.7% lower scan cost on a pay-per-scan service. Your numbers vary with data shape, but direction is consistent.

In Spark:
```python
spark.read.option("header","true").csv("s3://raw/events/") \
     .write.mode("overwrite").partitionBy("event_date").parquet("s3://curated/events/")
```

### 5. Under the hood
- **Why Avro still matters:** row-based, schema evolution, block-compressed and splittable; excellent for write-heavy pipelines where each record is processed whole (stream processing, ETL staging). The common combination is Avro for raw/ETL, Parquet for analytics.
- **ORC vs Parquet:** both columnar with stripes/row groups, stats and compression. ORC is deeply integrated with Hive (ACID tables, indexes); Parquet has broader support across Spark, cloud warehouses and lakehouse table formats. Some warehouses import and export Parquet but only import ORC. Default to Parquet unless a Hive stack dictates otherwise.
- **Proprietary warehouse formats** (BigQuery, Snowflake) are columnar and fast internally, but interoperating with external tools means deserializing and re-exporting; open files in the lake avoid that tax in a polyglot environment.
- **Gzip on CSV** is not splittable: one worker reads the entire file. Use splittable codecs/containers or Parquet.
- **Text parsing cost:** CSV/JSON require tokenizing, type inference and string conversion; binary columnar decoding is vectorized and often CPU-cheaper despite decompression.
- **Write amplification:** columnar files buffer rows to build row groups; frequent tiny writes yield small files.

### 6. Performance and design implications
- Scan cost scales with bytes read — columnar + pruning + skipping reduces this by orders of magnitude on wide tables.
- Landing raw data in an open row format preserves fidelity and enables replay; curated layers in columnar formats serve cost-efficient queries.
- Partition by date; keep file sizes in the 128 MB–1 GB range; avoid thousands of small files.
- Decide codecs: Snappy (speed), ZSTD (ratio/cost balance), Gzip (rarely best now).
- For real-time/streaming sinks, use micro-batching and later compaction rather than writing Parquet per record.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `df.write.parquet(...)` / `.partitionBy(...)` | Writes columnar, partitioned | Curated layer |
| `spark.sql.parquet.compression.codec` | Sets codec | Cost/speed tuning |
| `COPY INTO … FILE_FORMAT=(TYPE=PARQUET)` (Snowflake) | Load/unload Parquet | Warehouse interchange |
| `CREATE EXTERNAL TABLE … STORED AS PARQUET` | Query files in place | Lake query engines |
| Avro `spark.read.format("avro")` | Read raw Avro | Ingestion/ETL |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| CSV: tiny exports, human exchange | Anything large, typed or repeated |
| JSON: semi-structured logs/API payloads | Scanning at scale (costly parse, no skipping) |
| Avro: streaming, raw landing, evolving schema | Analytical scans |
| Parquet: analytics, lakehouse tables | Record-at-a-time updates and appends |
| ORC: Hive-centric ecosystem | Broad cross-tool interop is needed |

### 9. Common mistakes
- Keeping analytics data as CSV/JSON → huge scans and bills → convert to Parquet.
- Converting everything to Parquet at ingest, losing raw fidelity → keep raw immutable copy.
- Writing Parquet per message/stream event → small files → batch and compact.
- Gzipped single big CSV → no parallelism → splittable format.
- Ignoring schema inference inconsistencies across files → type conflicts → explicit schemas.
- Assuming ORC and Parquet are interchangeable everywhere → tool-support gaps → check targets.

### 10. 🎯 Interview kit
**30-second answer:** Row formats like CSV, JSON and Avro store whole records together, which suits ingestion, streaming and full-record processing. Columnar formats like Parquet and ORC store each column separately, enabling column pruning, strong compression and min/max skipping — which is why analytics tables use them. A typical lakehouse lands raw data as Avro/JSON and serves curated data as Parquet.

**Structure a longer answer:** access patterns → format properties (orientation, schema, splittability) → layer-by-layer recommendation → performance numbers → trade-offs and pitfalls.

**Likely follow-ups:**
- *Why not Parquet everywhere?* → Poor for record-at-a-time writes and schema-light raw capture; fine as the curated format.
- *Parquet vs ORC?* → Similar design; ORC for Hive, Parquet for ecosystem breadth.
- *Why is CSV bad for analytics?* → No types or schema, text parsing, no pruning/skipping, poor compression, delimiter/escape bugs.
- *What's Arrow for?* → In-memory columnar interchange; not for long-term storage.

**Signals interviewers look for:** you map format to pipeline stage and workload, quantify benefits, and mention small-file and schema concerns.

### 11. Check yourself
- [ ] Fill in a format comparison table from memory.
- [ ] Justify formats for raw, curated and serving layers.
- [ ] Explain why gzipped CSV isn't parallelizable.
- [ ] Describe two reasons Parquet files are smaller than CSV.
- [ ] Explain Arrow's role relative to Parquet.
<!-- explainer:end -->

## Practice questions

- [Basic] [What does it mean that Avro carries its schema with the data and why is that useful?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Q-09479_What_does_it_mean_that_Avro_carries_its_schema_with_the_data.md>)
- [Basic] [Why does storing data by column let you read only the columns a query needs?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Q-09590_Why_does_storing_data_by_column_let_you_read_only_the_column.md>)
- [Basic] [Why do columnar formats compress better than row formats?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Q-09612_Why_do_columnar_formats_compress_better_than_row_formats.md>)
- [Intermediate] [What is the trade-off between compression ratio and CPU cost when picking a codec?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Q-09457_What_is_the_trade-off_between_compression_ratio_and_CPU_cost.md>)
- [Intermediate] [When would you still keep data in CSV or JSON instead of a columnar format?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Q-09592_When_would_you_still_keep_data_in_CSV_or_JSON_instead_of_a_c.md>)
- [Deep] [What is partition pruning and how is it different from column pruning?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Q-09571_What_is_partition_pruning_and_how_is_it_different_from_colum.md>)
- [Deep] [If your pipeline ingests Avro from Kafka and needs to serve analytics what format would you convert to and why?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Q-09610_If_your_pipeline_ingests_Avro_from_Kafka_and_needs_to_serve_.md>)
- [Deep] [Why can column statistics be misleading when a column contains many NULLs, and how does that affect row-group pruning decisions?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Q-09450_Why_can_column_statistics_be_misleading_when_a_column_contai.md>)

## Related concepts

- [Columnar Formats in the Lake](<../Lakehouse/Columnar_Formats_in_the_Lake.md>) *(Lakehouse)*
- [Min Max Statistics and Data Skipping](<../../../01_Databases_and_SQL/Query_Performance/Indexing/Min_Max_Statistics_and_Data_Skipping.md>) *(Indexing)*
- [Vector Data in the Lakehouse](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Vector_Data_in_the_Lakehouse.md>) *(Data Architecture)*

## References

- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Storage The Heart of the Lakehouse › Avro
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Storage The Heart of the Lakehouse › Key features
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Storage The Heart of the Lakehouse › Similarities, differences, and use cases
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Future of Data Engineering › Avro
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Data Storage Formats for Data Lakes › Column-Based File Formats
- *Building Data Integration Solutions* — Jay Borthen — Key Concepts in Data Integration › ORC
- *Database Internals* — Alex Petrov — Introduction and Overview › Distinctions and Optimizations
- *Hadoop: The Definitive Guide* — Tom White — Hadoop IO › Other File Formats and Column-Oriented Formats
- *Hadoop: The Definitive Guide* — Tom White — Avro › In-Memory Serialization and Deserialization
- *Hadoop: The Definitive Guide* — Tom White — Avro › Avro Datafiles
- *Hadoop: The Definitive Guide* — Tom White — Avro › Avro in Other Languages

