---
id: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/File_and_Table_Formats_for_Features"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration"
tools: ["Apache_Spark", "Databricks", "Apache_Airflow", "Python"]
---

# File and Table Formats for Features

**L4 Advanced** · [↑ Pipeline Orchestration](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Store pipeline and feature data in columnar files (Parquet) managed by an open table format (Delta, Iceberg, Hudi) on object storage, so you get cheap scans, schema control, and atomic, versioned commits that many engines can share.

### 1. The big picture
Where and how data is stored decides how fast it can be read, how safely it can be updated, and how locked-in you are. Three layers matter:

1. **Storage** — usually object storage (S3, GCS, ADLS): cheap, durable, effectively unlimited, but files are immutable and there are no transactions or indexes.
2. **File format** — how rows and columns are laid out inside files: row-based (CSV, JSON, Avro) or columnar (Parquet, ORC).
3. **Table format** — metadata on top of files that turns a pile of Parquet files into a *table*: schema, partitions, snapshots, atomic commits (Delta Lake, Apache Iceberg, Apache Hudi).

For feature tables and analytics, the common stack is object storage + Parquet + an open table format. The format decision is a long-lived one — it affects governance, tooling and lock-in — so it is worth reasoning about explicitly.

### 2. Core concepts and vocabulary
- **Object storage** — key/value blob store; files cannot be modified in place, only replaced.
- **Row-oriented format** — stores each record contiguously (CSV, JSON, Avro); good for writes and record-level access.
- **Columnar format** — stores each column contiguously (Parquet, ORC); good for scans reading a few columns, better compression.
- **Compression/encoding** — dictionary, run-length, Snappy/ZSTD; shrink storage and I/O.
- **Schema evolution** — adding/renaming/changing columns without rewriting everything.
- **Table format** — metadata layer (transaction log or snapshots) providing ACID commits, time travel, schema enforcement.
- **Snapshot isolation** — readers see a consistent version while writers commit new ones.
- **Full vs incremental load** — rewrite the whole dataset vs apply only changes.
- **Landing zone** — object-storage area where raw pipeline data first arrives.

### 3. How it works, step by step
**Why Parquet beats CSV for analytics**
1. A query asks for 3 of 100 columns.
2. Parquet reads only those column chunks (column pruning).
3. Row-group statistics (min/max) let it skip chunks that cannot match a filter (predicate pushdown).
4. Columns of the same type compress far better, so fewer bytes cross the network.

**What a table format adds**
1. A writer produces new Parquet data files in object storage (invisible so far).
2. It commits by atomically adding an entry to the table's log/manifest listing those files.
3. A reader opens the latest snapshot, listing exactly the files that belong to that version — never a half-written set.
4. Updates and deletes are expressed as new files plus metadata marking old files removed; compaction later cleans up.
5. Old snapshots remain until expired, enabling time travel and rollback.

**Choosing a format for a feature store**
1. Decide access pattern: large scans for training, point lookups for serving, streaming upserts.
2. Decide engines: which tools must read and write?
3. Pick the table format with the best support in that ecosystem, and standardize.

### 4. Worked example
```sql
-- Create a feature table on object storage in Delta format
CREATE TABLE features.customer_30d (
  customer_id BIGINT,
  snapshot_date DATE,
  orders_30d INT,
  spend_30d DECIMAL(12,2)
) USING DELTA
PARTITIONED BY (snapshot_date);

-- Incremental upsert of the latest features
MERGE INTO features.customer_30d t
USING staging.customer_30d_today s
  ON t.customer_id = s.customer_id AND t.snapshot_date = s.snapshot_date
WHEN MATCHED THEN UPDATE SET *
WHEN NOT MATCHED THEN INSERT *;

-- Reproduce what a model saw on a past date
SELECT * FROM features.customer_30d VERSION AS OF 42;
```
Reading it: the table is just Parquet files plus a `_delta_log`; `MERGE` commits atomically, so a concurrent reader sees either the old or the new version. Time travel lets you rebuild a training set exactly as it was. In Iceberg the same semantics come from snapshot metadata and manifest files, so a Spark reader and a streaming writer can operate together without seeing partial commits.

### 5. Under the hood   *(optional at L4)*
- **Parquet layout:** file → row groups → column chunks → pages, with a footer holding schema and statistics.
- **Delta:** ordered JSON commit files plus periodic Parquet checkpoints; concurrency by optimistic checks on the log.
- **Iceberg:** tree of metadata file → manifest list → manifests → data files; supports hidden partitioning, partition evolution, engine-neutral design.
- **Hudi:** emphasizes record-level upserts with indexes and copy-on-write vs merge-on-read table types.
- **Gotcha: same schema, different files.** Files claiming the same schema can still fail if column order, nullability, or physical types differ (e.g. int vs long, string vs timestamp); table formats enforce schema on write, plain file directories do not.
- **Small files problem:** frequent small writes create many tiny files; compaction (`OPTIMIZE`, `rewrite_data_files`) is required.

### 6. Performance and design implications
- Target file sizes of roughly 128 MB–1 GB; avoid thousands of tiny files.
- Partition by a low-cardinality column used in filters (date); over-partitioning hurts.
- Use clustering/sorting (Z-order, liquid clustering, sort order) for secondary filter columns.
- Full loads are simple but expensive and can disrupt readers; incremental loads with merge are efficient but need keys and idempotency.
- Retention: expire snapshots and vacuum, or storage grows indefinitely.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE TABLE … USING DELTA / ICEBERG` | Create managed table format | Any lake table needing ACID |
| `MERGE INTO` | Upsert changes | Incremental loads, CDC |
| `OPTIMIZE` / `rewrite_data_files` | Compact small files | After streaming or frequent writes |
| `VACUUM` / `expire_snapshots` | Delete unreferenced files | Control storage |
| Time travel (`VERSION AS OF`, `TIMESTAMP AS OF`) | Query history | Reproducibility, rollback |
| Parquet compression codec (Snappy/ZSTD) | Size vs CPU trade-off | Tune storage cost |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Delta — Spark/Databricks-centric, simple operations | Need broad multi-engine neutrality (check support/UniForm) |
| Iceberg — multi-engine (Spark, Trino, Flink, warehouses), partition evolution | Team lacks catalog/metadata management skills |
| Hudi — heavy record-level upserts/streaming ingestion | Simpler batch workloads that need less machinery |
| Plain Parquet — read-only, immutable datasets | Concurrent writers, updates, deletes |
| Full load — small tables, no reliable change tracking | Large tables; costly and disruptive |
| Incremental — reliable keys/timestamps | Source deletes not captured |

### 9. Common mistakes
- Using CSV/JSON for analytics → slow, large, schema drift → Parquet in the curated layers.
- Over-partitioning → many small files and slow planning → coarser partitions.
- Skipping compaction → read performance decays → schedule optimize.
- Never expiring snapshots → storage bloat → retention policy (balanced with time-travel needs).
- Choosing a format on hype → lock-in surprises → decide with engine/governance requirements.

### 10. 🎯 Interview kit
**30-second answer:** I keep data on object storage as Parquet because columnar layout gives column pruning and compression, and I put an open table format like Delta or Iceberg on top for ACID commits, schema enforcement, time travel and safe concurrent reads and writes. Format choice depends on engines, workload, and lock-in tolerance.

**Structure a longer answer:** clarify workloads → file format → table format → partition/compaction → governance/time travel → decision criteria.

**Likely follow-ups:**
- *Full vs incremental?* → full for small/unreliable change tracking; incremental with merge for large.
- *How does Iceberg let readers and writers coexist?* → immutable snapshots; commit swaps metadata pointer atomically.
- *Why do compatible-looking files error?* → column order/type/nullability mismatches.
- *How standardize on one format?* → pick by engine support, ecosystem, governance and exit options.

**Signals interviewers look for:** layered thinking (storage/file/table), awareness of compaction and retention, and neutral comparison.

### 11. Check yourself
- [ ] Explain column pruning and predicate pushdown in Parquet.
- [ ] Describe what an atomic commit looks like in a table format.
- [ ] State when a plain Parquet directory is enough.
- [ ] List three maintenance tasks for a lakehouse table.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)
- [Databricks](<../../../TOOLS/Databricks/_index.md>)
- [Apache Airflow](<../../../TOOLS/Apache_Airflow/_index.md>)
- [Python](<../../../TOOLS/Python/_index.md>)

## Practice questions

- [Basic] [What is a data lake versus a data warehouse and where do raw logs usually land first?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13415_What_is_a_data_lake_versus_a_data_warehouse_and_where_do_raw.md>)
- [Basic] [What is the difference between a data warehouse and a data lakehouse in simple terms?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13608_What_is_the_difference_between_a_data_warehouse_and_a_data_l.md>)
- [Basic] [Why is Parquet often a better choice than CSV as the output format of a pipeline stage?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13615_Why_is_Parquet_often_a_better_choice_than_CSV_as_the_output_.md>)
- [Basic] [What is the difference between the transform step in ETL versus ELT and where does the transform happen in each?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13640_What_is_the_difference_between_the_transform_step_in_ETL_ver.md>)
- [Intermediate] [What is the difference between a full refresh and an incremental load in a data pipeline?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13425_What_is_the_difference_between_a_full_refresh_and_an_increme.md>)
- [Intermediate] [What is a staging area in a pipeline and why load into staging before moving to the final table?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13604_What_is_a_staging_area_in_a_pipeline_and_why_load_into_stagi.md>)
- [Deep] [What are the trade-offs between Parquet Delta and Iceberg for storing enriched alert history consumed by both correlation training and incident analytics?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13503_What_are_the_trade-offs_between_Parquet_Delta_and_Iceberg_fo.md>)
- [Deep] [How do you handle time-travel and rollback with Iceberg or Delta when a bad pipeline run corrupted the gold table that BI is already querying?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13730_How_do_you_handle_time-travel_and_rollback_with_Iceberg_or_D.md>)

## Related concepts

- [Lakehouse Fundamentals](<../../Storage_and_Lakehouse/Lakehouse/Lakehouse_Fundamentals.md>) *(Lakehouse)*
- [Table Format Internals](<../../Storage_and_Lakehouse/Table_Formats/Table_Format_Internals.md>) *(Table Formats)*
- [Tiered Lake Storage](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/_index.md>) *(Data Architecture)*

## References

- *Building Medallion Architectures* — Piethein Strengholt — Demystifying the Medallion Architecture › Processing Full Data Loads
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Data integration and interoperability
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Raw Ingredients of Data Storage
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Object lookup
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Moving Data with Object Storage
- *Data Pipelines Pocket Reference* — James Densmore — Data Ingestion Loading Data › Incremental Versus Full Loads
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Access Controls and Permissions Model › Fine-grained access controls
- *Azure Data Fundamentals* — Michael John Pena — Data Storage Fundamentals › CSV: The Universal Translator

