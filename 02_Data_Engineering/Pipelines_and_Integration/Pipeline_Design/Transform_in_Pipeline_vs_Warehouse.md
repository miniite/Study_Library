---
id: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Transform_in_Pipeline_vs_Warehouse"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design"
tools: ["AWS", "Apache_Airflow"]
---

# Transform in Pipeline vs Warehouse

**L4 Advanced** · [↑ Pipeline Design](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** You can transform data in the pipeline's own compute (ETL: Spark, Python, Lambda) before it lands, or load it raw and transform with SQL inside the warehouse (ELT / push-down); the right choice depends on data size, governance, who owns the logic, and where compute is cheapest.

### 1. The big picture
Every pipeline has to turn raw source data into something analysts can use. The question is *where* that work happens. In the classic ETL model, a separate engine extracts data, transforms it, and loads the finished result. In ELT, you load raw data first and let the warehouse or lakehouse engine do the transforming with SQL — often called **push-down**, because the work is pushed to where the data already lives.

The shift toward ELT came from cloud warehouses: elastic, columnar, massively parallel engines made it cheap to run large SQL transformations, and keeping raw data means you can re-derive anything later. But ELT is not universally better. Some transformations (parsing binaries, calling ML models, masking PII before it lands, heavy per-record logic) are awkward or impossible in SQL, and sometimes you must not store raw data at all.

### 2. Core concepts and vocabulary
- **ETL** — Extract, Transform, Load: transform in an intermediate engine before loading.
- **ELT** — Extract, Load, Transform: land raw data, transform inside the target.
- **Push-down** — executing transformation logic in the database/warehouse instead of pulling data out to process it.
- **Raw / bronze layer** — unmodified landed data, kept for replay.
- **Transformation layer (silver/gold, marts)** — cleaned and modeled outputs built by SQL models.
- **Compute-storage separation** — warehouses/lakehouses scale compute independently of storage, making in-warehouse transforms elastic.
- **Declarative transform** — you describe the result (SQL, dbt models); the engine plans execution.
- **Lineage** — the dependency graph between datasets; easier to derive when transforms are SQL models.

### 3. How it works, step by step
**ETL path**
1. Extract from the source into the pipeline engine.
2. Transform there (clean, join, mask, compute features).
3. Load only the finished output into the warehouse.

**ELT path**
1. Extract and load raw tables to the warehouse (or lake) with minimal change.
2. Run SQL models (views, incremental tables) that build cleaned layers.
3. Orchestrate model order via the dependency graph; test and document in the same tool.

**Deciding where each step should live**
1. List the transformations.
2. For each, ask: is it set-based and relational (joins, aggregations, windowing)? → warehouse SQL. Does it need arbitrary code, external calls, or very custom parsing? → pipeline engine.
3. Ask whether data may land raw (privacy, residency). If not, minimal transform/masking must happen before load.
4. Check volume and cost: moving big data out of the warehouse to transform and back costs time and egress.
5. Consider ownership: analytics engineers write SQL; data engineers maintain Spark code. Put logic where its owners work.

### 4. Worked example
Orders arrive as JSON from an API. 

ELT version:
```sql
-- raw landing (loaded as-is, one VARIANT/JSON column + load metadata)
-- raw.orders_json(payload, _loaded_at, _file)

-- staging model: parse and type
CREATE OR REPLACE VIEW stg.orders AS
SELECT payload:id::bigint            AS order_id,
       payload:customer_id::bigint   AS customer_id,
       payload:amount::decimal(12,2) AS amount,
       payload:created_at::timestamp AS created_at_utc
FROM raw.orders_json;

-- mart model: aggregate
CREATE OR REPLACE TABLE mart.daily_revenue AS
SELECT DATE(created_at_utc) AS day, SUM(amount) AS revenue
FROM stg.orders
GROUP BY 1;
```
ETL version: a Spark job reads the JSON, drops sensitive fields, validates, writes Parquet or loads `mart.daily_revenue` directly.

Reading the comparison: the ELT version keeps raw JSON, so if the definition of "revenue" changes you re-run SQL with no re-ingest. The ETL version minimizes what is stored and may be required if the payload contains card numbers that must never land.

### 5. Under the hood   *(optional at L4)*
- **Why push-down is fast:** the warehouse already has the data in columnar compressed format next to its compute; the optimizer prunes partitions/columns, parallelizes joins, and avoids shipping rows over the network.
- **Why it can be slow or costly:** every re-run of a heavy model consumes warehouse credits; poorly written SQL can scan entire tables repeatedly. Incremental models and partition filters are the mitigation.
- **Hybrid is the norm:** use the pipeline engine for ingestion, parsing, ML scoring, and privacy steps; use warehouse SQL for modeling and aggregation. Many "ETL vs ELT" debates dissolve into "which step goes where".
- **Streaming complicates it:** event-time semantics, windows and late data are usually handled in the stream processor before landing, with the warehouse serving analytical aggregation afterward.

### 6. Performance and design implications
- Measure where time is spent: extraction, network transfer, transformation, load.
- ETL cost = pipeline cluster + transfer; ELT cost = warehouse compute per run. Compare on your volumes.
- Keep transformations deterministic and idempotent in both models.
- ELT makes lineage, testing and documentation easier because logic is SQL in version control; ETL code needs equivalent discipline.
- Governance: apply masking policies and access controls on raw layers if you land raw data.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| dbt models / SQL views | Declarative in-warehouse transforms | ELT modeling layers |
| Incremental model / `MERGE` | Process only new data | Large fact tables |
| Spark / Python job | Custom or heavy pre-load processing | ETL, parsing, ML |
| Masking policies, column-level security | Protect raw PII | When landing raw data |
| Warehouse sizing / auto-suspend | Control compute spend | Heavy ELT workloads |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| ELT — powerful elastic warehouse, SQL-friendly logic, want raw history | Raw data cannot be stored; logic needs non-SQL code |
| ETL — must filter/mask before landing, custom processing, expensive warehouse compute | Pipeline cluster becomes a second system to maintain for simple SQL work |
| Hybrid — ingestion/parsing outside, modeling inside | Logic is scattered with no clear ownership |
| Push everything down — small team, SQL skills | Queries scan huge raw tables repeatedly with no incremental design |

### 9. Common mistakes
- Pulling large tables out of the warehouse to transform in Python → slow, costly → push down set-based work.
- Landing PII raw without controls → compliance exposure → mask early or secure the raw layer.
- Rebuilding full tables every run → warehouse bill spikes → incremental models.
- Mixing business logic in ingestion code and SQL → duplicate definitions → one owner per metric.
- Forgetting UTC and type normalization → inconsistent results → standardize in the first transform layer.

### 10. 🎯 Interview kit
**30-second answer:** ETL transforms before loading in a separate engine; ELT loads raw and transforms in the warehouse using SQL, which is push-down. With cheap elastic warehouses ELT is the default for modeling because it keeps raw history and is easy to test and trace, but I use ETL-style steps for parsing, masking, or logic SQL cannot do.

**Structure a longer answer:** clarify (volume, latency, privacy, skills) → compare both → decide per transformation → cost and governance → close with a hybrid example.

**Likely follow-ups:**
- *Why did ELT become popular?* → cloud warehouses decoupled compute and storage; SQL tooling like dbt.
- *When is ETL still right?* → privacy before landing, heavy custom code, constrained warehouse.
- *How do you control ELT cost?* → incremental models, pruning, warehouse sizing.
- *Where does streaming fit?* → process event-time logic in the stream, land results, model with SQL.

**Signals interviewers look for:** you treat it as a per-step decision, mention governance and cost, and avoid ideology.

### 11. Check yourself
- [ ] Define push-down and give an example.
- [ ] Name two transformations that belong outside the warehouse and why.
- [ ] Explain how ELT preserves the ability to change a metric definition later.
- [ ] Describe how you would control warehouse cost for a heavy model.
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)
- [Apache Airflow](<../../../TOOLS/Apache_Airflow/_index.md>)

## Practice questions

- [Basic] [How do you decide the schedule (hourly, daily, weekly) for a batch pipeline?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12853_How_do_you_decide_the_schedule_hourly_daily_weekly_for_a_bat.md>)
- [Basic] [What is the difference between orchestration and the actual data processing in a pipeline?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12900_What_is_the_difference_between_orchestration_and_the_actual_.md>)
- [Intermediate] [What is the difference between a full load and an incremental load in a pipeline?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12854_What_is_the_difference_between_a_full_load_and_an_incrementa.md>)
- [Intermediate] [How do you decide whether a transformation should happen row-by-row or in a set-based way?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12882_How_do_you_decide_whether_a_transformation_should_happen_row.md>)
- [Intermediate] [What is an event-driven pipeline and how does it differ from a batch pipeline?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12904_What_is_an_event-driven_pipeline_and_how_does_it_differ_from.md>)
- [Intermediate] [How would you handle a pipeline that suddenly starts taking twice as long to run?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12913_How_would_you_handle_a_pipeline_that_suddenly_starts_taking_.md>)
- [Intermediate] [What is the difference between extracting data by query versus by exporting a full table dump?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12932_What_is_the_difference_between_extracting_data_by_query_vers.md>)
- [Intermediate] [How do you decide when to introduce a message queue between two pipeline stages instead of a direct call?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12994_How_do_you_decide_when_to_introduce_a_message_queue_between_.md>)

## Related concepts

- [Pipeline Design Patterns](<Pipeline_Design_Patterns.md>) *(Pipeline Design)*
- [Batch vs Streaming Cost Trade-offs](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_vs_Streaming_Cost_Trade-offs.md>) *(Batch vs Stream)*
- [Pipeline Data Monitoring](<../Pipeline_Orchestration/Pipeline_Data_Monitoring.md>) *(Pipeline Orchestration)*

## References

- *Building Data Integration Solutions* — Jay Borthen — Key Concepts in Data Integration › Pipelines
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › DataOps
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Future of Data Engineering › Streaming Pipelines and Real-Time Analytical Databases
- *Data Engineering Design Patterns* — Bartosz Konieczny — Pattern: Skew Detector › Solution
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Be Intentional About the Batching Model in Your Data Pipelines › ATW and DTW Batching in the Same Pipeline
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Time (Semantics) Won't Wait › Chapter 83. Time (Semantics) Won't Wait
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Prerequisite Knowledge for Aspiring Data Engineers › Different Ways to Process Data
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Implementing Batch and Streaming Pipelines › Implementing a Batch Processing Pipeline
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Collecting, Cleaning, Transforming, and Testing Data › Batch Versus Stream Processing

