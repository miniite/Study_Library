---
id: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Pipeline_Design_Patterns"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design"
tools: ["AWS", "Apache_Kafka"]
---

# Pipeline Design Patterns

**L4 Advanced** · [↑ Pipeline Design](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A data pipeline is a chain of source → ingest → stage → transform → serve steps, and good design is a small set of repeatable patterns — idempotent tasks, staging, incremental loads, parameterization, monitoring — applied consistently.

### 1. The big picture
A data pipeline is the set of processes that moves and transforms data from sources to a destination where it creates value (reporting, analytics, ML). Simple ones copy one API into one table; real ones chain extraction, validation, transformation and loading across several systems and languages, and a team usually owns dozens that share dependencies.

Design patterns are the answer to the questions every pipeline raises: *when does it run, how does it avoid duplicates, how does it recover, how do we know it is healthy, how do we change it safely?* Learning the patterns lets you reason about any tool — Airflow, dbt, Spark, a cron script — in the same vocabulary. Reliability (the data engineering books stress this) is the cornerstone: when a pipeline stalls, every downstream consumer stalls with it, and silently stale data is worse than a loud failure.

### 2. Core concepts and vocabulary
- **Source / sink** — where data comes from / where it lands (database, API, file drop, topic; warehouse, lake, queue).
- **Stage (staging area)** — intermediate storage between extract and final load; lets you validate and re-run without touching the target.
- **ETL vs ELT** — transform before loading vs load raw then transform inside the warehouse.
- **Batch vs streaming** — process bounded groups on a schedule vs continuously per event.
- **Full vs incremental load** — recopy everything vs only what changed since a cursor.
- **Idempotency** — re-running yields the same result; the foundation of safe retries.
- **Parameterization** — dates, paths and credentials passed in rather than hard-coded, so one pipeline serves many runs/environments.
- **Orchestration** — scheduling and dependency management (DAGs, retries, alerts).
- **Medallion layers (bronze/silver/gold)** — raw → cleaned → business-ready zones.
- **Observability** — metrics, logs, lineage and data-quality checks that show health.

### 3. How it works, step by step
A typical well-designed pipeline looks like this:

1. **Trigger** — time-based (schedule), event-based (file arrived, message published) or manual (backfill).
2. **Extract** — pull or receive from the source with a cursor (timestamp, ID, CDC position). Prefer reading from replicas/exports to avoid hurting the source.
3. **Land raw** — store the unmodified payload (immutable, partitioned by ingest date). This is your replay source.
4. **Validate** — schema, null/required fields, row counts, freshness; decide to halt, warn, or quarantine bad rows.
5. **Transform** — clean, conform types and time zones (store UTC), deduplicate, enrich with reference tables, aggregate.
6. **Load** — write atomically and idempotently (partition overwrite, merge by key).
7. **Publish & observe** — update a freshness marker, emit metrics, notify consumers, record lineage.
8. **Recover** — retries for transient errors, checkpoints for resume, backfill for history.

Core patterns that appear across those steps:

| Pattern | Problem it solves | Idea |
|---|---|---|
| Idempotent task | Retries create duplicates | Overwrite/merge keyed by run or business key |
| Staging + atomic swap | Readers see half-loaded data | Load aside, then promote in one operation |
| Incremental / high-water mark | Full reloads are slow and costly | Process only rows after the last cursor |
| Parameterized run (logical date) | Cannot backfill or reuse | Task takes date/partition as input |
| Dead-letter / quarantine | One bad record fails everything | Route bad rows aside with reason |
| Fan-out / fan-in | Independent units processed serially | Parallel per partition/file, then combine |
| Concurrent runner with limits | Backfills starve other jobs | Cap parallel runs |
| Fail loudly, alert | Silent staleness | Explicit failure states and notifications |

### 4. Worked example
Daily CSV files land in a folder; load them into a warehouse table.

```python
def run(logical_date):                      # parameterized
    path = f"landing/sales/{logical_date}/*.csv"
    df = read_csv(path)                     # extract
    assert len(df) > 0, "no rows"           # row-count check: fail loudly
    df = (df.dropna(subset=["order_id"])    # validate / clean
            .assign(ts=lambda d: pd.to_datetime(d.ts, utc=True))
            .drop_duplicates("order_id"))   # dedupe on business key
    write_parquet(df, f"stage/sales/{logical_date}/")      # stage
    sql("""
      MERGE INTO dw.sales t
      USING stage.sales_{d} s ON t.order_id = s.order_id
      WHEN MATCHED THEN UPDATE SET ...
      WHEN NOT MATCHED THEN INSERT ...""")  # idempotent load
    emit_metric("sales_rows", len(df), logical_date)       # observe
```

Re-running for the same date yields the same table state; backfilling a month is a loop over dates. The same function works in dev, staging and prod by changing configuration only.

### 5. Under the hood   *(optional at L4)*
- **Time matters:** track event time, ingestion time and processing time separately; many "wrong totals" bugs are time-zone or late-arrival bugs.
- **ETL vs ELT decision:** modern cloud warehouses make compute cheap and elastic, so loading raw first and transforming with SQL (dbt) keeps raw history and simplifies logic; ETL still wins when data must be masked or reduced before landing.
- **Orchestrator vs engine:** orchestrators decide *when* and *in what order*; engines (Spark, SQL warehouse) do the work. Keep heavy compute out of the scheduler.
- **Workflow granularity:** one giant task is simple but has a large failure domain; many tiny tasks isolate failures but add scheduling overhead. Choose steps that are independently re-runnable.

### 6. Performance and design implications
- Parallelism over single-node speed: design units (partitions, files) that can run in parallel.
- Right-size batches: small batches mean more overhead; huge ones mean long locks and memory pressure.
- Cost: pay for work, not idle; use incremental loads and partition pruning.
- Monitor uptime, latency, volumes, freshness and quality; "no alert" must not mean "nothing ran".

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| Cron / schedule interval | Time-based trigger | Regular batch |
| Sensors / event triggers | Start on arrival | Variable arrival times |
| Retries + backoff | Auto re-attempt | Transient errors |
| `MERGE`, partition overwrite | Idempotent write | Every load |
| Row-count / freshness checks | Basic quality gates | Every pipeline |
| Environment configs (dev/stage/prod) | Same code, different targets | Safe promotion |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Batch — analytics tolerate minutes-to-hours latency | Business needs seconds-level reaction |
| Streaming — continuous freshness needed | Team lacks streaming expertise and latency need is minutes |
| ELT — warehouse is powerful, raw history valuable | Sensitive fields must not land unmasked |
| Few large tasks — simple, small data | Failures are frequent and costly to redo |
| Many small tasks — clear failure isolation | Orchestrator overhead dominates |

### 9. Common mistakes
- Hard-coding dates/paths → cannot backfill → parameterize by logical date.
- Append loads with retries → duplicates → idempotent merge/overwrite.
- No validation → bad data spreads silently → add row-count, null and freshness checks.
- Treating "real-time" literally → needless complexity → confirm the real latency tolerance.
- No monitoring until production → stale data for weeks → build observability in from the start.
- Mixing time zones → daily totals shift → store UTC, convert for display.

### 10. 🎯 Interview kit
**30-second answer:** A pipeline moves data from sources to a destination through ingest, stage, transform and load. I design each task to be idempotent and parameterized, stage before publishing, load incrementally, validate with quality gates, orchestrate with retries and alerts, and monitor freshness and volume.

**Structure a longer answer:** clarify (sources, latency, volume, consumers) → draw stages → choose batch/stream and ETL/ELT → reliability patterns → monitoring → scenario.

**Likely follow-ups:**
- *ETL or ELT?* → depends on warehouse capability, privacy needs, and desire to keep raw data.
- *How do you make it rerunnable?* → idempotent writes and logical-date parameters.
- *How do you handle bad records?* → quarantine with reason, alert on thresholds.
- *Push vs pull ingestion?* → pull is simple but loads the source; push/CDC reduces source load, needs a reliable landing.

**Signals interviewers look for:** you think about failure and recovery first, ask about latency requirements, and name concrete patterns instead of tools.

### 11. Check yourself
- [ ] Sketch the stages of a pipeline and mark where validation and idempotency belong.
- [ ] Explain staging and atomic promotion in two sentences.
- [ ] List four things you would monitor.
- [ ] Argue for or against ELT for a case with PII.
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)
- [Apache Kafka](<../../../TOOLS/Apache_Kafka/_index.md>)

## Practice questions

- [Basic] [What is a DAG and how does it represent dependencies between tasks in a pipeline?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12845_What_is_a_DAG_and_how_does_it_represent_dependencies_between.md>)
- [Intermediate] [Why might you choose Parquet over CSV for a pipeline that feeds an analytics warehouse?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12890_Why_might_you_choose_Parquet_over_CSV_for_a_pipeline_that_fe.md>)
- [Intermediate] [Why might you deliberately denormalize in a warehouse pipeline, and what maintenance burden does that design choice create downstream?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-13140_Why_might_you_deliberately_denormalize_in_a_warehouse_pipeli.md>)
- [Deep] [How would you design a pipeline to enforce PII masking and data-residency rules for EU vs India users in the same flow?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-12964_How_would_you_design_a_pipeline_to_enforce_PII_masking_and_d.md>)
- [Deep] [Walk me through how you'd design incremental materialization in dbt for a huge fact table to avoid full rebuilds.](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-13015_Walk_me_through_how_you_d_design_incremental_materialization.md>)
- [Deep] [You must join a 2TB fact table with a 5GB dimension in Spark and it keeps spilling to disk; how do you redesign the pipeline stage?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-13058_You_must_join_a_2TB_fact_table_with_a_5GB_dimension_in_Spark.md>)
- [Deep] [Design a pipeline that computes rolling 30-day metrics efficiently without rescanning 30 days of raw data every run.](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-13101_Design_a_pipeline_that_computes_rolling_30-day_metrics_effic.md>)
- [Deep] [Design a pipeline that must ingest data whose volume and schema vary wildly by tenant; how do you keep one pipeline maintainable across all?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Q-13139_Design_a_pipeline_that_must_ingest_data_whose_volume_and_sch.md>)

## Related concepts

- [Graceful Degradation in Streaming](<../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Graceful_Degradation_in_Streaming.md>) *(Streaming Architecture)*
- [File and Table Formats for Features](<../Pipeline_Orchestration/File_and_Table_Formats_for_Features.md>) *(Pipeline Orchestration)*
- [Validation Coverage and Quarantine](<../../Data_Quality/Validation/Validation_Coverage_and_Quarantine.md>) *(Validation)*

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Pipelines as code
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Reliability and Durability
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › DataOps
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Orchestration
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Data Engineering Fundamentals › 2. Data Engineering Fundamentals
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Building End-to-End Data Pipelines › 13. Building End-to-End Data Pipelines
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Operationalizing Data Pipelines › 14. Operationalizing Data Pipelines
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Automate Your Pipeline Tests › Chapter 8. Automate Your Pipeline Tests
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Automate Your Pipeline Tests › Prefer Textual Data Formats over Binary for Testing
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Data Engineering for Autonomy and Rapid Innovation › Create and Support Staging Tables
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Data Pipelines---Design Patterns for Reusability, Extensibility Evolve Data Pipelines for Quality › Chapter 22. Data Pipelines---Design Patterns for Reusability, Extensibility: Evolve Data Pipelines for Quality, Flexibility, Transparency, and Growth
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Preparing for technical interviews › Data Pipeline Design for Data Engineers
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Data pipeline foundations
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Data pipeline foundations › Key components of a data pipeline
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Steps to design your data pipeline
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Data consistency
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Event Driven
- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › Stateful pipelines and very late data
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Flow Design Patterns › Sequence
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Prerequisite Knowledge for Aspiring Data Engineers › High-Level Architecture Overview of Data Processing Pipelines
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Chapter 6. Data Operations and Support
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Analyzing Data from Operational Data Stores Using Amazon Redshift › Data Pipeline Resiliency
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Event-Driven Pipeline Maintenance with EventBridge

