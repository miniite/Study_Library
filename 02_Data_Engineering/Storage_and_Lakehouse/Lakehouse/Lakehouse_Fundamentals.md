---
id: "02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Lakehouse_Fundamentals"
level: 4
type: "concept"
topic: "02_Data_Engineering/Storage_and_Lakehouse/Lakehouse"
tools: ["AWS", "Databricks", "Apache_Spark", "Hadoop"]
---

# Lakehouse Fundamentals

**L4 Advanced** · [↑ Lakehouse](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A lakehouse puts a transactional table layer (Delta, Iceberg, Hudi) on top of cheap object storage so that one copy of the data can serve both warehouse-style SQL/BI and open-ended data-science and streaming workloads.

### 1. The big picture
Two older architectures each solved half the problem. A **data warehouse** stores structured, modelled data in a proprietary engine: fast SQL, ACID transactions, strong governance, but expensive, rigid, and poor with raw, semi-structured or unstructured data. A **data lake** stores raw files in their native format (CSV, JSON, images, logs, Parquet/ORC) on HDFS or cloud object storage: cheap, flexible and schema-on-read, but with no transactions, no enforced schema, weak governance, and a tendency to decay into a "data swamp" — a lake nobody trusts or can navigate.

Teams ended up running both: raw data in the lake, a curated copy in a warehouse, plus pipelines to keep them in sync. That means two copies, two security models, stale warehouse data and double the cost. The **lakehouse** removes the second system: keep the lake's open files and cheap storage, and add the warehouse's reliability features (ACID, schema enforcement, time travel, indexing/statistics, governance) through an **open table format** and a **catalog**.

A key point from the books: the table format alone does not make a successful lakehouse. Without a metadata catalog, schema-evolution rules, dataset ownership, quality validation, file compaction/lifecycle management and access control, query performance degrades, trust declines, and teams drift back to building separate marts — the lake just becomes a swamp with better file formats.

### 2. Core concepts and vocabulary
- **Data lake** — repository of raw data in native format on a distributed file system or object store; processing and metadata are layered on top.
- **Data warehouse** — structured, schema-on-write store optimised for analytical SQL.
- **Lakehouse** — lake storage + warehouse-style management through a table format, so BI, ML and streaming share one copy.
- **Object storage** — S3, ADLS, GCS: cheap, durable, effectively unlimited, but files are immutable and there is no native rename/transaction across files.
- **Open table format** — metadata protocol (Delta Lake transaction log, Iceberg metadata/manifests, Hudi timeline) that defines which files make up a table version.
- **Schema-on-read vs schema-on-write** — apply structure when querying (lake) versus when loading (warehouse). Lakehouses do schema-on-write *enforcement* on tables but keep raw zones schema-on-read.
- **Medallion layers** — Bronze (raw), Silver (cleaned/conformed), Gold (business-level, consumption-ready).
- **Compute/storage separation** — engines (Spark, Trino, Snowflake external tables, Athena) scale independently from the files.
- **Catalog / metastore** — service mapping table names to locations and schemas (see the catalogs lesson).
- **Data swamp** — a lake with no catalog, ownership or quality control.

### 3. How it works, step by step
1. **Ingest** — batch loads or streams (Kafka, Kinesis, CDC) land raw data in a Bronze zone, usually as-is plus ingestion metadata (source, load time).
2. **Store as columnar files** — data is written as Parquet (or ORC). Columnar layout + compression gives cheap scans and per-file column min/max statistics.
3. **Add the table layer** — a transaction log or manifest tree records, per commit, which files were added or removed. A write is "visible" only when its commit is atomically recorded, giving ACID semantics over immutable files.
4. **Register in a catalog** — the table gets a name, schema, owner, permissions; every engine resolves the same definition.
5. **Refine** — jobs read Bronze, validate, deduplicate and conform into Silver, then model into Gold (star schema, aggregates, feature tables).
6. **Serve** — a SQL engine reads Gold for BI; ML jobs read Silver/Gold directly with the same files; streaming jobs read the table's change feed.
7. **Maintain** — compaction, clustering/Z-order, snapshot expiry, vacuum, quality checks run continuously.

Why file-level statistics matter: each Parquet file (and the table format's manifests) stores min/max and null counts per column. For `WHERE region = 'Europe'`, the engine compares the predicate against the statistics and skips files that cannot contain a match — **data skipping** — without opening them. Partition pruning does the same at directory level; statistics do it at file and row-group level.

### 4. Worked example
```sql
-- Spark SQL on a Delta lakehouse
CREATE TABLE bronze.orders_raw (payload STRING, _src STRING, _ingested TIMESTAMP)
USING DELTA;

CREATE TABLE silver.orders (
  order_id BIGINT, customer_id BIGINT, amount DECIMAL(12,2),
  order_ts TIMESTAMP, order_date DATE
) USING DELTA PARTITIONED BY (order_date);

-- Upsert from a CDC batch (idempotent)
MERGE INTO silver.orders t
USING staged_orders s ON t.order_id = s.order_id
WHEN MATCHED AND s.op = 'U' THEN UPDATE SET *
WHEN NOT MATCHED THEN INSERT *;

-- Time travel and audit
SELECT * FROM silver.orders VERSION AS OF 42;
DESCRIBE HISTORY silver.orders;

-- Skip files using stats
SELECT sum(amount) FROM silver.orders
WHERE order_date = '2026-10-01' AND customer_id = 1001;
```
The `MERGE` rewrites only the files that contain matching rows and commits a new table version atomically; readers running at that moment keep seeing the previous version (snapshot isolation). `DESCRIBE HISTORY` shows each commit, which is the audit trail. The final query prunes to one partition directory and then skips files whose `customer_id` min/max range excludes 1001.

**Late-arriving data:** an event stamped three days ago arrives today. Do not append blindly to Gold; `MERGE` into the correct daily partition keyed on the natural/business key (or recompute the affected partition with a partition-scoped overwrite such as Spark's dynamic partition overwrite / Delta `replaceWhere`), so reruns do not double count.

### 5. Under the hood
- **Immutable files + log** — object stores cannot update a file in place. Updates write new files and the log marks old ones as removed. Readers reconstruct the table by replaying the log (with periodic checkpoints in Delta; snapshots → manifest lists → manifests in Iceberg).
- **Optimistic concurrency** — writers read the current version, produce files, and attempt an atomic commit; if another writer committed conflicting changes, the commit fails and is retried or aborted. This is why concurrent overwrites of the same partition need care.
- **Metadata scale** — the table format keeps statistics in metadata, so planning does not require listing millions of object-store keys (listing is slow and was a major cost of Hive-style tables).
- **Migration without rewrite** — Parquet files from a Hive-partitioned lake can often be *registered* into Iceberg or Delta (e.g., Iceberg `add_files` / `migrate`, Delta `CONVERT TO DELTA`) by writing metadata that points at the existing files; only metadata is created, not data.
- **Serving layer mix** — the books stress that no single engine is best at everything: a typical lakehouse combines a serverless SQL endpoint for ad hoc work, a columnar store for fast reporting, a relational database for complex queries and a time-series store for IoT, all reading the governed Gold data.

### 6. Performance and design implications
- Partition on low-cardinality columns that queries filter on (date, region); too many partitions create small files (see small-files lesson).
- Keep files in the ~128 MB–1 GB range; compact regularly.
- Sort/cluster (Z-order, Iceberg sort order, liquid clustering) on high-cardinality filter columns so min/max stats are tight and skipping is effective.
- Isolate workloads: separate compute clusters/warehouses for backfills vs dashboards so a heavy write does not hurt read latency; readers are safe from writers by snapshot isolation, but compute is still shared if you share a cluster.
- Multi-tenant design: partition or cluster by tenant, use per-tenant resource pools/quotas, and consider separate Gold tables for the largest tenants.
- Measure: scan bytes vs table size, files scanned vs files total, planning time, number of small files, snapshot count.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `OPTIMIZE` / `rewrite_data_files` | Compacts small files, optionally sorts | After many small appends |
| `VACUUM` / `expire_snapshots` | Removes unreferenced files / old versions | Control storage cost; respects retention |
| `MERGE INTO` | Atomic upsert/delete | CDC, late data, SCDs |
| `VERSION AS OF` / `TIMESTAMP AS OF` | Time travel | Audit, rollback, reproducing reports |
| `CONVERT TO DELTA` / `add_files` | Adopt existing Parquet | Migrating Hive-style lakes |
| `ALTER TABLE ... SET TBLPROPERTIES` | Retention, auto-optimise, constraints | Table-level policy |
| `mergeSchema` / schema enforcement | Controls schema evolution on write | Keep accidental columns out of Silver |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Mixed BI + ML + streaming on shared data | Only small, structured BI workloads — a plain warehouse is simpler |
| Need open formats and engine choice | Team lacks capacity for governance, compaction, catalog ownership |
| Large raw/semi-structured volumes | Extremely low-latency point lookups (use a serving DB) |
| Want one security/lineage story | Sub-second concurrent dashboards without a serving engine tuned for it |

### 9. Common mistakes
- Treating raw lake files as directly queryable business data → inconsistent, untrusted results → enforce layers and tables.
- No catalog, ownership or lineage → swamp → adopt a catalog and domain ownership from day one.
- Ignoring compaction and lifecycle → thousands of small files, slow queries and cost growth → schedule maintenance.
- Over-partitioning by high-cardinality columns → partition explosion → partition coarsely, cluster finely.
- Skipping quality checks because "it's just storage" → bad data reaches Gold → validate at layer boundaries.
- Assuming a table format removes the need for data modelling → Gold becomes an unusable pile → still model for consumption.

### 10. 🎯 Interview kit
**30-second answer:** A lakehouse keeps data as open Parquet files on object storage and adds an ACID table format and catalog on top, so the same single copy supports warehouse-style SQL, ML and streaming, with schema enforcement, time travel and governance instead of maintaining separate lake and warehouse copies.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.
Define lake vs warehouse limits, explain the transaction log/manifest and statistics, then trade-offs (operational burden, governance) and close with a migration or multi-tenant example.

**Likely follow-ups:**
- *How does a lakehouse handle consistency?* → atomic commit of a metadata/log entry plus snapshot isolation and optimistic concurrency.
- *How does file-level statistics help?* → min/max per column lets the planner skip files that cannot match the predicate.
- *Migrate Hive-partitioned Parquet to Iceberg without rewrite?* → register existing files in Iceberg metadata (`add_files`/`migrate`), then evolve partition spec afterwards.
- *Late-arriving event?* → MERGE or partition-scoped overwrite keyed on business key; idempotent.
- *Noisy tenant?* → partition/cluster by tenant, separate compute pools, quotas.
- *Dashboard latency during backfill?* → separate compute, snapshot isolation, write to staging then atomic swap/merge.

**Signals interviewers look for:** knows why a table format is needed on object storage; talks about governance and maintenance, not just formats; can contrast with warehouse honestly.

### 11. Check yourself
- [ ] Explain in three sentences why object storage alone cannot offer ACID.
- [ ] Draw the path of a row from Bronze to a dashboard and name where statistics and the catalog are used.
- [ ] List four reasons a lake becomes a swamp and the control that prevents each.
- [ ] Describe how you would migrate an existing Hive-partitioned lake to Iceberg with zero data rewrite.
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)
- [Databricks](<../../../TOOLS/Databricks/_index.md>)
- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)
- [Hadoop](<../../../TOOLS/Hadoop/_index.md>)

## Practice questions

- [Basic] [What is a data lake and how does it differ from a traditional data warehouse?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Q-11842_What_is_a_data_lake_and_how_does_it_differ_from_a_traditiona.md>)
- [Basic] [What is the role of an ingestion tool (like Kafka or Kinesis) feeding a data lake?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Q-11915_What_is_the_role_of_an_ingestion_tool_like_Kafka_or_Kinesis_.md>)
- [Intermediate] [How does file-level statistics help skip data during a lakehouse query?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Q-11887_How_does_file-level_statistics_help_skip_data_during_a_lakeh.md>)
- [Intermediate] [A late-arriving event lands three days after its event date. How do you correctly update the affected daily partition in the gold table?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Q-11955_A_late-arriving_event_lands_three_days_after_its_event_date_.md>)
- [Intermediate] [Describe how you would detect and address partition drift where old code writes to a deprecated partition scheme.](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Q-12033_Describe_how_you_would_detect_and_address_partition_drift_wh.md>)
- [Deep] [Walk me through how you would migrate a Hive-partitioned S3 data lake to Iceberg without a full rewrite of the underlying Parquet files.](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Q-11942_Walk_me_through_how_you_would_migrate_a_Hive-partitioned_S3_.md>)
- [Deep] [How would you design a multi-tenant lakehouse so one noisy tenant cannot degrade queries for others?](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Q-11973_How_would_you_design_a_multi-tenant_lakehouse_so_one_noisy_t.md>)
- [Deep] [Explain how you would keep read latency stable for a dashboard while a large backfill writes to the same Iceberg table.](<../../../PRACTICE/02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Q-12005_Explain_how_you_would_keep_read_latency_stable_for_a_dashboa.md>)

## Related concepts

- [End to End No Data Loss](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/End_to_End_No_Data_Loss.md>) *(Data Architecture)*
- [Real-time OLAP Serving Layer](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Real-time_OLAP_Serving_Layer.md>) *(Data Architecture)*
- [Data Warehouse Fundamentals](<../../Modeling_and_Warehousing/Data_Warehousing/Data_Warehouse_Fundamentals.md>) *(Data Warehousing)*
- [Tiered Lake Storage](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/_index.md>) *(Data Architecture)*
- [File and Table Formats for Features](<../../Pipelines_and_Integration/Pipeline_Orchestration/File_and_Table_Formats_for_Features.md>) *(Pipeline Orchestration)*
- [Table Format Internals](<../Table_Formats/Table_Format_Internals.md>) *(Table Formats)*

## References

- *Building Medallion Architectures* — Piethein Strengholt — The Evolution of Data Architecture › A Brief History of Data Lakes
- *Building Medallion Architectures* — Piethein Strengholt — The Evolution of Data Architecture › Moving Forward with Data Lakes
- *Building Medallion Architectures* — Piethein Strengholt — The Evolution of Data Architecture › A Brief History of Lakehouse Architecture
- *Building Medallion Architectures* — Piethein Strengholt — The Evolution of Data Architecture › The Rise of Lakehouse Architectures
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — The Evolution of Data Architectures › Introducing Data Lakes
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — The Evolution of Data Architectures › Data Lakehouse
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — The Evolution of Data Architectures › Data Lakehouse Benefits
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — The Evolution of Data Architectures › Implementing a Lakehouse
- *Azure Data Fundamentals* — Michael John Pena — Large-Scale Analytics › Data Lakes
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Data Engineering Fundamentals › 2. Data Engineering Fundamentals
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Architectural Patterns in Data Engineering › 3. Architectural Patterns in Data Engineering
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Storage Design Patterns in Data Engineering › 5. Storage Design Patterns in Data Engineering
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Evaluating storage systems: Key engineering considerations
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Chapter 3. Designing Good Data Architecture
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Data Lake
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Convergence, Next-Generation Data Lakes, and the Data Platform
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Introduction to Lakehouse Architecture › Chapter 1. Introduction to Lakehouse Architecture
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Introduction to Lakehouse Architecture › Why Do We Need a New Data Architecture?
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Introduction to Lakehouse Architecture › Lakehouse Architecture: A New Pattern
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Introduction to Lakehouse Architecture › The Lakehouse: Best of Both Worlds

