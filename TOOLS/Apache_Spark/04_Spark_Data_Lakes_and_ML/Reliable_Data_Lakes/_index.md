---
id: "TOOLS/Apache_Spark/04_Spark_Data_Lakes_and_ML/Reliable_Data_Lakes"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Reliable Data Lakes with Spark

**L4 Advanced** · [↑ Spark Data Lakes and Machine Learning](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A plain data lake on object storage is cheap but unreliable (no transactions, no schema control); table formats such as Delta Lake add a transaction log on top of Parquet files to give ACID writes, schema enforcement, updates/deletes and time travel — the "lakehouse".

### 1. The big picture
"Data lake 1.0" meant dumping raw files (first on HDFS, then on cloud object stores like S3/ADLS/GCS) and using engines such as Spark or Hive to query them. Storage was cheap and scalable but the book describes the result as frequently a *data swamp*: no schema discipline, no safe way to update or delete, concurrent jobs corrupting each other, and partial writes leaving half-finished output.

The response is the **lakehouse** (Databricks' term): keep data in open files on object storage but add warehouse-like management — ACID transactions, schema management, incremental updates/deletes, table history — through a metadata/transaction layer. The main open implementations are **Delta Lake**, **Apache Iceberg** and **Apache Hudi**; Spark is a first-class engine for all three. This lesson uses Delta Lake as the worked example.

### 2. Core concepts and vocabulary
- **Data lake** — raw files in object storage/HDFS, schema-on-read.
- **Data swamp** — a lake without governance: undiscoverable, untrusted data.
- **Data lakehouse** — lake storage + warehouse features (ACID, schema, governance, BI performance).
- **ACID** — atomicity, consistency, isolation, durability of table operations.
- **Transaction log** — ordered record of table changes (Delta: `_delta_log/`).
- **Schema enforcement** — writes that don't match the table schema are rejected.
- **Schema evolution** — controlled changes (`mergeSchema`).
- **Time travel** — query older table versions by version number or timestamp.
- **MERGE / UPDATE / DELETE** — row-level modifications on file-based tables.
- **Compaction (OPTIMIZE) / VACUUM** — rewrite small files / remove unreferenced files.
- **Copy-on-write vs merge-on-read** — how row-level changes are physically applied.

### 3. How it works, step by step
**Why plain Parquet is unsafe:** a job writing 100 files and failing at file 60 leaves a directory with 60 files; readers see a partial result. Two concurrent writers overwrite each other; "overwrite" deletes then writes, with a visible gap.

**Delta Lake write path**
1. Writer reads the latest table version from the log.
2. It writes new data files (Parquet) to the table directory — not yet visible.
3. It attempts to commit a new JSON entry `_delta_log/000…N.json` listing `add` and `remove` actions, using atomic put-if-absent on the store.
4. If another writer committed version N first, the loser checks for logical conflicts (optimistic concurrency); if none, it retries at N+1, otherwise it fails.
5. Readers always reconstruct a snapshot from the log (latest checkpoint + subsequent JSON files), so they see either the old or the new version, never a partial write.
6. Every ~10 commits a **checkpoint** Parquet file summarises state to speed up reads.

**Update/delete/merge:** find affected files (using file-level min/max stats), rewrite those files with changes, commit `remove` of old files + `add` of new ones — all in one atomic commit.

### 4. Worked example
```python
from delta.tables import DeltaTable
from pyspark.sql import functions as F

path = "s3://lake/silver/customers"

# 1. create / append with enforced schema
(spark.read.json("/landing/customers/2024-06-01")
      .write.format("delta").mode("append").save(path))

# 2. upsert (CDC feed) atomically
updates = spark.read.parquet("/landing/customer_changes/2024-06-02")
(DeltaTable.forPath(spark, path).alias("t")
   .merge(updates.alias("s"), "t.customer_id = s.customer_id")
   .whenMatchedUpdateAll()
   .whenNotMatchedInsertAll()
   .execute())

# 3. audit and time travel
spark.sql(f"DESCRIBE HISTORY delta.`{path}`").show(truncate=False)
old = spark.read.format("delta").option("versionAsOf", 3).load(path)

# 4. maintenance
spark.sql(f"OPTIMIZE delta.`{path}` ZORDER BY (customer_id)")
spark.sql(f"VACUUM delta.`{path}` RETAIN 168 HOURS")
```
What to inspect: the `_delta_log` folder (one JSON per commit), `DESCRIBE HISTORY` (operation, user, metrics such as numTargetRowsUpdated), and that a second run of the same MERGE changes no rows (idempotent). A write with an extra column fails with a schema mismatch unless `mergeSchema=true`.

### 5. Under the hood   *(L4: optional)*
- **Log actions:** `metaData` (schema, partition columns), `add` (path, size, stats, partitionValues), `remove` (tombstone), `commitInfo`, `protocol` (reader/writer versions).
- **Isolation level:** writes are serializable by default per table; reads are snapshot-isolated.
- **Object-store caveats:** requires atomic "put if absent" or a coordination service (older S3 needed a DynamoDB log store for multi-cluster writers).
- **Iceberg/Hudi differences:** Iceberg uses manifest lists/metadata files and hidden partitioning; Hudi adds record-level indexing and merge-on-read tables; all give snapshots and time travel.
- **Small-file problem:** streaming and frequent appends create many tiny files; `OPTIMIZE` (bin-packing) and auto-compaction fix it.
- **Retention:** `VACUUM` deletes files no longer referenced older than the retention window; time travel beyond that fails. Default 7 days; shortening it risks breaking long-running readers.

### 6. Performance and design implications
- **Data skipping:** per-file min/max stats let queries skip files; Z-ordering/clustering co-locates related values.
- **Partitioning:** partition by low-cardinality columns used in filters (date); too many partitions → small files.
- **Streaming + batch on one table:** Delta supports Structured Streaming as source and sink with exactly-once semantics through the log.
- **Write amplification:** MERGE rewrites whole files; frequent small upserts into large files are costly — consider merge-on-read formats or batching.
- **Medallion layout:** bronze (raw), silver (cleaned), gold (aggregates) gives clear contracts and replay.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `df.write.format("delta")` | Write Delta table | Replace Parquet dirs |
| `DeltaTable.merge(...)` / `MERGE INTO` | Upsert | CDC, SCD |
| `DESCRIBE HISTORY t` | Audit log | Debug, governance |
| `VERSION AS OF` / `TIMESTAMP AS OF` | Time travel | Reproduce, rollback |
| `RESTORE TABLE t TO VERSION AS OF n` | Roll back | Bad write |
| `OPTIMIZE t [ZORDER BY (c)]` | Compact + cluster | Many small files |
| `VACUUM t RETAIN n HOURS` | Delete stale files | Storage cleanup |
| `ALTER TABLE ... ADD CONSTRAINT CHECK` | Enforce data rules | Quality gates |
| `option("mergeSchema","true")` | Evolve schema | Controlled additions |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Delta/Iceberg/Hudi table | You need updates, deletes, concurrent writers, history |
| Plain Parquet | Write-once immutable data, single writer, simplest tooling |
| Delta | Databricks/Spark-centric stack |
| Iceberg | Multi-engine (Trino, Flink, Snowflake) neutrality |
| Hudi | Heavy upsert/streaming ingestion with indexing |
| Warehouse (Snowflake/BigQuery) | Mostly SQL BI with little need for open files |

### 9. Common mistakes
- **Plain `overwrite` on Parquet paths** → readers see missing data; failure loses old data → use a table format with atomic commits.
- **Too-aggressive VACUUM** → time travel and long queries break → keep ≥ 7 days.
- **Over-partitioning** → millions of small files → partition coarsely, OPTIMIZE.
- **Mixing writers without a log store on S3** → corruption risk → use supported coordination.
- **Schema drift ignored** → silent breaking changes → enforcement + explicit evolution.
- **Treating the lake as a dumping ground** → swamp → catalog, ownership, quality checks.

### 10. 🎯 Interview kit
**30-second answer:** "Plain data lakes lack transactions and schema control, so partial writes and concurrent jobs corrupt data. Table formats like Delta Lake add a transaction log over Parquet to provide ACID commits, schema enforcement, upserts and time travel — that's the lakehouse."

**Structure a longer answer:** lake 1.0 problems → transaction-log mechanism → features (MERGE, time travel, schema) → operations (OPTIMIZE/VACUUM) → format comparison.

**Likely follow-ups:**
- *How does Delta get atomicity on S3?* → Atomic creation of the next numbered log file; files are invisible until committed.
- *What happens with two concurrent writers?* → Optimistic concurrency; conflict check; retry or fail.
- *Lakehouse vs warehouse?* → Open files + separate compute vs integrated managed storage; converging.
- *Why VACUUM?* → Removes unreferenced files; limits time travel.

**Signals interviewers look for:** log-based reasoning, trade-offs between formats, operational maintenance.

### 11. Check yourself
- [ ] Explain why a failed Spark write to plain Parquet is dangerous.
- [ ] Describe how a Delta commit becomes visible atomically.
- [ ] Write a MERGE that upserts CDC records idempotently.
- [ ] Say what happens to time travel after VACUUM.
- [ ] Compare Delta, Iceberg and Hudi in two sentences each.
<!-- explainer:end -->

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Convergence, Next-Generation Data Lakes, and the Data Platform
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Building Reliable Data Lakes with Apache Spark › Building Lakehouses with Apache Spark and Delta Lake
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Building Reliable Data Lakes with Apache Spark › Data Lakes
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › The Data Lake
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Building Reliable Data Lakes with Apache Spark › Delta Lake
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Building Reliable Data Lakes with Apache Spark › Enforcing Schema on Write to Prevent Data Corruption
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Building Reliable Data Lakes with Apache Spark › Limitations of Data Lakes
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Modeling Streaming Data › Update patterns
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — D
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Data Lake

