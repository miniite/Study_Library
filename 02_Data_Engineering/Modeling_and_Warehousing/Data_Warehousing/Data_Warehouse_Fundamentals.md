---
id: "02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Data_Warehouse_Fundamentals"
level: 4
type: "concept"
topic: "02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing"
tools: ["AWS"]
---

# Data Warehouse Fundamentals

**L4 Advanced** · [↑ Data Warehousing](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A data warehouse is a separate, integrated, history-keeping store, modeled for analytical questions, that is loaded from operational systems so reporting doesn't hurt (or depend on) them.

### 1. The big picture
Operational systems (OLTP) run the business: place an order, update an address. They are optimized for many small reads and writes touching a few rows, with current-state data and strict consistency. Analysts ask different questions: *revenue by region by month for three years, compared with last year.* Those queries scan huge numbers of rows, group them, and join across subject areas. Running them on the operational database is slow and risks the transactions.

A **data warehouse (DW)** solves this by copying data from many sources into a separate database designed for analytics. Classic properties: **subject-oriented** (organized around business subjects such as sales or customers), **integrated** (consistent names, units, keys across sources), **time-variant** (history is kept), and **non-volatile** (data is added, not constantly changed in place). It is typically built with a schema defined first (schema-on-write) and queried with SQL.

### 2. Core concepts and vocabulary
- **OLTP vs OLAP** — transactional processing (many small writes, current state) vs analytical processing (large scans, aggregations, history).
- **Schema-on-write** — structure is enforced when data is loaded; the warehouse approach.
- **Schema-on-read** — structure applied at query time; the data lake approach.
- **ETL vs ELT** — transform before loading vs load raw then transform inside the warehouse engine.
- **Staging area** — landing zone for extracted data before transformation.
- **Fact table** — measurable events (sales lines, clicks) at a defined grain.
- **Dimension table** — descriptive context (customer, product, date).
- **Star schema** — a central fact table joined to denormalized dimensions.
- **Data mart** — a subject-focused subset of the warehouse for a team or process.
- **Incremental load** — load only new/changed records since the last run.
- **Columnar storage / MPP** — storing by column and spreading work across nodes to scan quickly.
- **Metadata/catalog** — documentation of tables, lineage, owners, refresh times.

### 3. How it works, step by step
1. **Extract** data from operational databases, SaaS APIs, files, or event streams — often via change data capture or timestamp/ID watermarks.
2. **Land** it in a staging area unchanged, so loads are repeatable and auditable.
3. **Cleanse and conform** — standardize types and codes, deduplicate, resolve keys across systems, apply business rules.
4. **Model** — load dimensions (assigning surrogate keys, handling history) then facts (looking up dimension keys).
5. **Publish** to marts/semantic layers and BI tools.
6. **Operate** — schedule, monitor freshness and quality, manage retention, control access, and document.

Why *batch* (often nightly)? It uses off-peak hours, avoids competing with transactional load, allows heavy transformations with consistent snapshots, and is cheaper than continuous updates. Business needs increasingly push toward micro-batch or streaming loads, but the design principle remains: choose freshness by what the decision needs.

**Incremental vs full reload:** a full reload re-reads and rewrites everything; an incremental load processes only changes, reducing source load, compute, and runtime — at the cost of needing a reliable change signal (CDC, `updated_at`, a monotonically increasing key) and a way to handle deletes.

### 4. Worked example
```sql
-- Staging → dimension (incremental, SCD1 for brevity) → fact
MERGE INTO dw.dim_customer AS d
USING stg.customer_changes AS s
   ON d.customer_nk = s.customer_id
WHEN MATCHED THEN UPDATE SET name = s.name, country = s.country, updated_at = s.updated_at
WHEN NOT MATCHED THEN INSERT (customer_nk, name, country, updated_at)
                      VALUES (s.customer_id, s.name, s.country, s.updated_at);

INSERT INTO dw.fact_sales (date_key, customer_key, product_key, quantity, net_amount)
SELECT dd.date_key, dc.customer_key, dp.product_key, o.quantity, o.net_amount
FROM stg.order_lines o
JOIN dw.dim_date     dd ON dd.full_date   = o.order_date
JOIN dw.dim_customer dc ON dc.customer_nk = o.customer_id
JOIN dw.dim_product  dp ON dp.product_nk  = o.product_id
WHERE o.load_batch_id = :batch;                       -- idempotent per batch

-- A typical analytical question
SELECT dd.year, dd.month, dc.country, SUM(f.net_amount) AS revenue
FROM dw.fact_sales f
JOIN dw.dim_date dd ON dd.date_key = f.date_key
JOIN dw.dim_customer dc ON dc.customer_key = f.customer_key
GROUP BY dd.year, dd.month, dc.country;
```
The star shape makes the analytic query a simple fact-to-dimension join-and-group; dimension loads precede facts so every key resolves.

### 5. Under the hood
- **Columnar storage** keeps each column contiguous, so a query touching 4 of 80 columns reads only those, and similar values compress well (run-length, dictionary encoding).
- **MPP (massively parallel processing)** engines (Redshift, Synapse dedicated pools, Snowflake, BigQuery) split data across nodes; each node scans its slice and results are combined. Data distribution and sort/cluster choices determine how much movement joins require.
- **Separation of storage and compute** in modern cloud warehouses lets compute scale and pause independently of data volume.
- **Warehouse vs lake:** warehouses hold structured, curated, typically recent-years data with low-latency SQL; lakes hold raw and diverse data cheaply with schema applied on read and are used by Spark/Flink/Python. Most organizations use both, and lakehouse formats (Delta, Iceberg) blur the line by adding table semantics to lake storage.
- **Choosing an analytical store** depends on data structure (structured → warehouse; evolving/unstructured → lake), query patterns (known and repeatable → warehouse; exploratory → lake), performance needs, and integration/skills.

### 6. Performance and design implications
- Model at the **smallest grain you need, no lower** — daily vs hourly granularity multiplies rows needlessly.
- Pre-aggregate only for proven hot paths (summary tables/materialized views).
- Partition/cluster by common filter columns (usually date).
- Keep history in the warehouse for analytics; archive cold data to cheaper storage.
- Measure: load duration and freshness, query latency percentiles, scan bytes, concurrency queue time, cost per query.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `MERGE` | Upsert from staging | Incremental dimension/fact loads |
| `COPY` / bulk load | Fast ingestion from object storage | Loading staged files |
| Partitioning / clustering / sort keys | Reduce scanned data | Large fact tables |
| Materialized views / summary tables | Precompute aggregates | Repeated dashboard queries |
| Result cache / workload management | Concurrency and cost control | Shared BI workloads |
| Time travel / snapshots | Recover or audit | Bad loads |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Warehouse — structured, governed analytics | Large volumes of raw/unstructured data needing cheap storage |
| Lake/lakehouse — varied data, ML | Business users need fast, curated SQL without engineering help |
| ETL — transform before load, constrained target | Target engine is powerful; ELT is simpler |
| ELT — cloud MPP warehouse | Sensitive data must be masked before landing |
| Batch — stable reporting | Decisions require seconds-level freshness |

### 9. Common mistakes
- Reporting directly off the OLTP system → contention and unstable numbers → separate analytical store.
- Loading facts before dimensions → unresolved keys → enforce load order and handle unknowns.
- Overwriting history in place → can't answer "as it was" → design history handling (SCDs).
- Full reload of huge tables nightly → long windows, high cost → incremental with reliable change detection.
- No catalog or ownership → users don't trust or find data → maintain metadata and lineage.

### 10. 🎯 Interview kit
**30-second answer:** A data warehouse is an integrated, historical, analytics-optimized store separate from operational systems, loaded via ETL/ELT and usually modeled as star schemas. It's built for large scans and aggregations (OLAP), whereas operational databases are for small, frequent transactions (OLTP).

**Structure a longer answer:** clarify (users, freshness, volume) → core mechanism (staging → conform → dimensions → facts → marts) → trade-offs (warehouse vs lake, ETL vs ELT, batch vs streaming) → scenario (move reporting off the production database).

**Likely follow-ups:**
- *OLTP vs OLAP?* → access pattern, schema (normalized vs dimensional), storage (row vs column).
- *Why batch nightly?* → off-peak, consistent snapshots, cost, no impact on OLTP.
- *How does incremental load save cost?* → processes only deltas; needs change detection.
- *Warehouse vs lake?* → schema-on-write vs schema-on-read; curated vs raw.

**Signals interviewers look for:** crisp OLTP/OLAP distinction, load-pattern reasoning, awareness of modern cloud architecture.

### 11. Check yourself
- [ ] State the four classic warehouse properties.
- [ ] Draw a pipeline from source to mart.
- [ ] Compare a full reload with an incremental load and name the prerequisites.
- [ ] Decide warehouse vs lake for three scenarios.
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Basic] [What is an enterprise data warehouse (EDW) and how is it different from a departmental data mart?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-19151_What_is_an_enterprise_data_warehouse_EDW_and_how_is_it_diffe.md>)
- [Basic] [How would you explain the flow of data from a source system into dashboards to a non-technical stakeholder?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-19061_How_would_you_explain_the_flow_of_data_from_a_source_system_.md>)
- [Intermediate] [How does an incremental load reduce cost and time compared to reloading everything?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-19019_How_does_an_incremental_load_reduce_cost_and_time_compared_t.md>)
- [Intermediate] [What is backfilling in a warehouse and when would you need to run one?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-19057_What_is_backfilling_in_a_warehouse_and_when_would_you_need_t.md>)
- [Deep] [What is an accumulating snapshot fact table and give a real example such as order fulfillment.](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-19015_What_is_an_accumulating_snapshot_fact_table_and_give_a_real_.md>)
- [Deep] [You need to backfill 3 years of a redesigned fact table without impacting the nightly SLA. How do you sequence and throttle it?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-19084_You_need_to_backfill_3_years_of_a_redesigned_fact_table_with.md>)
- [Deep] [A fact load duplicated one day's data after an upstream Kafka replay. How do you detect and cleanly remove the dupes without a full reload?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-19108_A_fact_load_duplicated_one_day_s_data_after_an_upstream_Kafk.md>)
- [Deep] [What's your approach to handling schema drift from a semi-structured JSON source landing into a warehouse VARIANT/STRUCT column?](<../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Q-19135_What_s_your_approach_to_handling_schema_drift_from_a_semi-st.md>)

## Related concepts

- [Handling Source Changes and Spikes](<../../Pipelines_and_Integration/ETL_ELT/Handling_Source_Changes_and_Spikes.md>) *(ETL ELT)*
- [Lakehouse Fundamentals](<../../Storage_and_Lakehouse/Lakehouse/Lakehouse_Fundamentals.md>) *(Lakehouse)*
- [End to End No Data Loss](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/End_to_End_No_Data_Loss.md>) *(Data Architecture)*
- [Warehouse as System of Record](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Warehouse_as_System_of_Record.md>) *(Data Architecture)*
- [Databases and Schemas](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Databases_and_Schemas.md>) *(DBMS Concepts)*
- [File and Table Formats for Features](<../../Pipelines_and_Integration/Pipeline_Orchestration/File_and_Table_Formats_for_Features.md>) *(Pipeline Orchestration)*

## References

- *Azure Data Fundamentals* — Michael John Pena — Introduction to Data Representation › Exam Tip
- *Azure Data Fundamentals* — Michael John Pena — Data Storage Fundamentals › Analytical architecture
- *Azure Data Fundamentals* — Michael John Pena — Data Storage Fundamentals › Data warehouse use cases
- *Azure Data Fundamentals* — Michael John Pena — Understanding Data Workloads › Chapter 3. Understanding Data Workloads
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Prerequisite Knowledge for Aspiring Data Engineers › What Is a Data Warehouse?
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Prerequisite Knowledge for Aspiring Data Engineers › Data Warehouse Versus Data Lake
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Overview of AWS Analytics and Auxiliary Services › Database
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Chapter 5. Data Store Management
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Introduction to Data Engineering › 1. Introduction to Data Engineering
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Architectural Patterns in Data Engineering › 3. Architectural Patterns in Data Engineering
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Storage Design Patterns in Data Engineering › 5. Storage Design Patterns in Data Engineering
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Future of Data Engineering › 15. Future of Data Engineering
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Data Engineering's Role › Different types of data roles
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Data Engineering's Role › Inventory control
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — Data Models › Reference architecture, data marts and data lakes
- *Data Engineering Best Practices* — Luiz Fernando F. Dos Santos — DataOps and AI › Storage and data organization
- *The Enterprise Big Data Lake* — Alex Gorelik — Introduction to Data Lakes › Data Puddles
- *The Enterprise Big Data Lake* — Alex Gorelik — Introduction to Data Lakes › Data Ponds
- *The Enterprise Big Data Lake* — Alex Gorelik — Introduction to Data Lakes › Preparing the data
- *The Enterprise Big Data Lake* — Alex Gorelik — Historical Perspective › The Data Warehouse Ecosystem
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Technical interview questions › Data Warehouses and Data Lakes
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Exploring data warehouse essentials for data engineers
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Technical interview questions
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Technical interview questions › Summary

