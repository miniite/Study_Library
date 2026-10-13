---
id: "TOOLS/Apache_Spark/PySpark/Schema_Evolution_in_PySpark"
level: 4
type: "concept"
topic: "TOOLS/Apache_Spark/PySpark"
tools: ["Apache_Spark", "Databricks"]
---

# Schema Evolution in PySpark

**L4 Advanced** · [↑ PySpark](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Plain Parquet/JSON/ORC folders have no table-level schema, so a PySpark pipeline must define schemas explicitly, enforce them on write, and evolve them deliberately (Delta `mergeSchema` / `ALTER TABLE`) so upstream changes fail fast or are absorbed safely instead of silently corrupting data.

### 1. The big picture
Source schemas change: columns are added, renamed, retyped, dropped. Spark file formats store a schema *per file*, not per table, so nothing stops a job from writing a file with a different schema into an existing folder — readers then see mixed layouts, nulls or errors. Table formats such as **Delta Lake** (and Iceberg/Hudi) store the schema as table-level metadata in the transaction log and check every write against it.

Schema evolution is the discipline of (1) *declaring* the expected schema, (2) *detecting* drift at ingest, (3) *choosing* per change whether to reject, quarantine, or evolve, and (4) *propagating* evolution to downstream consumers without breaking them.

### 2. Core concepts and vocabulary
- **Schema inference** — Spark scans data to guess types; costs an extra pass and can guess differently across runs.
- **Explicit schema** — a `StructType` you pass to `.schema(...)`; faster (no scan) and deterministic.
- **Schema enforcement (on write)** — reject writes whose schema doesn't match the table (Delta default).
- **Schema evolution** — intentionally changing the table schema (add column, widen type).
- **`mergeSchema`** — Delta write option that adds new columns found in the incoming data to the table schema.
- **`overwriteSchema`** — replaces the whole table schema on overwrite (dangerous).
- **Read modes** — `PERMISSIVE` (keep row, put raw text in `_corrupt_record`), `DROPMALFORMED` (silently drop), `FAILFAST` (error on first bad record).
- **Backward / forward compatible change** — old readers still read new data / new readers still read old data (adding a nullable column is backward compatible).
- **High-water mark** — max processed `updated_at`/id; next run reads only rows above it.

### 3. How it works, step by step
Handling an upstream change safely:
1. **Pin the contract**: store the expected schema (StructType JSON, a registry, or table DDL) in version control.
2. **Read with the explicit schema** and a read mode that fits (`PERMISSIVE` + `_corrupt_record` for ingestion; `FAILFAST` when any bad row is unacceptable).
3. **Compare** incoming schema vs. target: new columns, missing columns, type changes.
4. **Classify**: additive nullable → evolve; type widening (int→long) → allowed; narrowing/rename/drop → reject or manual migration.
5. **Evolve deliberately**: `ALTER TABLE ADD COLUMNS` or one-off write with `mergeSchema=true` (or `spark.databricks.delta.schema.autoMerge.enabled` for MERGE).
6. **Notify consumers** (catalog/contract change) and update downstream selects (prefer explicit column lists over `SELECT *`).
7. **Backfill** if the new column should be populated for history.

### 4. Worked example
```python
from pyspark.sql.types import *
from delta.tables import DeltaTable

schema = StructType([
    StructField("loan_id",     LongType(),    False),
    StructField("funded_amnt", IntegerType(), True),
    StructField("paid_amnt",   DoubleType(),  True),
    StructField("addr_state",  StringType(),  True),
])

raw = (spark.read
       .schema(schema.add("_corrupt_record", StringType()))   # keep bad rows visible
       .option("mode", "PERMISSIVE")
       .option("columnNameOfCorruptRecord", "_corrupt_record")
       .json("s3://lake/raw/loans/2026-10-12/"))

good = raw.filter("_corrupt_record IS NULL").drop("_corrupt_record")
bad  = raw.filter("_corrupt_record IS NOT NULL")
bad.write.mode("append").format("delta").save("s3://lake/quarantine/loans")

# Delta enforces the table schema: a new column 'closed' in 'good' would raise AnalysisException
(good.write.format("delta").mode("append").save("s3://lake/silver/loans"))

# Intentional evolution (reviewed change):
(good_with_closed.write.format("delta").mode("append")
     .option("mergeSchema", "true").save("s3://lake/silver/loans"))
```
Reading it: without `mergeSchema`, Delta aborts *before* committing any file ("A schema mismatch detected when writing to the Delta table"), so the table is never half-written. With it, the transaction log records a new schema version; existing rows read the new column as `NULL`. Check with `DESCRIBE HISTORY` and `DESCRIBE DETAIL`.

### 5. Under the hood
- **Parquet** files carry their own footer schema; Spark's `mergeSchema` read option for plain Parquet unions the footers (expensive: reads every footer) and fails on incompatible types. Delta keeps the authoritative schema in `_delta_log` JSON/checkpoint so no footer scan is needed.
- **Type changes:** Delta allows a small set of safe widenings (version-dependent); anything else needs a rewrite (`overwriteSchema`) because existing Parquet files still contain the old type.
- **Column mapping** (Delta `delta.columnMapping.mode = name`) allows rename/drop without rewriting data by mapping logical names to physical column ids.
- **Inference cost:** `inferSchema` on CSV/JSON does a full extra scan; on TBs it can dominate runtime — one book experiment saw a CSV read go from slow to ~5 seconds just by supplying a schema.
- **Streaming:** a streaming query's checkpoint stores state/offsets tied to the plan; many logic or schema changes (stateful operators, aggregation keys) require a new checkpoint, so schema changes in streaming need a migration plan.
- **MERGE evolution:** `MERGE INTO` only adds new columns if `autoMerge` is enabled; it never silently drops columns.

### 6. Performance and design implications
- Always provide schemas for large CSV/JSON reads; infer once on a sample, then freeze.
- Use `PERMISSIVE` + quarantine for ingestion; `FAILFAST` on curated layers.
- Prefer additive-only changes in contracts; treat rename = add + backfill + deprecate + drop.
- Frequent schema changes create many log versions and can break dashboards; gate with review.
- Incremental pipelines: persist the high-water mark in a control table, filter `updated_at > :hwm` with an overlap window and dedup via MERGE to tolerate late rows.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `.schema(StructType)` | Explicit schema on read | Any large/untrusted file read |
| `.option("mode", "FAILFAST")` | Error on malformed row | Strict ingestion |
| `columnNameOfCorruptRecord` | Capture raw bad record | Quarantine pattern |
| `.option("mergeSchema","true")` | Add new columns on write | Reviewed additive change |
| `spark.databricks.delta.schema.autoMerge.enabled` | Evolve in MERGE | Upserts with new columns |
| `ALTER TABLE t ADD COLUMNS (...)` | Explicit evolution | Controlled migration |
| `DESCRIBE HISTORY t` | See schema/commit history | Audit & rollback |
| `RESTORE TABLE t TO VERSION AS OF n` | Roll back | Bad evolution |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `mergeSchema` on bronze/raw tables | Governed gold tables with contracts |
| Strict enforcement + alert | Sources change daily and you need availability (use quarantine/evolve) |
| Explicit schema | Exploratory work on unknown data |
| Column mapping | Old readers/engines that don't support it |

### 9. Common mistakes
- Enabling `mergeSchema` globally → typos and bad upstream columns become permanent table columns → enable per reviewed job.
- Relying on inference → type flips between runs (int vs. string) → define the schema.
- `SELECT *` into curated tables → upstream additions leak downstream → list columns.
- No rollback plan → can't undo a bad evolution → rely on Delta history/`RESTORE`, test in staging.
- Treating `DROPMALFORMED` as safe → silent data loss → quarantine instead.

### 10. 🎯 Interview kit
**30-second answer:** "Plain Parquet has no table-level schema, so I use explicit schemas on read, a table format like Delta for enforcement on write, quarantine malformed rows, and evolve only through reviewed additive changes with `mergeSchema` or `ALTER TABLE`, with history to roll back."

**Structure a longer answer:** why drift happens → detection → classification (additive/widening/breaking) → mechanism (enforce, evolve) → consumer impact → rollback/monitoring.

**Likely follow-ups:**
- *When is `mergeSchema` dangerous?* → governed tables; a bad upstream column becomes permanent.
- *PERMISSIVE vs DROPMALFORMED vs FAILFAST?* → keep-with-flag / silently drop / fail immediately.
- *Why does explicit schema help cost and correctness?* → avoids the inference scan and type drift.
- *SCD2 with Delta MERGE?* → close old row (`end_date`, `is_current=false`) and insert new version; handle out-of-order and deleted records.
- *Delta ACID on object storage?* → optimistic concurrency over an ordered transaction log; commits are atomic file-puts.

**Signals interviewers look for:** distinguishing enforcement from evolution, treating schema as a contract, a rollback story.

### 11. Check yourself
- [ ] Write a StructType and read JSON with a corrupt-record column.
- [ ] Explain why a Delta write with an extra column fails by default.
- [ ] List three changes that are safe and three that are breaking.
- [ ] Describe how you would roll back a bad `mergeSchema` change.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../_index.md>)
- [Databricks](<../../Databricks/_index.md>)

## Practice questions

- [Intermediate] [Explain the difference between spark.read.parquet lazy schema inference and providing an explicit schema — why does explicit schema matter for cost and correctness?](<../../../PRACTICE/TOOLS/Apache_Spark/PySpark/Q-36415_Explain_the_difference_between_spark_read_parquet_lazy_schem.md>)
- [Intermediate] [When designing a daily incremental pipeline how do you use a watermark/high-water-mark column in PySpark to only read new source rows?](<../../../PRACTICE/TOOLS/Apache_Spark/PySpark/Q-36423_When_designing_a_daily_incremental_pipeline_how_do_you_use_a.md>)
- [Intermediate] [How do you handle corrupt or malformed records when reading JSON/CSV in PySpark — explain PERMISSIVE DROPMALFORMED and FAILFAST modes and _corrupt_record.](<../../../PRACTICE/TOOLS/Apache_Spark/PySpark/Q-36450_How_do_you_handle_corrupt_or_malformed_records_when_reading_.md>)
- [Deep] [What is predicate pushdown and column pruning in Spark's Catalyst optimizer and how do you know from explain() output that they were applied?](<../../../PRACTICE/TOOLS/Apache_Spark/PySpark/Q-36408_What_is_predicate_pushdown_and_column_pruning_in_Spark_s_Cat.md>)
- [Deep] [Explain how MERGE INTO on a Delta table works from PySpark and what makes it more expensive than an append — how do you keep it performant?](<../../../PRACTICE/TOOLS/Apache_Spark/PySpark/Q-36435_Explain_how_MERGE_INTO_on_a_Delta_table_works_from_PySpark_a.md>)
- [Deep] [How does Delta Lake give you ACID transactions on object storage and what role does the transaction log play when two writers commit concurrently?](<../../../PRACTICE/TOOLS/Apache_Spark/PySpark/Q-36436_How_does_Delta_Lake_give_you_ACID_transactions_on_object_sto.md>)
- [Deep] [When would you run OPTIMIZE and ZORDER on a Delta table from PySpark and how do you decide the ZORDER columns?](<../../../PRACTICE/TOOLS/Apache_Spark/PySpark/Q-36437_When_would_you_run_OPTIMIZE_and_ZORDER_on_a_Delta_table_from.md>)
- [Deep] [Explain how VACUUM works on Delta and why aggressive vacuuming can break a long-running reader or time-travel query.](<../../../PRACTICE/TOOLS/Apache_Spark/PySpark/Q-36438_Explain_how_VACUUM_works_on_Delta_and_why_aggressive_vacuumi.md>)

## Related concepts

- [Pandas vs Polars vs PySpark](<../../Python/Python_Language/Pandas_vs_Polars_vs_PySpark.md>) *(Python Language)*
- [Spark Execution Caching and Debugging](<../Spark_Core/Spark_Execution_Caching_and_Debugging.md>) *(Spark Core)*
- [Shuffle Partition Tuning](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Shuffle_Partition_Tuning.md>) *(Big Data Processing)*

## References

- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Apache Spark's Structured APIs › Common DataFrame Operations
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and DataFrames Introduction to Built-in Data Sources › In Python
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Building Reliable Data Lakes with Apache Spark › Enforcing Schema on Write to Prevent Data Corruption
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Getting Started with Delta Lake › Chapter 2. Getting Started with Delta Lake
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Getting Started with Delta Lake › Creating and Running a Spark Program: helloDeltaLake
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Getting Started with Delta Lake › Create a Spark instance with the builder\
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Getting Started with Delta Lake › Conclusion
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Learning about Machine Learning and Graph Processing in Databricks › EDA
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Big Data Analytics › Experiment 2
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Installing Delta Lake › PySpark Shell
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Installing Delta Lake › Setting Up Delta Lake with Apache Spark
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Installing Delta Lake › Python
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Installing Delta Lake › Bash\
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Chapter 6. Building Production Pipelines › SQL support
- *Building Medallion Architectures* — Piethein Strengholt — Construct the Bronze Layer › Infer base parameters from the pipeline context
- *Building Medallion Architectures* — Piethein Strengholt — Construct the Bronze Layer › Fetch parameters from Data Factory
- *Building Medallion Architectures* — Piethein Strengholt — Construct the Bronze Layer › Delete any existing data
- *Building Medallion Architectures* — Piethein Strengholt — Construct the Bronze Layer › Function to Load YAML File

