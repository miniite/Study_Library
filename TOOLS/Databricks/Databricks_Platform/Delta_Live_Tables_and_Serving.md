---
id: "TOOLS/Databricks/Databricks_Platform/Delta_Live_Tables_and_Serving"
level: 4
type: "concept"
topic: "TOOLS/Databricks/Databricks_Platform"
tools: ["Databricks", "Apache_Spark"]
---

# Delta Live Tables and Serving

**L4 Advanced** · [↑ Databricks Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** On Databricks you serve data from Delta tables (optimised with `OPTIMIZE`, `MERGE`, partition/cluster choices) and can build the pipeline that produces them declaratively with Delta Live Tables (DLT), where you state *what* each table is and the platform handles orchestration, dependencies, incremental processing and data-quality expectations.

### 1. The big picture
Two concerns meet here. First, **the tables themselves**: how to write, upsert, partition and maintain Delta tables so that gold-layer consumers (Databricks SQL, BI, downstream jobs) get fast, correct reads. Second, **how those tables get built**: either hand-written notebooks scheduled as Jobs, or **Delta Live Tables**, a declarative framework where each dataset is a query and the engine builds the dependency graph, manages checkpoints and retries, and records data-quality metrics.

Choosing between them is a real design decision: DLT removes plumbing and gives built-in expectations and CDC helpers; hand-written notebooks give maximum control for unusual logic.

### 2. Core concepts and vocabulary
- **Streaming table** — a Delta table fed incrementally from an append-only source; processes each input row once.
- **Materialized view** — a table defined by a query over other datasets, refreshed (incrementally where possible) by the pipeline.
- **Expectation** — a declared data-quality rule on a dataset: `EXPECT (valid_id IS NOT NULL) ON VIOLATION DROP ROW | FAIL UPDATE`, or warn only.
- **Pipeline** — the deployed set of DLT datasets with a target schema, compute, and settings.
- **Triggered vs continuous mode** — run once and stop vs run always-on with low latency.
- **Development vs production mode** — development reuses clusters and does not retry on failure; production restarts and retries.
- **Full refresh** — rebuild a dataset from scratch (clears checkpoints and data).
- **`APPLY CHANGES INTO`** — DLT's declarative CDC (SCD type 1 or 2) from a change feed.
- **`MERGE INTO`** — Delta upsert: match on a key, then update/insert/delete.
- **`OPTIMIZE` / `VACUUM`** — compaction vs deletion of unreferenced files (different jobs; do not confuse).
- **Change Data Feed (CDF)** — Delta's row-level change log used to propagate changes incrementally to downstream tables.
- **Photon / serverless SQL warehouse** — the serving compute for queries.

### 3. How it works, step by step
**DLT pipeline execution (triggered):**
1. The pipeline parses all datasets (Python `@dlt.table` / SQL `CREATE OR REFRESH STREAMING TABLE`) and builds a DAG from the references between them.
2. Compute is provisioned (or reused in development mode).
3. Datasets run in dependency order; streaming tables consume only new input since their checkpoint; materialized views are recomputed or incrementally maintained.
4. Expectations are evaluated per row; rows are kept, dropped, or fail the update depending on the declared action; counts are written to the event log.
5. Results are committed to Delta tables; the event log records lineage, metrics and errors.
6. The pipeline stops (triggered) or keeps running (continuous).

**A hand-written upsert (`MERGE`) for comparison:**
1. Read the incoming batch (e.g. latest CDC rows per key).
2. `MERGE INTO target USING updates ON target.id = updates.id` — match scan finds affected files.
3. Matched rows update, unmatched insert, flagged rows delete; new files are written and committed atomically.

### 4. Worked example
```sql
-- DLT SQL: bronze -> silver with expectations, then a gold aggregate
CREATE OR REFRESH STREAMING TABLE bronze_orders
AS SELECT * FROM STREAM read_files('/Volumes/main/raw/orders/', format => 'json');

CREATE OR REFRESH STREAMING TABLE silver_orders (
  CONSTRAINT valid_id     EXPECT (order_id IS NOT NULL) ON VIOLATION DROP ROW,
  CONSTRAINT positive_amt EXPECT (amount >= 0)          -- warn only: keep + count
)
AS SELECT order_id, customer_id, CAST(amount AS DECIMAL(12,2)) AS amount, order_ts
   FROM STREAM bronze_orders;

CREATE OR REFRESH MATERIALIZED VIEW gold_daily_revenue
AS SELECT date(order_ts) AS d, sum(amount) AS revenue
   FROM silver_orders GROUP BY date(order_ts);
```
CDC with the declarative API:
```sql
CREATE OR REFRESH STREAMING TABLE customers_silver;

APPLY CHANGES INTO customers_silver
FROM STREAM customers_cdc
KEYS (customer_id)
SEQUENCE BY updated_at
APPLY AS DELETE WHEN op = 'D'
STORED AS SCD TYPE 1;
```
`SEQUENCE BY` is what makes it correct when events arrive out of order — a hand-written `MERGE` must dedupe to the latest event per key (window `row_number()` over `updated_at DESC`) *before* merging, or the merge errors/produces wrong rows when one key appears twice in a batch.

The hand-written equivalent:
```sql
MERGE INTO customers_silver t
USING (SELECT * FROM (
         SELECT *, row_number() OVER (PARTITION BY customer_id ORDER BY updated_at DESC) rn
         FROM customers_cdc) WHERE rn = 1) s
ON t.customer_id = s.customer_id
WHEN MATCHED AND s.op = 'D' THEN DELETE
WHEN MATCHED THEN UPDATE SET *
WHEN NOT MATCHED AND s.op <> 'D' THEN INSERT *;
```

### 5. Under the hood
- **Streaming source rules** — a Delta table used as a stream source must be append-only. An `UPDATE`/`DELETE`/`MERGE` upstream produces the "source table has been updated non-append" failure. Fixes: read with `ignoreChanges`/`skipChangeCommits` (accepting that changes won't propagate), or read the **Change Data Feed** to propagate updates and deletes explicitly; if you must reprocess, start a new checkpoint.
- **How `APPLY CHANGES` differs from `MERGE`** — it manages sequencing, tombstones for deletes, and SCD type 2 history columns for you, and uses an internal table plus a view. Limitations: the target is managed by the pipeline (you can't write to it elsewhere), and some features are less flexible than custom merge logic.
- **Expectations are row-level** — implemented as filters/assertions over each micro-batch; "drop" keeps a count, "fail" aborts the update and rolls back. To *quarantine* instead of discard, route violating rows to a separate table with the inverted rule.
- **Delta table sizing** — Databricks autotunes `delta.targetFileSize` based on table size (smaller files for merge-heavy tables, larger for big tables) when `delta.tuneFileSizesForRewrites` / autotune is enabled.
- **Time travel vs retention** — logs default to 30 days and removed files to 7 days; a `VACUUM` shorter than the longest reader needs can break running queries.

### 6. Performance and design implications
- **Partition choice**: partition by a low-cardinality, always-filtered column (date, region). A high-cardinality partition column creates an explosion of tiny files and slow metadata operations. Cluster/Z-order for the rest.
- **MERGE slowdowns after growth**: diagnose which of three causes — (a) match scan reading the whole target because the `ON` clause lacks a partition/cluster predicate, (b) small-file count inflating task overhead, (c) skew on a hot key. Fix with predicate pruning, `OPTIMIZE`, and key salting or splitting respectively.
- **Triggered vs continuous**: continuous keeps a cluster running 24/7; if freshness requirement is "within 15–60 minutes", triggered on a schedule is much cheaper.
- **Spot instances**: workers on spot and driver on-demand cuts cost; a long pipeline must tolerate worker loss (retries, checkpoints). Driver loss kills the run, which is why it stays on-demand.
- **Serving**: BI should read gold tables through a SQL warehouse with Photon and result caching; keep gold narrow, pre-aggregated and clustered by dashboard filters.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE OR REFRESH STREAMING TABLE` | incremental append-only dataset | bronze/silver ingestion |
| `CREATE OR REFRESH MATERIALIZED VIEW` | query-defined dataset | gold aggregates |
| `CONSTRAINT … EXPECT … ON VIOLATION` | data-quality rule | validate and drop/fail/quarantine |
| `APPLY CHANGES INTO … SEQUENCE BY` | declarative CDC | SCD1/SCD2 from change feeds |
| `MERGE INTO` | custom upsert | logic DLT can't express |
| `OPTIMIZE` / `VACUUM` | compact / clean | scheduled table maintenance |
| `delta.enableChangeDataFeed` | expose row-level changes | silver → gold incremental propagation |
| Pipeline mode: triggered/continuous | cadence | cost vs latency trade-off |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| DLT — standard bronze/silver/gold pipelines, built-in quality and lineage, small team | highly custom control flow, tight integration with external orchestration, or features DLT doesn't support |
| Hand-written notebooks + Jobs — maximum flexibility, existing frameworks | you'd re-implement checkpoints, retries and quality metrics already provided |
| `APPLY CHANGES` — ordered CDC with deletes and optional history | non-standard merge semantics, multi-table side effects |
| Triggered mode — freshness in minutes–hours | strict seconds-level latency |
| Continuous mode — low-latency serving needs | cost-sensitive pipelines with relaxed SLAs |
| Spot workers — fault-tolerant, restartable work | long single-shot jobs with no retry tolerance |

### 9. Common mistakes
- Confusing `OPTIMIZE` with `VACUUM` → expecting storage to shrink after compaction → OPTIMIZE adds new files; VACUUM removes old ones after retention.
- Merging a batch with duplicate keys → ambiguous-match error or wrong data → dedupe to latest per key first (or use `APPLY CHANGES`).
- Reading a mutated Delta table as a stream without handling changes → stream fails → use CDF or `skipChangeCommits`, understanding the consequences.
- Partitioning by high-cardinality columns → too many small files → choose date-like columns and cluster others.
- Leaving pipelines in continuous mode "by default" → surprise bills → decide per freshness SLA.
- Validating only at the end → bad rows silently flow to gold → put expectations at silver entry and quarantine rejects with alerting on the event-log counts.

### 10. 🎯 Interview kit
**30-second answer:** DLT is declarative pipelines on Delta: you define streaming tables and materialized views as queries, the platform builds the DAG, handles incremental processing, retries and checkpoints, and enforces data-quality expectations; `APPLY CHANGES INTO` handles CDC. For custom logic I'd use notebooks with `MERGE`.

**Structure a longer answer:** clarify freshness, source type, and quality needs → choose DLT vs notebooks → design layers and expectations → decide partition/cluster and maintenance (`OPTIMIZE`, `VACUUM`) → cost mode (triggered vs continuous, spot).

**Likely follow-ups:**
- *`APPLY CHANGES` vs `MERGE`?* → declarative with sequencing, deletes and SCD2 vs manual dedupe and full control.
- *Stream fails with "updated non-append"?* → upstream was modified; use CDF or `skipChangeCommits`, or reprocess with a new checkpoint.
- *Quarantine bad rows instead of failing?* → expectations with `DROP ROW` plus a rejects table using the inverse condition; alert on metrics.
- *Same `COUNT` returns different results twice?* → table changed between queries (new commits); pin a version or use snapshot reads.
- *Ingest Kafka/Debezium CDC exactly-once into silver?* → streaming with checkpoint, dedupe by key and sequence, idempotent upsert via `APPLY CHANGES` or `foreachBatch` + `MERGE`.

**Signals interviewers look for:** you know the limits of DLT, can reason about CDC ordering and duplicates, and tie design to cost and freshness.

### 11. Check yourself
- [ ] Write a streaming silver table with a drop-row and a warn-only expectation.
- [ ] Explain what `SEQUENCE BY` protects against.
- [ ] Diagnose a MERGE that went from 15 minutes to 2 hours.
- [ ] State what happens to a stream when an upstream `UPDATE` hits its Delta source.
- [ ] Decide triggered vs continuous for a "ready by 6 a.m." SLA.
<!-- explainer:end -->

## Implementations (tools)

- [Databricks](<../_index.md>)
- [Apache Spark](<../../Apache_Spark/_index.md>)

## Practice questions

- [Basic] [How would you write a DataFrame out as a Delta table in Databricks?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21777_How_would_you_write_a_DataFrame_out_as_a_Delta_table_in_Data.md>)
- [Intermediate] [What is partitioning a Delta table by a column and how does it help query performance?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21779_What_is_partitioning_a_Delta_table_by_a_column_and_how_does_.md>)
- [Deep] [What is a slowly changing dimension (SCD Type 2) and how could you implement it with MERGE in Delta?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21843_What_is_a_slowly_changing_dimension_SCD_Type_2_and_how_could.md>)
- [Deep] [How would you implement GDPR right-to-be-forgotten deletes across a Delta table that has time travel and 30 days of retained history?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21865_How_would_you_implement_GDPR_right-to-be-forgotten_deletes_a.md>)
- [Deep] [Explain how you'd handle schema evolution safely in a bronze layer that ingests from a source that adds/renames columns without warning.](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21900_Explain_how_you_d_handle_schema_evolution_safely_in_a_bronze.md>)
- [Deep] [Walk me through debugging a job that succeeds but produces wrong numbers — how do you isolate a logic bug from a data bug in a medallion pipeline?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21917_Walk_me_through_debugging_a_job_that_succeeds_but_produces_w.md>)
- [Deep] [Explain how DLT handles CDC with APPLY CHANGES INTO and what it does under the hood versus a hand-written MERGE.](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21939_Explain_how_DLT_handles_CDC_with_APPLY_CHANGES_INTO_and_what.md>)
- [Expert] [A stream reading from a Delta source fails with 'source table has been updated non-append'. What caused it and how do you recover safely?](<../../../PRACTICE/TOOLS/Databricks/Databricks_Platform/Q-21891_A_stream_reading_from_a_Delta_source_fails_with_source_table.md>)

## Related concepts

- [Table Format Internals](<../../../02_Data_Engineering/Storage_and_Lakehouse/Table_Formats/Table_Format_Internals.md>) *(Table Formats)*
- [Lakehouse Fundamentals](<../../../02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Lakehouse_Fundamentals.md>) *(Lakehouse)*

## References

- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Installing Delta Lake › Python
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Installing Delta Lake › Databricks Community Edition
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Diving into the Delta Lake Ecosystem › Data types
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Diving into the Delta Lake Ecosystem › CREATE TABLE options
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Mastering Relational Entities in Databricks › Setting Up Delta Tables
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Mastering Relational Entities in Databricks › Table Constraints
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Mastering Relational Entities in Databricks › Cloning Delta Lake Tables
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Transforming Data with Apache Spark › Impact of not having a Delta table
- *Ultimate Data Engineering with Databricks* — Mayank Malhotra — Chapter 2: Mastering Delta Tables in Databricks › Mastering Delta Tables in Databricks
- *Ultimate Data Engineering with Databricks* — Mayank Malhotra — Chapter 2: Mastering Delta Tables in Databricks › Structure
- *Ultimate Data Engineering with Databricks* — Mayank Malhotra — Chapter 2: Mastering Delta Tables in Databricks › Understanding the Benefits of Delta Tables in Databricks
- *Ultimate Data Engineering with Databricks* — Mayank Malhotra — Chapter 2: Mastering Delta Tables in Databricks › Enhanced Data Reliability and Consistency with Delta Tables
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — What are Columnstore Indexes RE COLUMNSTORE INDEXES › The Delta Store
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Index Maintenance
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Examples
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Example
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Storage Design Patterns › Examples
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Lineage › Solution
- *Azure Data Engineering Cookbook* — Ahmad Osama — Batch and Streaming Data Processing with Azure Databricks › How to do it...

