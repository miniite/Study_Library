---
id: "02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Columnar_Formats_in_the_Lake"
level: 4
type: "concept"
topic: "02_Data_Engineering/Storage_and_Lakehouse/Lakehouse"
tools: ["Databricks", "Apache_Spark"]
---

# Columnar Formats in the Lake

**L4 Advanced** · [↑ Lakehouse](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** In a lake, columnar open files (mainly Parquet, sometimes ORC) are the shared physical layer that lets Spark, Trino, warehouses and ML tools all read the same data without conversion — and table formats turn those files into reliable tables.

### 1. The big picture
A data lake stores files in cheap object storage. A **lakehouse** adds warehouse-like behaviour — schemas, ACID transactions, time travel, performance — on top of those files. The foundation is a columnar file format. Proprietary warehouse engines (BigQuery, Snowflake and others) also store data in columnar form, but in their own internal format: querying is fast inside the engine, while sharing with external engines means deserializing and exporting to an exchangeable format. Open columnar files in the lake avoid that: any engine that reads Parquet can read the data in place.

So the practical question in the lake is "which columnar format, how is it organized, and what manages it?" Parquet is the default: built-in schema, native nested data support, portable, excellent read and write performance, usually compressed with speed-optimized codecs such as Snappy. ORC is a similar columnar format that was very popular with Hive and is still widely used but supported less in modern cloud tools. This lesson places these formats within lakehouse layout and shows why the file format alone isn't enough.

### 2. Core concepts and vocabulary
- **Data lake** — object-store-based repository of raw and processed files.
- **Lakehouse** — lake + table-management layer giving ACID, schema enforcement and performance features.
- **Table format** — metadata layer (Delta Lake, Apache Iceberg, Apache Hudi) that defines which Parquet files form a table version.
- **Parquet / ORC** — columnar data file formats.
- **Medallion layers** — bronze (raw), silver (cleaned/conformed), gold (business-level aggregates).
- **Partitioning** — directory/key layout by column(s) enabling pruning.
- **Compaction / OPTIMIZE** — rewriting many small files into fewer large ones.
- **Z-order / clustering** — physically co-locating related values to improve skipping on multiple columns.
- **Schema enforcement vs evolution** — rejecting vs permitting changes to table schema (e.g., adding columns; type widening within a type category).
- **Data skipping** — using file/column statistics to avoid reading files.

### 3. How it works, step by step
A typical lakehouse flow:

1. **Ingest** raw data to the bronze layer — often as-is (JSON/CSV) or Avro for fidelity.
2. **Convert** to Parquet inside a table format (e.g., Delta): files are written plus a transaction log entry listing them.
3. **Partition and cluster** by the main filter (date), optionally Z-order by secondary columns.
4. **Transform** bronze → silver → gold with Spark/SQL; each write is an atomic commit (new files added, old files logically removed).
5. **Query**: the engine reads the table metadata, prunes partitions and files using stats, then reads only required column chunks from the selected Parquet files.
6. **Maintain**: compact small files, expire old snapshots/vacuum, update statistics.

**Evidence of benefit** (published comparison for a 1 TB dataset on object storage):

| Storage | Size | Query time | Data scanned | Cost |
|---|---|---|---|---|
| CSV | 1 TB | 236 s | 1.15 TB | ~$5.75 |
| Parquet | 130 GB | 6.8 s | 2.5 GB | ~$0.01 |

The ratios (≈87% smaller, ≈34× faster, ≈99% less scanned) come from column pruning, compression and stats-based skipping together.

### 4. Worked example
Create a partitioned Delta (Parquet-backed) table in Spark SQL and see the files:

```sql
CREATE TABLE silver.events (
    event_id   STRING,
    user_id    BIGINT,
    event_type STRING,
    amount     DECIMAL(12,2),
    event_ts   TIMESTAMP,
    event_date DATE
) USING DELTA
PARTITIONED BY (event_date);

INSERT INTO silver.events
SELECT event_id, user_id, event_type, amount, event_ts, CAST(event_ts AS DATE)
FROM bronze.events_raw;

-- maintenance
OPTIMIZE silver.events ZORDER BY (user_id);
DESCRIBE DETAIL silver.events;     -- numFiles, sizeInBytes
SELECT * FROM silver.events VERSION AS OF 3;   -- time travel
```
How to read it: after `OPTIMIZE`, `numFiles` drops and average file size rises; a filter on `event_date` and `user_id` should show a small fraction of files read in the query plan/UI ("files pruned"). If you list the table's storage path you will see `_delta_log/` plus directories per `event_date` containing `.parquet` files — the table format is just metadata over ordinary Parquet.

Schema notes: type widening within the same category (e.g., int → long) can be handled by the table format; changing category (string → boolean) requires a full overwrite with schema overwrite enabled.

### 5. Under the hood
- **Why open columnar beats proprietary in polyglot shops:** with Parquet in the lake, Spark, Presto/Trino, Athena, Snowflake external tables and ML libraries read the same bytes. With a proprietary format, each tool pair needs export/import.
- **Metadata separation:** Parquet files are immutable; transaction logs/manifests define the current set. "Delete" writes a new snapshot without the file (or uses deletion vectors) and physical removal happens later at vacuum/expiry.
- **Statistics layers:** Parquet footers hold per-file stats; table formats gather per-file stats in their metadata (Delta log/Iceberg manifests) so the engine can prune files *without opening them*.
- **Layout vs stats:** skipping works only if data is clustered; Z-order/sort and partitioning make min/max ranges selective.
- **Compression choices:** Snappy is the common default for speed in lake workloads; ZSTD gives better ratios for cold data; benefit varies, with 75%+ reductions vs text formats typical.
- **ORC:** stripes, built-in indexes and bloom filters; Hive ACID heritage; many cloud warehouses can import/read but not export ORC.
- **Catalog dependency:** to find tables by name, engines need a catalog (see Lake Catalogs).

### 6. Performance and design implications
- Files too small → metadata and request overhead; too big → poor parallelism and expensive rewrites. Aim for hundreds of MB.
- Partition on low-cardinality filter columns (date, region); use clustering for high-cardinality ones.
- Keep only needed history for time travel; retention affects storage cost.
- Use column projection and avoid `SELECT *`; wide gold tables are fine in columnar form.
- Measure with engine metrics: files read vs total, bytes scanned, shuffle spill.
- Streaming writers produce small files → schedule compaction or enable auto-optimize features.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE TABLE … USING DELTA/ICEBERG` | Table format over Parquet | Lakehouse tables |
| `OPTIMIZE … ZORDER BY` / Iceberg `rewrite_data_files` | Compaction and clustering | Skewed/small files |
| `VACUUM` / `expire_snapshots` | Remove unreferenced files | Control storage cost |
| `DESCRIBE DETAIL` / `SHOW … FILES` | Inspect files/size | Diagnose layout |
| `overwriteSchema` option | Replace schema on overwrite | Breaking type changes |
| `spark.sql.parquet.compression.codec` | Codec selection | Cost/speed |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Parquet + Delta/Iceberg: multi-engine, ACID, large analytics | Tiny datasets where a single DB suffices |
| ORC: Hive-centric platform | You need broad managed-service export support |
| Proprietary warehouse storage: single engine, best managed performance | Many external engines need the same data |
| Fine partitioning | Cardinality explosion → small files |

### 9. Common mistakes
- Treating a folder of Parquet files as a "table" without a table format → no atomic commits, partial reads → use Delta/Iceberg/Hudi.
- Partitioning by high-cardinality columns (user_id) → millions of tiny files → partition by date, cluster by user.
- Never running maintenance → small files and stale snapshots → scheduled OPTIMIZE/VACUUM.
- Changing column types freely → read failures across files → use schema evolution rules.
- Choosing ORC by habit for a Spark/cloud stack → reduced tool support.

### 10. 🎯 Interview kit
**30-second answer:** In a lakehouse, data lives as open columnar files — typically Parquet — in object storage, and a table format such as Delta or Iceberg adds transactions, schema control and stats on top. Columnar layout gives pruning, compression and skipping, and being open means every engine can read the same files without export.

**Structure a longer answer:** why columnar → Parquet vs ORC vs proprietary → table format role → layout (partition, cluster, file size) → maintenance → trade-offs.

**Likely follow-ups:**
- *Why Parquet over a warehouse's native format?* → Interoperability and no data movement across engines; native may be faster in-engine.
- *What does Delta add to Parquet?* → Transaction log, ACID, time travel, schema enforcement, file-level stats.
- *Parquet vs ORC?* → Equivalent in design; Parquet has broader cloud/tool support.
- *Small files?* → Compaction cadence trade-off between freshness and scan efficiency.

**Signals interviewers look for:** you connect file format to table format, layout and cost; you mention maintenance and interoperability.

### 11. Check yourself
- [ ] Explain the relationship between Parquet files and a Delta/Iceberg table.
- [ ] Give three reasons columnar open files reduce cost.
- [ ] Decide partition and clustering columns for an events table.
- [ ] Describe a maintenance schedule for a streaming-fed table.
- [ ] State when ORC is a reasonable choice.
<!-- explainer:end -->

## Implementations (tools)

- [Databricks](<../../../TOOLS/Databricks/_index.md>)
- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)

## Practice questions

- [Basic] [Why is columnar file format like Parquet preferred for analytical data in a lake?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Q-11850_Why_is_columnar_file_format_like_Parquet_preferred_for_analy.md>)
- [Basic] [What are the three main open table formats used in lakehouses and can you name them?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Q-11851_What_are_the_three_main_open_table_formats_used_in_lakehouse.md>)
- [Basic] [Why is Parquet described as a self-describing file format?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Q-11864_Why_is_Parquet_described_as_a_self-describing_file_format.md>)
- [Basic] [What is the difference between ORC and Parquet at a high level?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Q-11865_What_is_the_difference_between_ORC_and_Parquet_at_a_high_lev.md>)
- [Intermediate] [How do open table formats bring ACID guarantees to files sitting in object storage?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Q-11856_How_do_open_table_formats_bring_ACID_guarantees_to_files_sit.md>)
- [Intermediate] [When would you use Avro instead of Parquet in a data lake pipeline?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Q-11866_When_would_you_use_Avro_instead_of_Parquet_in_a_data_lake_pi.md>)
- [Intermediate] [Why can't you simply UPDATE a row in a plain Parquet file on a lake?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Q-11885_Why_can_t_you_simply_UPDATE_a_row_in_a_plain_Parquet_file_on.md>)
- [Intermediate] [Why is Parquet compression important and which codecs are commonly used?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Q-11921_Why_is_Parquet_compression_important_and_which_codecs_are_co.md>)

## Related concepts

- [Parquet Internals](<../File_Formats/Parquet_Internals.md>) *(File Formats)*

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Future of Data Engineering › Parquet
- *Building Data Integration Solutions* — Jay Borthen — Key Concepts in Data Integration › Parquet
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Introduction to Lakehouse Architecture › Open file formats
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Introduction to Lakehouse Architecture › Open table formats
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Storage The Heart of the Lakehouse › File Formats
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Storage The Heart of the Lakehouse › Parquet
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and DataFrames Introduction to Built-in Data Sources › Parquet
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Essential Delta Lake Operations › Parquet Conversions
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Architecting Your Lakehouse › Open file format
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Getting Started with Delta Lake › Parquet Files
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Conclusion › P

