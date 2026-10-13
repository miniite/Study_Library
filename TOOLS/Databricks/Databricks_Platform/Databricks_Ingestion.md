---
id: "TOOLS/Databricks/Databricks_Platform/Databricks_Ingestion"
level: 4
type: "concept"
topic: "TOOLS/Databricks/Databricks_Platform"
tools: ["Databricks", "Apache_Spark", "AWS", "Azure", "Python"]
---

# Databricks Ingestion

**L4 Advanced** · [↑ Databricks Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Databricks ingestion means getting files and change feeds into Delta tables reliably — with Auto Loader or `COPY INTO` for files, Structured Streaming for event sources — on the right compute (job clusters or serverless), governed by Unity Catalog.

### 1. The big picture
The platform has a few moving parts, and ingestion questions are really questions about which part does what:

- **Workspace and notebooks** — the collaborative UI where code is written (Python, SQL, Scala, R via `%python`, `%sql`, … magic commands).
- **Compute** — clusters (driver + workers running Spark under a specific *Databricks Runtime*), SQL warehouses, and serverless compute.
- **Storage** — your cloud object storage (S3/ADLS/GCS). DBFS is only an abstraction layer that maps paths onto it; a "DBFS file" is a cloud object.
- **Governance** — Unity Catalog: a metastore with the three-level namespace `catalog.schema.table`, access control, and lineage.
- **Jobs / Workflows** — the scheduler that runs notebooks, wheels, SQL and DLT pipelines on *job clusters*.

Ingestion lives where these meet: you point compute at storage or a source, write into governed Delta tables (usually the *bronze* layer of a medallion design), and schedule it as a job.

### 2. Core concepts and vocabulary
- **Cluster** — a driver node (plans the job, holds the SparkSession, collects results) plus worker nodes (run tasks on partitions).
- **All-purpose (interactive) cluster** — shared, long-lived, for development; billed at a higher DBU rate.
- **Job cluster** — created for a run and terminated after; cheaper, isolated; use it for production.
- **SQL warehouse** — compute tuned for SQL/BI (Photon, result caching, concurrency scaling); no arbitrary Python.
- **Autoscaling / auto-termination** — grow and shrink workers with load; stop an idle cluster after N minutes (always keep it on).
- **DBU (Databricks Unit)** — the billing unit of processing per hour; cost = DBUs × rate for the workload type, **plus** the cloud VM cost.
- **Photon** — Databricks' native vectorized C++ query engine for SQL and DataFrame operations on **CPUs**; speeds up scan/join/aggregate-heavy work, gives little for Python UDF-bound or tiny jobs.
- **Auto Loader (`cloudFiles`)** — incremental file ingestion built on Structured Streaming that tracks new files in a checkpoint.
- **`COPY INTO`** — idempotent SQL command that loads *new* files from a path into a Delta table.
- **Unity Catalog** — central governance: catalogs, schemas, tables, volumes, permissions, lineage.
- **Lakehouse Federation** — query external databases in place without copying data.
- **`dbutils`** — notebook utility library: `dbutils.fs`, `dbutils.widgets`, `dbutils.secrets`.

### 3. How it works, step by step
**Auto Loader, the standard path for files landing in cloud storage:**
1. Files arrive in a landing path (`s3://…/raw/orders/`).
2. Auto Loader discovers them with either **directory listing** (incremental listing of the path) or **file notification** (cloud events → queue; scales to huge directories).
3. The set of already-seen files is recorded in the **checkpoint** (RocksDB state) — this is what gives exactly-once file processing and restart safety.
4. Each micro-batch reads the new files, applies the schema (inferred and stored under `cloudFiles.schemaLocation`), and appends to the bronze Delta table.
5. Unparseable or unexpected fields land in the `_rescued_data` column instead of failing the stream (rescue mode).
6. The trigger controls cadence: `availableNow=True` processes everything new and then stops (a scheduled "batch" run), while a processing-time trigger or continuous run keeps the stream alive.

**Choosing the compute:** interactive work → all-purpose cluster with auto-termination; scheduled production → job cluster or serverless jobs; BI/dashboards → SQL warehouse.

### 4. Worked example
```python
# bronze ingestion: JSON files -> Delta, incremental, schema-tracked
bronze = (spark.readStream
    .format("cloudFiles")
    .option("cloudFiles.format", "json")
    .option("cloudFiles.schemaLocation", "/Volumes/main/raw/_schemas/orders")
    .option("cloudFiles.schemaEvolutionMode", "addNewColumns")
    .load("/Volumes/main/raw/orders/"))

(bronze.writeStream
    .option("checkpointLocation", "/Volumes/main/raw/_checkpoints/orders_bronze")
    .trigger(availableNow=True)           # run as an incremental batch, then stop
    .toTable("main.bronze.orders"))
```
```sql
-- the SQL-only alternative for modest volumes
COPY INTO main.bronze.orders
FROM '/Volumes/main/raw/orders/'
FILEFORMAT = JSON
COPY_OPTIONS ('mergeSchema' = 'true');
```
Re-running either command only loads files not seen before. Look at the stream's progress (`numInputRows`, `numFilesOutstanding`) or `DESCRIBE HISTORY` of the target to confirm what each run added.

**Config-driven ingestion** (adding a source = adding a row, not code): keep a control table with `source_path, format, target_table, schema_hints, trigger`. One generic notebook loops over active rows and launches an Auto Loader stream per row; schema, paths and options all come from the row. Job parameters (widgets) select which group of sources to run.

### 5. Under the hood
- **Why Auto Loader beats listing the whole directory** — a plain `spark.read` re-lists everything each run; with millions of files the listing, not the read, dominates. Auto Loader keeps state and, in notification mode, never lists.
- **The driver is a bottleneck by design** — it plans and coordinates; `collect()`, `toPandas()` and large broadcast variables run on it. Putting heavy work in plain Python on the driver wastes the workers.
- **DBFS** — a mount-style path layer (`dbfs:/…`) over object storage; production code should use Unity Catalog **volumes** or external locations rather than DBFS root.
- **Unity Catalog lineage** — captured automatically at table and column level for queries run through UC-enabled compute; use it to answer "where did this table's data come from" and "what breaks if I change this column".
- **Runtime versions** — the Databricks Runtime bundles Spark, Delta and libraries; behaviour (and features such as deletion vectors) differs by version, so pin the runtime in job definitions.

### 6. Performance and design implications
- Use job clusters for production; interactive clusters left running are the commonest cost leak. Enable auto-termination and use spot workers for fault-tolerant batch with the driver on-demand.
- Prefer `availableNow` triggers on a schedule over an always-on stream when freshness needs are minutes-to-hours: you pay for compute only while it runs, and still get incremental, checkpointed semantics.
- Photon helps most on scan-/aggregation-/join-heavy SQL on Delta; to judge it, run representative queries with and without it and compare *total cost* (DBU rate is higher, runtime lower), not just runtime.
- Land raw data as-is in bronze (append-only, with ingestion metadata like source file and load time) and do cleaning in silver; this keeps ingestion fast and replayable.
- Environment drift ("works in the notebook, fails in the job"): compare Runtime version, installed libraries (cluster vs notebook-scoped), the identity running the job (service principal permissions, UC grants), and init scripts/Spark configs. Log and diff those first.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `cloudFiles` source | incremental file ingestion | files landing continuously or in batches |
| `cloudFiles.schemaEvolutionMode` | `addNewColumns`, `rescue`, `failOnNewColumns`, `none` | control drift handling |
| `cloudFiles.useNotifications` | file-notification discovery | very large/deep directories |
| `COPY INTO` | idempotent SQL loads | small/medium volumes, SQL-only teams |
| `trigger(availableNow=True)` | process backlog then stop | scheduled incremental batch |
| `dbutils.fs.ls/cp/rm` | file operations | exploration, small utility tasks |
| `dbutils.widgets` | notebook/job parameters | parameterised notebooks |
| `dbutils.secrets.get` | read secrets securely | credentials — never hardcode |
| cluster policy | restrict sizes/types/auto-termination | cost and governance control |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Auto Loader for any recurring file landing zone | one-off backfill of a static folder (`spark.read` or `COPY INTO` is simpler) |
| `COPY INTO` for simple, low-volume SQL-driven loads | huge file counts or complex schema evolution |
| Job clusters / serverless jobs for production | long interactive sessions needing library experiments |
| SQL warehouse for BI and ad-hoc SQL | custom Python/ML code (needs a cluster) |
| Lakehouse Federation to query a source in place | repeated heavy queries that would hammer the OLTP source — ingest into Delta instead |
| Config-driven ingestion for dozens of similar sources | one-off sources with bespoke logic |

### 9. Common mistakes
- Developing and running production on one shared all-purpose cluster → noisy neighbours and high cost → use job clusters with pinned runtimes.
- No auto-termination → idle clusters bill for days → always set it and use cluster policies.
- Using `collect()`/`toPandas()` on large data → driver OOM → aggregate or limit in Spark, write results to a table, use `display`/sampling.
- Re-reading the whole landing folder each run → cost grows with history → Auto Loader or `COPY INTO`.
- Hardcoding credentials in notebooks → leaks via git/lineage → secret scopes or UC credentials.
- Checking `count()` to "peek" at data → full job for no reason → use `display(df.limit(n))`.

### 10. 🎯 Interview kit
**30-second answer:** Databricks is a managed Spark-based lakehouse platform. For ingestion, files go through Auto Loader (or `COPY INTO`) into bronze Delta tables with a checkpoint that makes it incremental and restart-safe; production runs on job clusters or serverless, governed through Unity Catalog's `catalog.schema.table` namespace.

**Structure a longer answer:** clarify source type, volume and freshness → pick mechanism (Auto Loader / COPY INTO / streaming from Kafka) → schema and bad-record handling (rescued data, evolution mode) → compute and cost (job cluster, availableNow, spot) → governance and monitoring.

**Likely follow-ups:**
- *All-purpose vs job cluster?* → interactive/shared and pricier vs ephemeral/isolated and cheaper.
- *What is a DBU?* → processing-time billing unit; total cost also includes cloud VMs.
- *DBFS vs cloud storage?* → DBFS is an abstraction; the bytes live in object storage.
- *When does Photon help?* → vectorized CPU engine; helps scan/aggregate/join SQL; little for Python UDFs or tiny data.
- *Build a config-driven ingestion framework?* → control table + generic parameterised notebook + job per group.

**Signals interviewers look for:** cost awareness, clear separation of driver/worker work, knowing that exactly-once file ingestion comes from the checkpoint, and that you'd pick the simplest mechanism that meets the requirement.

### 11. Check yourself
- [ ] Explain what Auto Loader stores in its checkpoint and why that makes re-runs safe.
- [ ] Say when you would use `COPY INTO` instead of Auto Loader.
- [ ] Describe how you would cut an idle-cluster bill by half.
- [ ] Debug a notebook that works interactively but fails as a job.
- [ ] Sketch a control-table-driven ingestion design.
<!-- explainer:end -->

## Implementations (tools)

- [Databricks](<../_index.md>)
- [Apache Spark](<../../Apache_Spark/_index.md>)
- [AWS](<../../AWS/_index.md>)
- [Azure](<../../Azure/_index.md>)
- [Python](<../../Python/_index.md>)

## Practice questions

- [Basic] [What is a Databricks notebook and how does it differ from a normal Python script?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21750_What_is_a_Databricks_notebook_and_how_does_it_differ_from_a_.md>)
- [Basic] [What is DBFS (Databricks File System) and how is it related to underlying cloud storage like S3 or ADLS?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21759_What_is_DBFS_Databricks_File_System_and_how_is_it_related_to.md>)
- [Basic] [What are Databricks Units (DBUs) and how do they relate to cost?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21837_What_are_Databricks_Units_DBUs_and_how_do_they_relate_to_cos.md>)
- [Intermediate] [What is the difference between a SQL warehouse and an all-purpose cluster for running queries?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21809_What_is_the_difference_between_a_SQL_warehouse_and_an_all-pu.md>)
- [Intermediate] [What is data lineage and how does Unity Catalog help you see where a table's data came from?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21849_What_is_data_lineage_and_how_does_Unity_Catalog_help_you_see.md>)
- [Deep] [When does Photon actually help and when does it give you nothing, and how do you measure the DBU tradeoff of enabling it?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21873_When_does_Photon_actually_help_and_when_does_it_give_you_not.md>)
- [Deep] [A notebook job works interactively but fails in the scheduled job cluster with import/permission errors. How do you systematically debug environment drift?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21909_A_notebook_job_works_interactively_but_fails_in_the_schedule.md>)
- [Deep] [How would you build a metadata/config-driven ingestion framework in Databricks so onboarding a new source table is a config change, not new code?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21936_How_would_you_build_a_metadata_config-driven_ingestion_frame.md>)

## Related concepts

- [Lakehouse Fundamentals](<../../../02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Lakehouse_Fundamentals.md>) *(Lakehouse)*
- [Choosing Processing Engines](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Batch_vs_Streaming_Features/Choosing_Processing_Engines.md>) *(Pipeline Orchestration)*
- [Warehouse Vendor Exit Strategy](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Warehouse_Vendor_Exit_Strategy.md>) *(Data Architecture)*
- [Profiling Spark Stages](<../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Profiling_Spark_Stages.md>) *(Pipeline Orchestration)*
- [Distributed Joins in MPP Systems](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/_index.md>) *(Distributed Systems Core)*

## References

- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — CHAPTER 12Popular Technologies › Databricks
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — ETL and ELT › Databricks
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Operations and monitoring › Databricks Ganglia integration
- *Building Data Integration Solutions* — Jay Borthen — Comprehensive Integration Suites › Databricks
- *Building Data Integration Solutions* — Jay Borthen — Implementing a Batch Solution › Setting Up Databricks
- *Building Data Integration Solutions* — Jay Borthen — Implementing a Batch Solution › Connecting Databricks
- *Building Data Integration Solutions* — Jay Borthen — Implementing a Batch Solution › Conclusion
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Discovering Databricks › Chapter 1: Discovering Databricks
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Discovering Databricks › Introducing Databricks
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Discovering Databricks › Creating an Azure Databricks workspace
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Discovering Databricks › Core Databricks concepts
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Getting Started with Databricks › Chapter 1. Getting Started with Databricks
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Getting Started with Databricks › Understanding the Databricks Platform
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Getting Started with Databricks › High-Level Architecture of the Databricks Lakehouse
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Getting Started with Databricks › Deployment of Databricks Resources
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — The Modern Governance Stack › Databricks Data Intelligence Platform
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — The Modern Governance Stack › Databricks Platform Architecture
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Unity Catalog Under the Hood › Credential passthrough
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Unity Catalog Under the Hood › Databricks account
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Data Catalogs › Using Databricks
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Lakehouse in the Real World › A Question I Get from Most Customers: "Should We Use Databricks or Snowflake?"
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Lakehouse in the Real World › Third-party platform implementation
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — E › F

