---
id: "TOOLS/Google_Cloud/BigQuery/BigQuery_Loading_and_Table_Design"
level: 4
type: "concept"
topic: "TOOLS/Google_Cloud/BigQuery"
tools: ["Google_Cloud"]
---

# BigQuery Loading and Table Design

**L4 Advanced** · [↑ BigQuery](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** BigQuery is a serverless columnar warehouse where you load data in batch (free) or stream it (paid), then design tables around partitioning, clustering and nested/repeated columns so that queries read little data and updates are done with MERGE rather than row-by-row.

### 1. The big picture
BigQuery is not a row-store OLTP database like MySQL. Data is stored per column in Google's Capacitor format on Colossus; compute (Dremel slots) is separate and elastic; there are no user-managed indexes. Design therefore revolves around *how data is physically arranged* (partition, cluster), *how it is shaped* (denormalised with `STRUCT`/`ARRAY` instead of many joins) and *how it arrives* (batch load jobs, streaming, external tables, federated queries, transfer service).

### 2. Core concepts and vocabulary
- **Dataset** — a container (with a location) for tables and views; unit of access control.
- **Load job** — asynchronous batch ingestion from Cloud Storage/local files (CSV, JSON, Avro, Parquet, ORC); no per-load charge.
- **Storage Write API / streaming** — low-latency row ingestion, charged by volume; supports exactly-once with committed streams.
- **External table** — data stays in Cloud Storage/Drive/Bigtable; queried in place.
- **Partitioned table** — split by ingestion time, a `DATE/TIMESTAMP/DATETIME` column or integer range.
- **Clustered table** — data sorted by up to four columns within partitions.
- **`STRUCT` / `ARRAY` (`REPEATED`)** — nested and repeated fields for denormalised one-to-many data.
- **Materialized view** — precomputed, incrementally refreshed query result that the optimizer can use automatically.
- **`MERGE`** — upsert/delete statement combining INSERT, UPDATE and DELETE in one DML.
- **Schema modes** — `NULLABLE`, `REQUIRED`, `REPEATED`.
- **Audit logs** — Cloud Audit Logs data-access records of who queried what.

### 3. How it works, step by step
**Loading**
1. Land files in Cloud Storage (prefer Avro/Parquet/ORC: self-describing, typed, splittable and compressed; CSV/JSON need explicit schema/autodetect).
2. Run a load job (`bq load`, `LOAD DATA`, console, transfer service) with `WRITE_APPEND`, `WRITE_TRUNCATE` or `WRITE_EMPTY`.
3. The job reads in parallel, validates against the schema, and commits atomically — all rows or none (unless `max_bad_records` is set).
4. For continuous data, use the Storage Write API (or Pub/Sub → BigQuery subscription / Dataflow); rows are visible within seconds, but streaming buffer rows can't be modified by DML for a short time.

**Query**
1. Partitions that don't match filter predicates are pruned; cluster blocks pruned by min/max metadata.
2. Referenced columns are read from Capacitor files; nested columns read only the needed leaf fields.
3. Stages execute across slots with shuffles between stages (joins, aggregations).

### 4. Worked example
```sql
-- Denormalised orders table with nested line items, partitioned + clustered
CREATE TABLE `proj.shop.orders`
(
  order_id   STRING NOT NULL,
  customer   STRUCT<id STRING, country STRING>,
  order_ts   TIMESTAMP NOT NULL,
  items      ARRAY<STRUCT<sku STRING, qty INT64, price NUMERIC>>
)
PARTITION BY DATE(order_ts)
CLUSTER BY customer.country, order_id
OPTIONS (partition_expiration_days = 730);
```
```sql
-- Query the nested items without a join
SELECT order_id, SUM(i.qty * i.price) AS total
FROM `proj.shop.orders`, UNNEST(items) AS i
WHERE DATE(order_ts) = '2026-10-01'
GROUP BY order_id;
```
```sql
-- Idempotent upsert from a staging table
MERGE `proj.shop.customers` T
USING `proj.shop.customers_stage` S
ON T.id = S.id
WHEN MATCHED AND S.op = 'D' THEN DELETE
WHEN MATCHED THEN UPDATE SET name = S.name, email = S.email, updated_at = S.updated_at
WHEN NOT MATCHED AND S.op != 'D' THEN INSERT (id, name, email, updated_at)
     VALUES (S.id, S.name, S.email, S.updated_at);
```
```sql
-- Row→column pivot
SELECT * FROM (SELECT region, quarter, revenue FROM `proj.shop.sales`)
PIVOT (SUM(revenue) FOR quarter IN ('Q1','Q2','Q3','Q4'));
```
```sql
-- How much did the last query scan? (after the fact)
SELECT job_id, total_bytes_processed, total_bytes_billed, total_slot_ms
FROM `region-us`.INFORMATION_SCHEMA.JOBS_BY_USER
ORDER BY creation_time DESC LIMIT 5;
```

### 5. Under the hood
- **Capacitor + Dremel.** Columnar, run-length and dictionary encodings, reorganised in the background for better pruning; a tree of execution stages exchanges data through a distributed in-memory shuffle service.
- **Why nested/repeated fields.** A one-to-many relationship stored as an array of structs keeps child rows physically next to the parent, so no join/shuffle is needed to reassemble them; `UNNEST` flattens at query time. It's BigQuery's analogue of pre-joining, with storage still columnar per leaf.
- **DML.** Each DML statement rewrites affected partitions/blocks (copy-on-write); frequent single-row updates are slow and quota-limited (DML concurrency queues), so batch changes with `MERGE` from a staging table. Partition-level operations (`TRUNCATE`, partition decorators, `WRITE_TRUNCATE` to a partition) are cheap.
- **Partition limits.** 10,000 partitions per table; granularity can be hourly, daily, monthly or yearly. Clustering benefits large tables (roughly > 1 GB); it's maintained automatically via background re-clustering.
- **Materialized view rewrite.** The optimizer can use a materialized view for queries on the base table when the query is *semantically a subset* of the view's supported shapes (a restricted set of aggregates, no non-deterministic functions, limited joins). Surprises: unsupported features (analytic functions, certain joins, UDFs) stop it; staleness is bounded by `max_staleness`; base table partition/cluster alignment affects refresh cost.
- **Schema evolution.** Adding nullable columns and relaxing `REQUIRED → NULLABLE` are metadata-only. Dropping or renaming columns is supported via DDL (`ALTER TABLE … DROP COLUMN / RENAME COLUMN`) but dropped data may remain in time travel; changing a column's type is limited (a few widening coercions) — otherwise create a new table with `CREATE TABLE AS SELECT` and swap.
- **Audit.** Data-access logs (BigQuery `AuditData`/`BigQueryAuditMetadata`) record `tableDataRead`, principal, query text; export logs to a BigQuery sink to answer "who read the PII table in 30 days" with SQL, and alert on unusual volume/new principals.

### 6. Performance and design implications
- **Prefer batch loads** to streaming when minutes of latency are fine; they're free and atomic.
- **Use Avro/Parquet** for schema safety and speed; partition files in GCS by date for parallel loads and backfills.
- **Pick the partition column by the dominant filter**, cluster by next-most common filter/join columns, and use `require_partition_filter`.
- **Denormalise within reason**: nest one-to-many that is always read together; keep dimensions that change independently separate and join them (broadcast joins on small dimensions are fast).
- **Avoid tiny frequent DML**; stage and MERGE; consider append-only + latest-view (`ROW_NUMBER() OVER (PARTITION BY id ORDER BY ts DESC)`) for CDC.
- **Cost-aware access**: authorised views, row-level security and policy tags (column-level security) for PII; labelled tables.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `bq load` / `LOAD DATA` | Batch load | Files in GCS |
| `PARTITION BY`, `CLUSTER BY` | Physical layout | Any big table |
| `MERGE` | Upsert/delete | CDC and dimension loads |
| `ALTER TABLE ADD/DROP/RENAME COLUMN`, `ALTER COLUMN SET DATA TYPE` | Schema change | Evolution |
| `CREATE MATERIALIZED VIEW … OPTIONS(max_staleness…)` | Pre-aggregation | Repeated aggregates |
| `PIVOT`, `UNPIVOT`, `UNNEST` | Reshape | Reporting |
| `INFORMATION_SCHEMA.JOBS`, `TABLE_STORAGE` | Metadata | Cost, usage |
| Policy tags, row access policies | Fine-grained security | PII |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Load jobs | You need sub-minute freshness |
| Storage Write API | You want exactly-once, high-throughput streaming |
| External tables | Data must stay in lake and queries are infrequent — slower, less pruning |
| Nested/repeated | You need to update individual child rows often |
| Partition by ingestion time | Event time differs widely from arrival (late data) — prefer event-time column |
| Clustering | Table is small or filters don't use cluster columns |

### 9. Common mistakes
- Treating BigQuery like MySQL (row updates, normalised star with 10 joins) → slow and costly → denormalise and batch DML.
- Partitioning by a high-cardinality or non-filtered column → no pruning, partition limit hit → partition on date filters.
- CSV loads with autodetect in production → type drift/failed loads → explicit schema or Avro/Parquet.
- Streaming everything → unnecessary cost → batch where possible.
- Not setting `require_partition_filter` → full-table scans → enforce it.
- Dropping a column assuming data is gone → remains in time travel/fail-safe → handle PII deletion via policy.

### 10. 🎯 Interview kit
**30-second answer:** BigQuery stores data columnarly, so I load in batch (Avro/Parquet), partition by the date column queries filter on, cluster by common filters, denormalise one-to-many with nested/repeated fields, and use MERGE for upserts. Cost and speed come from reading fewer columns and partitions.

**Structure a longer answer:** architecture → ingestion modes → physical design → modelling → DML/updates → governance and cost.

**Likely follow-ups:**
- *Columnar storage benefit?* → read only needed columns, better compression.
- *BigQuery vs MySQL?* → OLAP, serverless, no indexes, append-oriented, DML limits.
- *When will a materialized view be used?* → eligible query shapes against the base table, within staleness window.
- *Safe schema evolution?* → add nullable, relax required; for type changes create a new table and backfill.

**Signals interviewers look for:** physical design vocabulary; awareness of DML limits; cost/governance mindset.

### 11. Check yourself
- [ ] Write DDL for a partitioned, clustered events table with a nested array.
- [ ] Write a MERGE that handles insert, update and delete from a CDC stage.
- [ ] Explain why batch load beats streaming for cost.
- [ ] Describe which schema changes are free vs need a rewrite.
- [ ] Write the query that finds who read a PII table in 30 days.
<!-- explainer:end -->

## Implementations (tools)

- [Google Cloud](<../_index.md>)

## Practice questions

- [Basic] [What is a columnar storage format and why does BigQuery use one?](<../../../PRACTICE/TOOLS/Google_Cloud/BigQuery/Q-05007_What_is_a_columnar_storage_format_and_why_does_BigQuery_use_.md>)
- [Basic] [How does BigQuery differ from a traditional relational database like MySQL?](<../../../PRACTICE/TOOLS/Google_Cloud/BigQuery/Q-05170_How_does_BigQuery_differ_from_a_traditional_relational_datab.md>)
- [Intermediate] [How can you query data stored in Cloud Storage without loading it into BigQuery?](<../../../PRACTICE/TOOLS/Google_Cloud/BigQuery/Q-05142_How_can_you_query_data_stored_in_Cloud_Storage_without_loadi.md>)
- [Intermediate] [How would you validate that a nightly load into BigQuery completed correctly?](<../../../PRACTICE/TOOLS/Google_Cloud/BigQuery/Q-05192_How_would_you_validate_that_a_nightly_load_into_BigQuery_com.md>)
- [Deep] [How would you design a partitioned table for daily sales data to keep costs low?](<../../../PRACTICE/TOOLS/Google_Cloud/BigQuery/Q-05164_How_would_you_design_a_partitioned_table_for_daily_sales_dat.md>)
- [Deep] [Explain when a BigQuery materialized view will and will not be used to accelerate a query automatically, and the limitations that surprise engineers.](<../../../PRACTICE/TOOLS/Google_Cloud/BigQuery/Q-05016_Explain_when_a_BigQuery_materialized_view_will_and_will_not_.md>)
- [Deep] [How do you evolve a BigQuery table schema safely when adding, relaxing, or removing a column, and which changes require a rewrite?](<../../../PRACTICE/TOOLS/Google_Cloud/BigQuery/Q-05048_How_do_you_evolve_a_BigQuery_table_schema_safely_when_adding.md>)
- [Deep] [Explain the difference between INFORMATION_SCHEMA and the older __TABLES__ and Data Catalog for metadata, and how you would build a lineage/discovery layer.](<../../../PRACTICE/TOOLS/Google_Cloud/BigQuery/Q-05077_Explain_the_difference_between_INFORMATION_SCHEMA_and_the_ol.md>)

## Related concepts

- [Data Warehouse Fundamentals](<../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Data_Warehouse_Fundamentals.md>) *(Data Warehousing)*
- [Partitioning and Partition Pruning](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Partitioning_and_Partition_Pruning.md>) *(DBMS Concepts)*
- [Large Values and TOAST](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Large_Values_and_TOAST.md>) *(SQL Fundamentals)*
- [Views and Materialized Views](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Views_and_Materialized_Views.md>) *(DBMS Concepts)*
- [Databases and Schemas](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Databases_and_Schemas.md>) *(DBMS Concepts)*
- [Choosing a Database for the Workload](<../../../01_Databases_and_SQL/Database_Strategy/Database_Selection/Choosing_a_Database_for_the_Workload.md>) *(Database Selection)*

## References

- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — CHAPTER 12Popular Technologies › Google BigQuery
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Compute Engines for Lakehouse Architectures › BigQuery
- *Automating Data Quality Monitoring* — Jeremy Stanley, Paige Schwartz — Integrating Monitoring with Data Tools and Systems
- *Building Data Integration Solutions* — Jay Borthen — Data Stores and Management Systems › Google Bigtable
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Storage Design Patterns › Examples

