---
id: "02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Managed_Ingestion_and_Re-syncs"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion"
tools: ["AWS"]
---

# Managed Ingestion and Re-syncs

**L4 Advanced** · [↑ Data Ingestion](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Ingestion is a design problem of questions (what, how often, how much, in what format, with what guarantees); managed tools answer many of them for you, but you remain responsible for quality at the boundary, schema change, re-syncs and recovery.

### 1. The big picture
Before choosing a tool, an ingestion design asks a fixed set of questions: What are the use cases for this data, and can one copy serve many? Is the source reliable and available when we need it? Where does the data land? How often will it be read, and in what volume will it arrive? What format is it in, and can downstream storage and transformation handle it? Is it in good shape for immediate use, and for how long? For streaming sources, must it be transformed in flight?

Managed ingestion tools (Fivetran, Airbyte, cloud-native services) take on connectors, state, retries and schema handling. They shorten time to value but they don't remove the need for: data quality gates at the boundary, deliberate handling of schema change, re-sync (backfill) procedures, and cost control.

### 2. Core concepts and vocabulary
- **Raw / bronze layer** — landing area that mirrors the source with minimal transformation, kept so you can reprocess.
- **Re-sync (historical sync / backfill)** — re-extracting data for a table or time range, either because of a bug, schema change or a recovered outage.
- **Schema drift** — the source changes (new column, type change, renamed or dropped column) without coordination.
- **Dead-letter queue / quarantine** — place where records that fail validation are parked for inspection, rather than blocking the load.
- **Late-arriving data** — records whose event time is older than the batch window they arrive in.
- **Truncate-and-load** — empty the target then reload; **incremental merge** — upsert changes.
- **Data reliability at ingestion** — the practice of testing and cleaning data as it enters, because defects caught at the boundary are cheapest.
- **External trigger pattern** — start a pipeline when a notification event arrives (a file landed, an upstream published) instead of on a clock.

### 3. How it works, step by step
**A well-run managed ingestion flow**
1. **Define requirements** (freshness, volume, consumers, retention, sensitivity).
2. **Configure connection and streams**; select only needed tables/columns.
3. **Initial historical sync** into raw tables; verify counts.
4. **Incremental syncs** on a schedule or trigger.
5. **Boundary checks:** schema validation, null/uniqueness checks on keys, row-count and volume anomaly detection, freshness checks.
6. **Route failures:** bad records to quarantine, alert owner, continue where safe.
7. **Transform downstream** in the warehouse (ELT) from raw to curated.
8. **Re-sync when needed** with an agreed procedure (below).

**Re-sync procedure for one table**
1. Decide scope (full table vs date range) and impact (source load, API quota, cost).
2. Pause or flag downstream consumers if they would read inconsistent data.
3. Run the re-sync into the raw table (or a shadow table).
4. Validate counts and checksums against the source.
5. Swap/merge into the live table; resume consumers.
6. Record the event and the reason.

### 4. Worked example
Source volume suddenly doubles overnight (marketing campaign). What breaks and how do you respond?
- Extract and load jobs run longer; API rate limits or warehouse concurrency are hit; downstream models miss their window.
- Response: check pipeline lag and queue depth; scale compute temporarily or increase parallelism; verify no truncation; confirm quality checks still pass (volume anomaly rules should be adaptive or should alert, not silently pass); review whether batch size and timeouts are right; for the longer term, move from fixed-time schedule to event-driven triggers and add autoscaling.

Handling a column that is renamed across file batches:

```sql
-- Raw layer keeps both; curated layer coalesces
SELECT COALESCE(cust_id, customer_id) AS customer_id, ...
FROM raw.orders_files;
```

with an explicit mapping table or schema-evolution rule recorded in the repo so the rename is documented and tested.

Truncate-and-load vs merge for a mid-size dimension changing ~5% daily: truncate-reload is simple, self-healing and fine at modest sizes, but it creates a window where the table is empty (fix by loading to a shadow table and swapping) and discards history; merge touches only the 5% and supports history/SCD patterns, but requires reliable keys, change detection and delete handling.

### 5. Under the hood
- **Quality at the boundary.** Defects are cheapest to catch where data enters: issues found downstream require more rework, and trust erodes. Typical checks: schema conformance, required fields, referential keys, accepted values, duplicate keys, volume and freshness compared to a baseline.
- **Schema change strategies.** Options: (a) strict — fail on unexpected change and alert; (b) additive — allow new columns, block removals/type changes; (c) permissive — evolve automatically. Raw layers often accept all changes (store as variant/JSON or add columns) while curated layers enforce contracts, which separates "capture everything" from "serve reliably".
- **Late and duplicate data.** At-least-once delivery produces duplicates; dedupe on a business key plus a source version/position. For late-arriving events, either reprocess recent partitions (lookback window) or write to event-time partitions and recompute affected aggregates.
- **Event-driven vs scheduled.** A scheduled pipeline wastes compute when nothing changed and may start before data is ready. An event-driven external trigger (message bus notification, file-arrival event, API webhook) runs only when something new exists. The cost is more infrastructure and handling of missed or duplicate events (add a periodic safety-net run).
- **Retries.** Flaky partner APIs need bounded retries with exponential backoff and jitter, timeouts, and idempotency keys so repeated requests do not create duplicates. Retrying without backoff can amplify an outage.
- **Backfill gaps.** Re-syncs must know all gaps; partial manual backfills often miss ranges. Drive them from a computed list of missing partitions rather than memory.

### 6. Performance and design implications
- Selecting fewer columns/tables lowers cost, transfer, and risk.
- Large re-syncs compete with regular syncs and the source's production workload; schedule them and throttle.
- Quarantine tables prevent a few bad records from blocking millions, but unmonitored quarantines become silent data loss; alert on quarantine volume.
- Track freshness and completeness SLOs per table; show them to consumers.
- Time zones and DST can break date-partitioned incremental loads (a day with 23 or 25 hours); store and partition in UTC.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| Connector "re-sync/historical sync" per table | Re-extracts a table | After bug, schema change, or data repair |
| Schema change policy (allow/block/alert) | Governs drift | Protect curated layer |
| Quarantine / error table | Stores rejected rows | Bad-record handling |
| Event/webhook trigger or sensor | Starts runs on arrival | Irregular or unpredictable arrivals |
| `MERGE`, partition overwrite, table swap | Safe load mechanics | Incremental and truncate-load alternatives |
| Backoff and idempotency key settings | Safe retries | Flaky or rate-limited APIs |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Managed ingestion for standard sources and small teams | Unusual sources, strict cost control, special semantics |
| ELT: land raw then transform in warehouse | Sensitive data that must be masked before landing |
| Truncate-and-load for small tables | Large tables, history needed, long empty windows |
| Incremental merge for big mutable tables | No reliable key or change signal |
| Event-driven triggers for unpredictable arrivals | Highly regular, predictable loads where cron is enough |

### 9. Common mistakes
- Ingesting every table and column "just in case" → cost, PII sprawl → select intentionally.
- No boundary tests → garbage flows downstream → add schema, null, key, volume and freshness checks.
- Re-sync without validating → silent mismatch → counts and checksums before swapping.
- Ignoring late data → daily totals revised unnoticed → lookback windows and reprocessing rules.
- Letting schema drift auto-propagate into curated models → downstream breakage → contract and review at curated layer.
- Retrying endlessly without backoff → worse outage → bounded backoff with jitter.

### 10. 🎯 Interview kit
**30-second answer:** Managed ingestion handles connectors, state and retries, but I still define requirements, select streams, validate quality at the boundary, handle schema drift deliberately, and keep a tested re-sync procedure with validation. I prefer raw landing plus ELT, quarantine for bad records, and event-driven triggers where arrival is unpredictable.

**Structure a longer answer:** requirements → configuration and landing → boundary checks → schema change policy → re-sync/backfill → monitoring and cost.

**Likely follow-ups:**
- *Truncate-reload vs merge?* → size, history, delete handling, downtime window.
- *Detect and handle late data?* → compare event time to partition; lookback and reprocess.
- *Schema drift breaks ETL?* → strict vs additive policies, raw-vs-curated separation, alerts.
- *Duplicates from at-least-once?* → dedupe by business key + version, idempotent writes.

**Signals interviewers look for:** you think in requirements and failure modes, not tool features.

### 11. Check yourself
- [ ] List six questions to ask before designing ingestion for a new source.
- [ ] Write a re-sync runbook for one table including validation.
- [ ] Contrast strict, additive and permissive schema-change policies.
- [ ] Explain why raw landing is kept even when curated tables exist.
- [ ] Describe an event-driven trigger and its safety net.
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Basic] [What is the difference between a source and a destination (connector) in Airbyte or Fivetran?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11644_What_is_the_difference_between_a_source_and_a_destination_co.md>)
- [Intermediate] [What happens to your pipeline when a new column is added to the source table?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11661_What_happens_to_your_pipeline_when_a_new_column_is_added_to_.md>)
- [Intermediate] [What is data type mapping between a source and destination, and what can go wrong?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11702_What_is_data_type_mapping_between_a_source_and_destination_a.md>)
- [Intermediate] [Your Salesforce connector fell behind by two days because of a schema change. Walk me through recovery so downstream SLAs are met.](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11782_Your_Salesforce_connector_fell_behind_by_two_days_because_of.md>)
- [Deep] [What happens when a source column changes its data type mid-stream (e.g., int to string)?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11703_What_happens_when_a_source_column_changes_its_data_type_mid-.md>)
- [Deep] [How would you architect ingestion for 500 customer databases with identical schemas but strict per-tenant isolation requirements?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11751_How_would_you_architect_ingestion_for_500_customer_databases.md>)
- [Deep] [Describe how you'd load-test a custom connector before production, including edge cases like pagination cursors expiring mid-run.](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11784_Describe_how_you_d_load-test_a_custom_connector_before_produ.md>)
- [Deep] [Your warehouse merge step is the bottleneck, not the extraction. How do you tune destination-side ingestion performance?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Q-11818_Your_warehouse_merge_step_is_the_bottleneck_not_the_extracti.md>)

## Related concepts

- [End to End No Data Loss](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/End_to_End_No_Data_Loss.md>) *(Data Architecture)*
- [Pipeline Design Patterns](<../Pipeline_Design/Pipeline_Design_Patterns.md>) *(Pipeline Design)*
- [Orchestration vs Processing](<../Pipeline_Orchestration/Orchestration_vs_Processing/_index.md>) *(Pipeline Orchestration)*
- [Airflow Architecture](<../../../TOOLS/Apache_Airflow/Airflow_Core/Airflow_Architecture.md>) *(Airflow Core)*
- [File and Table Formats for Features](<../Pipeline_Orchestration/File_and_Table_Formats_for_Features.md>) *(Pipeline Orchestration)*

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Key engineering considerations for the ingestion phase
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Ingestion
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Key Engineering Considerations for the Ingestion Phase
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Message and Stream Ingestion Considerations
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Collecting, Cleaning, Transforming, and Testing Data › Collecting Data
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Architecting for Data Reliability › Measuring and Maintaining High Data Reliability at Ingestion
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Architecting for Data Reliability › Data Ingestion
- *Data Engineering Fundamentals* — Zhaolong Liu — CHAPTER 2Data Ingestion and Acquisition › CHAPTER 2Data Ingestion and Acquisition(#contents.xhtmlch02a)
- *Data Engineering Fundamentals* — Zhaolong Liu — CHAPTER 9Real-world Data Engineering Use Cases › CHAPTER 9Real-world Data Engineering Use Cases(#contents.xhtmlch09a)
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Data Ingestion Patterns in Data Engineering › 4. Data Ingestion Patterns in Data Engineering
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Chapter 2. Data Ingestion Design Patterns
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Not Only Ingestion
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Data Readiness
- *Data Engineering Design Patterns* — Bartosz Konieczny — C › D
- *Data Pipelines Pocket Reference* — James Densmore — A Modern Data Infrastructure › Ingestion Interface and Data Structure
- *Data Pipelines Pocket Reference* — James Densmore — A Modern Data Infrastructure › Data Ingestion Tools
- *Data Pipelines Pocket Reference* — James Densmore — Data Ingestion Loading Data › Chapter 5. Data Ingestion: Loading Data
- *Data Pipelines Pocket Reference* — James Densmore — Transforming Data › When to Transform? During or After Ingestion?

