---
id: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Pipeline_Data_Monitoring"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration"
tools: ["Apache_Spark", "Apache_Airflow"]
---

# Pipeline Data Monitoring

**L4 Advanced** · [↑ Pipeline Orchestration](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Monitoring a pipeline means watching the *data* (is it fresh, complete, shaped right, and plausible?) and not just the *jobs* (did they exit with code 0?).

### 1. The big picture
A pipeline can be green in the orchestrator UI and still deliver garbage: the job ran, but the source sent half the rows, an upstream team renamed a column, or yesterday's file arrived again. Job monitoring answers "did it run?"; **data monitoring** answers "can I trust what it produced?" Production teams need both: infrastructure observability (CPU, memory, task failures, retries) and data observability (freshness, volume, schema, distribution, quality).

This topic also covers *how downstream pipelines know upstream data is ready*, because "ready" is itself a monitored fact: readiness markers, batching models, and event-time windows determine what a given batch means and when it is safe to consume.

### 2. Core concepts and vocabulary
- **Freshness** — how recent the newest data is relative to now or to the SLA.
- **Volume / completeness** — row counts and partition counts compared with expectations.
- **Schema** — column names, types, and nullability matching the contract.
- **Distribution** — value ranges, null rates, distinct counts, categorical mix; detects "valid but wrong" data.
- **Uniqueness / validity / referential integrity** — keys not duplicated, values in allowed sets, foreign keys resolved.
- **Readiness marker** — a signal that a dataset is complete: a flag file (`_SUCCESS`), a new commit in a Delta/Iceberg log, or a conventional partition path.
- **Data time vs arrival time** — `data_timestamp` is inside the record; `arrival_timestamp` is when your system received it.
- **Data Time Window (DTW) batching** — a batch contains all records whose data timestamp falls in a window.
- **Arrival Time Window (ATW) batching** — a batch contains whatever arrived in a window.
- **Watermark / late data** — the engine's estimate of how late events can be; events older than it are late.
- **Data observability** — tools that learn baselines and alert on anomalies (Monte Carlo, Acceldata, Great Expectations suites, dbt tests).

### 3. How it works, step by step
1. **Define expectations** per dataset: freshness SLA, row-count bounds, null thresholds, key uniqueness, allowed values.
2. **Instrument at boundaries**: after ingestion, after each transform layer (bronze/silver/gold), and before publishing.
3. **Compute metrics** as part of the pipeline (row counts, min/max timestamps, null %, duplicates) and store them in a metrics table so trends are queryable.
4. **Evaluate**: compare to fixed rules (`rows > 0`, `max(ts) > now - 2h`) and to history (z-score or % deviation from the same weekday).
5. **Act**: fail the task (blocking), warn (non-blocking), or route to quarantine depending on severity.
6. **Signal readiness** only after checks pass — write the flag file or commit as the final step.
7. **Alert and own**: notifications go to the dataset owner with the failing check and the run link.

A scripted task in Airflow should exit non-zero on validation failure so the orchestrator retries or alerts; the UI then shows status, logs, and retry history, and email/Slack alerts can be attached.

### 4. Worked example
```sql
-- Metrics captured after each load of silver.orders for load_date = '2026-10-12'
INSERT INTO ops.dq_metrics (dataset, run_date, metric, value, captured_at)
SELECT 'silver.orders', DATE '2026-10-12', m.metric, m.value, CURRENT_TIMESTAMP
FROM (
  SELECT 'row_count'    AS metric, COUNT(*)::float AS value FROM silver.orders WHERE order_date = DATE '2026-10-12'
  UNION ALL
  SELECT 'null_customer_pct', 100.0 * AVG((customer_id IS NULL)::int) FROM silver.orders WHERE order_date = DATE '2026-10-12'
  UNION ALL
  SELECT 'dup_order_ids', COUNT(*) - COUNT(DISTINCT order_id) FROM silver.orders WHERE order_date = DATE '2026-10-12'
  UNION ALL
  SELECT 'minutes_since_latest_event',
         EXTRACT(EPOCH FROM (NOW() - MAX(event_ts)))/60 FROM silver.orders
) m;

-- Gate: compare today's volume to the median of the previous 7 same-weekday values
SELECT CASE WHEN today.value < 0.6 * hist.median THEN 'FAIL' ELSE 'OK' END AS volume_check
FROM (SELECT value FROM ops.dq_metrics WHERE dataset='silver.orders' AND metric='row_count' AND run_date = DATE '2026-10-12') today,
     (SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY value) AS median
      FROM ops.dq_metrics WHERE dataset='silver.orders' AND metric='row_count'
        AND run_date IN (DATE '2026-10-05', DATE '2026-09-28', DATE '2026-09-21')) hist;
```
Read it as: the load writes a handful of cheap numbers; a gate compares today with a seasonal baseline; only then does the final step create `_SUCCESS`. A volume drop to 60% of normal fails the check even though no row was malformed — exactly the failure that job-level monitoring misses.

### 5. Under the hood
- **Readiness marker implementations:** (a) an event — Spark writes `_SUCCESS` for file formats, Delta Lake appends a commit to its transaction log; (b) a conventional partition path where a time-partitioned location (e.g., `dt=2026-10-12/`) is only consumed after the next partition exists or a marker is written. Marker semantics depend on the storage layout; with object stores, renames aren't atomic, which is why a final marker is more reliable than "files appeared".
- **Batching model choice (97 Things):** DTW gives analysts a guarantee that all records for a window are present but is unpredictable when records arrive out of order; ATW is predictable and simple for push-based sources but a batch may contain records from different business times. Pull-based extraction with a filter on a data timestamp suits DTW.
- **Event-time windows:** aggregations by event time (e.g., five-minute counts per sensor with `window("eventTime", "5 minute")`) avoid transit-delay skew; the watermark bounds how long state is kept for late events.
- **Late data detection:** Spark Structured Streaming detects and drops late events but doesn't expose them; Flink allows capturing them. A partition-based event-time tracker should avoid `MIN` over partitions to avoid a "stuck in the past" stateful job.

### 6. Performance and design implications
- Full-table profiling every run is costly; compute metrics on the new partition only.
- Store metrics as numbers in a narrow table: cheap, trendable, and usable by dashboards.
- Over-alerting trains people to ignore alerts; use severity tiers and require an owner per dataset.
- Static thresholds break with seasonality; use same-weekday baselines or simple percent-change bands before reaching for ML.
- Typical SLAs: freshness within one pipeline interval + tolerance; volume ±20–40% depending on volatility.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `_SUCCESS` / commit log | Marks a write as complete | Cross-team readiness signal |
| Airflow `email_on_failure`, callbacks, SLAs | Alert on task failure or lateness | Operational alerting |
| Great Expectations / Deequ / dbt tests | Declarative data tests | Row-level and aggregate checks |
| `withWatermark("ts","10 minutes")` | Bound late data in streaming | Event-time aggregations |
| Metrics table + dashboard | Trends and baselines | All production datasets |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Blocking gate before publish — bad data is costly | Checks are flaky; you'll block healthy loads |
| Non-blocking warning — exploratory data | Consumers trust the table implicitly |
| DTW batching — completeness matters to analysts | Source is push-based and chaotic |
| ATW batching — predictable cost/latency | Business windows must be exact |
| Learned anomaly detection — many tables | Few critical tables where explicit rules are clearer |

### 9. Common mistakes
- Only monitoring job success → silent partial loads → add volume and freshness checks.
- Signaling readiness before validation → consumers read bad data → marker as last step.
- Filtering by arrival time when business needs event time → numbers shift on reruns → choose the batching model deliberately.
- Alerts with no owner or runbook → ignored → attach owner and next action to each check.
- Hard-coded thresholds across weekends/holidays → false alarms → seasonal baselines.

### 10. 🎯 Interview kit
**30-second answer:** Pipeline monitoring has two layers: job health (status, duration, retries) and data health (freshness, volume, schema, distribution, uniqueness). I compute cheap metrics per load, gate publication on rules and historical baselines, signal readiness with a marker only after checks pass, and alert the dataset owner.

**Structure a longer answer:** clarify (SLAs, consumers, tolerance) → core mechanism (metrics, gates, readiness marker) → trade-offs (blocking vs warn, static vs learned) → real scenario (half the rows arrived).

**Likely follow-ups:**
- *How does a downstream team know data is ready?* → readiness marker/commit/partition convention, or an orchestrator dataset trigger.
- *Data time vs arrival time?* → DTW vs ATW trade-offs.
- *How do you deal with late data?* → watermark, reprocess affected windows, capture late records.
- *How do you avoid alert fatigue?* → severity tiers, owners, baselines.

**Signals interviewers look for:** distinguishes jobs from data, thinks about SLAs and ownership, handles late data deliberately.

### 11. Check yourself
- [ ] List five data-quality dimensions and a check for each.
- [ ] Explain DTW vs ATW with an example.
- [ ] Design a readiness marker for an object-store dataset.
- [ ] Explain why a pipeline can be green and wrong.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)
- [Apache Airflow](<../../../TOOLS/Apache_Airflow/_index.md>)

## Practice questions

- [Basic] [What is a data pipeline and why do AIOps teams need one to collect metrics logs and traces from many systems?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13343_What_is_a_data_pipeline_and_why_do_AIOps_teams_need_one_to_c.md>)
- [Basic] [Why is it useful to give tasks clear meaningful task_ids and what makes a bad task_id?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13565_Why_is_it_useful_to_give_tasks_clear_meaningful_task_ids_and.md>)
- [Intermediate] [How would you build a simple pipeline that pulls CPU metrics every hour and stores them for later anomaly detection?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13381_How_would_you_build_a_simple_pipeline_that_pulls_CPU_metrics.md>)
- [Intermediate] [Why is it a bad idea to hardcode today's date inside a task instead of using the run's logical date?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13551_Why_is_it_a_bad_idea_to_hardcode_today_s_date_inside_a_task_.md>)
- [Deep] [What is the difference between event time and processing time in a metrics pipeline?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13412_What_is_the_difference_between_event_time_and_processing_tim.md>)
- [Deep] [How would you enforce a data contract between the telemetry-producer teams and your Spark anomaly pipeline so a breaking metric-label change can't silently poison detection?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13494_How_would_you_enforce_a_data_contract_between_the_telemetry-.md>)
- [Deep] [How would you set up CI for a Spark pipeline so pull requests run transformation unit tests on sampled data without a full cluster?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13703_How_would_you_set_up_CI_for_a_Spark_pipeline_so_pull_request.md>)
- [Deep] [How do you handle late-arriving data in a windowed feature aggregation so retrained models don't miss or double-count events?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13152_How_do_you_handle_late-arriving_data_in_a_windowed_feature_a.md>)

## Related concepts

- [Pipeline Design Patterns](<../Pipeline_Design/Pipeline_Design_Patterns.md>) *(Pipeline Design)*
- [End to End No Data Loss](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/End_to_End_No_Data_Loss.md>) *(Data Architecture)*
- [Computing SLIs with Spark](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Computing_SLIs_with_Spark.md>) *(Big Data Processing)*
- [Latency Budgets in Streaming Pipelines](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Latency_Budgets_in_Streaming_Pipelines.md>) *(Big Data Processing)*

## References

- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Ingestion Design Patterns › Solution
- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › Event and Processing Time
- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › Examples
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Batch Processing Patterns › 6. Batch Processing Patterns
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Operationalizing Data Pipelines › 14. Operationalizing Data Pipelines
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Be Intentional About the Batching Model in Your Data Pipelines › Chapter 9. Be Intentional About the Batching Model in Your Data Pipelines
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Aggregations with Event-Time Windows
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Managing, Deploying, and Scaling Machine Learning Pipelines with Apache Spark › Tracking
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Enriching Data Catalogs with Data Classification › Example: Building a Petabyte-Scale Log Analytics Solution on AWS
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Monitoring › Monitoring logs and traces

