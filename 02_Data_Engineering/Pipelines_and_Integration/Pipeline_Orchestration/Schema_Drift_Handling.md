---
id: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Schema_Drift_Handling"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration"
tools: ["Apache_Spark", "Apache_Airflow", "Apache_Kafka", "dbt"]
---

# Schema Drift Handling

**L4 Advanced** · [↑ Pipeline Orchestration](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Source schemas change without asking you, so a pipeline must detect each change, decide whether to absorb it or stop, and tell the people who depend on the data.

### 1. The big picture
Upstream systems are owned by other teams and ship changes on their own schedules: a column is added, renamed, retyped, or dropped; a new table appears; an enum gains a value. Left alone, drift causes two kinds of failure. *Loud* failures break the job (a column not found). *Silent* failures are worse: a type widens, a value changes meaning, or a new column is ignored, and reports quietly become wrong.

Tools increasingly detect schema changes and auto-update target tables, but that is a mixed blessing — downstream models may still break, or performance may degrade because of the new shape. The engineering answer has three parts: **detect**, **respond automatically where safe**, and **communicate** where not. Think of it as a contract between producers and consumers that is checked in code.

### 2. Core concepts and vocabulary
- **Schema drift** — unplanned change in structure or type of incoming data.
- **Schema evolution** — a *planned*, supported change (adding a nullable column, widening a type).
- **Schema enforcement (schema-on-write)** — reject writes that don't match the table schema.
- **Schema-on-read** — store raw, apply structure at query time.
- **Backward compatible** — new schema can read data written with the old one (e.g., consumers upgraded first).
- **Forward compatible** — old schema can read data written with the new one.
- **Full compatibility** — both.
- **Schema registry** — a service that stores versioned schemas (Avro/Protobuf/JSON Schema) and enforces compatibility rules on producers.
- **Rescued data column** — a catch-all column holding fields that didn't fit the schema, so nothing is lost.
- **Data contract** — an explicit, versioned agreement of schema, semantics, and SLAs between producer and consumer.
- **Expand/contract** — migrate by first adding the new shape alongside the old, then removing the old after consumers move.

### 3. How it works, step by step
1. **Land raw first.** Keep an unmodified bronze/raw copy (file, JSON string, or evolving table). This makes any later fix replayable.
2. **Detect.** On each load compare the observed schema to the expected one (column set, types, nullability) — via a registry, an ingestion tool's change events, or a simple `df.schema` diff.
3. **Classify the change.**
   - *Additive nullable column* → usually safe to absorb.
   - *Type widening* (int → long) → often safe.
   - *Rename / drop / narrowing / semantic change* → breaking; stop and notify.
4. **Apply policy.** Per dataset: `evolve` (add column automatically), `rescue` (capture unexpected fields in a rescue column), `fail` (stop the job), or `quarantine` (divert non-conforming records).
5. **Propagate deliberately.** Update downstream models, views, and tests in a controlled release; don't let raw-layer evolution cascade into published gold tables automatically.
6. **Notify.** Alert the producer team and consumers with a diff and a deadline for breaking changes.
7. **Backfill if needed.** If a field was captured only in the rescue column or raw JSON, reprocess to populate the new column.

### 4. Worked example
```python
# Auto Loader (Databricks) – ingest JSON with controlled evolution
(spark.readStream.format("cloudFiles")
  .option("cloudFiles.format", "json")
  .option("cloudFiles.schemaLocation", "s3://lake/_schemas/orders")
  .option("cloudFiles.schemaEvolutionMode", "addNewColumns")   # or rescue / failOnNewColumns / none
  .load("s3://lake/raw/orders/")
  .writeStream
  .option("checkpointLocation", "s3://lake/_chk/orders_bronze")
  .option("mergeSchema", "true")
  .toTable("bronze.orders"))
```
```sql
-- Delta: enforcement blocks incompatible appends, evolution is explicit
ALTER TABLE silver.orders ADD COLUMNS (coupon_code STRING);

-- A downstream view insulates consumers from raw changes
CREATE OR REPLACE VIEW gold.orders_v1 AS
SELECT order_id, customer_id, CAST(amount AS DECIMAL(12,2)) AS amount, order_ts
FROM silver.orders;
```
What happens: when the source adds `coupon_code`, `addNewColumns` stops the stream once, records the new schema in `schemaLocation`, and the restart picks it up (bronze evolves). Silver changes only by an explicit `ALTER`, and gold is exposed through a versioned view so dashboards don't see surprise columns. If the source *renames* `amount` to `total`, the data lands in the rescued column or the schema evolves with `total` and nulls in `amount`; a null-rate check on `amount` fires and blocks promotion to silver.

### 5. Under the hood
- **Schema-on-write systems** (warehouses, Delta/Iceberg) check each write against the table schema; Delta rejects mismatched appends unless `mergeSchema` or `overwriteSchema` is set, so enforcement protects against *accidental* corruption.
- **Schema registries** version each subject; compatibility mode (BACKWARD, FORWARD, FULL, or the `_TRANSITIVE` variants, or NONE) is checked on registration. With Avro, adding a field with a default is backward compatible; removing a required field is not. Choosing BACKWARD means consumers can upgrade before producers; FORWARD means producers can go first.
- **Medallion layering**: bronze keeps source-shaped data and may evolve; silver standardizes to a conformed schema; gold is consumer-facing and changes only through reviewed releases.
- **Row-based vs columnar formats:** Parquet files carry their own schema; reading a directory of files with differing schemas requires merge-schema reads or table formats that track schema at the table level.

### 6. Performance and design implications
- Schema inference scans data; for large inputs provide explicit schemas or persist inferred ones (`schemaLocation`).
- Auto-evolution can create very wide tables of mostly nulls — wide JSON flattened into hundreds of columns slows scans; keep semi-structured data in a `VARIANT`/map/JSON column until fields are proven stable.
- A type change (e.g., `string` → `int`) may force a rewrite of existing data; plan for the cost.
- Measure: number of schema changes per source per month, time-to-detect, count of pipeline failures caused by drift, and consumer breakages.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `mergeSchema` (Delta write/read) | Allows additive schema change on write | Planned additive evolution |
| `cloudFiles.schemaEvolutionMode` | `addNewColumns`, `rescue`, `failOnNewColumns`, `none` | Ingestion with Auto Loader |
| `_rescued_data` column | Stores unparsed/unexpected fields | Never lose data on drift |
| Schema Registry compatibility level | BACKWARD / FORWARD / FULL / NONE | Event streams between teams |
| `ALTER TABLE ... ADD/ALTER COLUMN` | Explicit DDL change | Controlled evolution in silver/gold |
| Data contract tests (dbt, Great Expectations) | Assert columns/types/semantics | CI and runtime gates |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Auto-evolve (raw/bronze) — never block ingestion | Consumers read bronze directly |
| Fail fast — finance, regulated feeds | A cosmetic add-column halts nightly runs |
| Rescue column — unknown fields must be kept | Nobody ever reviews rescued data |
| Strict registry (FULL) — many independent teams | Producers need rapid iteration with a single consumer |
| Expand/contract migration — breaking renames | You can't coordinate or time-box the dual-write period |

### 9. Common mistakes
- Letting schema changes flow to gold automatically → dashboards change silently → gate promotion and use versioned views.
- Relying on inference → type flips with sampling (all ints in a sample, a string later) → explicit schemas.
- Treating a rename as a drop + add → history lost and nulls appear → map explicitly or migrate with expand/contract.
- No notification to consumers → surprise breakage → publish change logs and run contract tests.
- Dropping columns "since nobody uses them" → hidden consumers → check lineage first.

### 10. 🎯 Interview kit
**30-second answer:** I keep raw data as received, detect schema differences on every load, auto-absorb safe additive changes in the raw layer, fail or quarantine breaking ones, and promote to curated layers only through explicit, tested changes. For event streams I enforce compatibility in a schema registry, and I notify downstream owners of any change.

**Structure a longer answer:** clarify (batch/stream, who owns the source, tolerance) → mechanism (detect, classify, policy, propagate) → trade-offs (strict vs permissive) → scenario (producer renames a column).

**Likely follow-ups:**
- *Backward vs forward compatibility?* → who upgrades first and which side can read the other's data.
- *How do you migrate a feature table schema without breaking models?* → expand/contract, dual-write, versioned views, cut over per consumer.
- *What do you do when an auto-evolving column is wrong?* → rescue/raw replay and fix in silver.
- *Why still alert if tooling auto-evolves?* → semantics and performance may change even if loads succeed.

**Signals interviewers look for:** layered thinking, compatibility vocabulary, treating drift as a people/process problem as well as a technical one.

### 11. Check yourself
- [ ] Classify five schema changes as safe or breaking.
- [ ] Explain BACKWARD vs FORWARD compatibility with an Avro field addition.
- [ ] Describe an expand/contract migration of a renamed column.
- [ ] Say which layer should evolve automatically and which should not.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)
- [Apache Airflow](<../../../TOOLS/Apache_Airflow/_index.md>)
- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)
- [dbt](<../../../TOOLS/dbt/_index.md>)

## Practice questions

- [Basic] [What is data lineage at a beginner level and why is it helpful to know where a table's data came from?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13638_What_is_data_lineage_at_a_beginner_level_and_why_is_it_helpf.md>)
- [Intermediate] [How would you back up or keep the previous version of a table before a pipeline overwrites it, so you can recover from a bad run?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13641_How_would_you_back_up_or_keep_the_previous_version_of_a_tabl.md>)
- [Deep] [How would you schedule and tune Iceberg table maintenance (expire snapshots rewrite manifests compaction) in Airflow so table read performance doesn't degrade over months?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13655_How_would_you_schedule_and_tune_Iceberg_table_maintenance_ex.md>)
- [Deep] [How would you enforce backward and forward compatibility of event schemas in a Schema Registry so producers can't break your downstream Spark pipelines?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13663_How_would_you_enforce_backward_and_forward_compatibility_of_.md>)
- [Deep] [How do you design a slowly changing dimension type 2 pipeline in Spark that correctly closes out old records and inserts new versions with valid-from/valid-to?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13708_How_do_you_design_a_slowly_changing_dimension_type_2_pipelin.md>)
- [Deep] [How would you design a canary rollout for a changed dbt transformation so it runs on a sampled schema before promoting to production models?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13729_How_would_you_design_a_canary_rollout_for_a_changed_dbt_tran.md>)
- [Deep] [How would you enforce and evolve a data contract between an upstream producer team and your feature pipeline?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13210_How_would_you_enforce_and_evolve_a_data_contract_between_an_.md>)
- [Expert] [How do you handle schema evolution end-to-end when an upstream Avro schema in the Kafka registry adds a field your Spark streaming job and Iceberg sink must both absorb?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13662_How_do_you_handle_schema_evolution_end-to-end_when_an_upstre.md>)

## Related concepts

- [Lakehouse Fundamentals](<../../Storage_and_Lakehouse/Lakehouse/Lakehouse_Fundamentals.md>) *(Lakehouse)*
- [Pipeline Design Patterns](<../Pipeline_Design/Pipeline_Design_Patterns.md>) *(Pipeline Design)*

## References

- *Building Medallion Architectures* — Piethein Strengholt — Demystifying the Medallion Architecture › Schema Evolution and Management
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Building End-to-End Data Pipelines › 13. Building End-to-End Data Pipelines
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Operationalizing Data Pipelines › 14. Operationalizing Data Pipelines
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Detecting and handling schema changes in upstream and downstream systems
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — D
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Solution
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Flow Design Patterns › Fan-In
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Collecting, Cleaning, Transforming, and Testing Data › Managing Operational Data Transformations Across AWS Kinesis and Apache Kafka

