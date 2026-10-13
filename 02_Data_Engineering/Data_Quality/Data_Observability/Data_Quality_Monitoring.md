---
id: "02_Data_Engineering/Data_Quality/Data_Observability/Data_Quality_Monitoring"
level: 4
type: "concept"
topic: "02_Data_Engineering/Data_Quality/Data_Observability"
tools: ["Prometheus", "AWS"]
---

# Data Quality Monitoring

**L4 Advanced** · [↑ Data Observability](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Data observability tells you whether data is *moving* correctly (fresh, complete in volume, structurally intact, traceable), while data quality monitoring checks whether the *values* are right — you need both, wired into incident management with clear owners and severity.

### 1. The big picture
Data breaks silently: a job succeeds but loads zero rows, a schema changes upstream, a column fills with nulls, a dashboard shows last week's numbers. Observability is the practice of continuously monitoring, understanding and ensuring the health of data systems so problems are found before business users find them. The books split the idea into two layers:

- **Data observability** — metadata-driven: is data flowing through the warehouse on time and in expected shape? It uses catalog metadata, job monitoring and lineage. It is relatively easy to roll out across an entire warehouse but ignores *contents*.
- **Data quality monitoring** — queries the data itself: are values valid, consistent, unique, accurate? It requires domain expertise or ML and is hard to scale. Using only observability is like watching the water pressure in a pipe without testing whether the water is clean.

A mature setup monitors five signals — **freshness, volume, schema, distribution, lineage** — across the *whole* stack (sources, streams, lake, warehouse, transformations, dashboards, ML features), because data can break anywhere.

### 2. Core concepts and vocabulary
- **Freshness** — how recently data was updated relative to expectation.
- **Volume** — row counts or bytes arriving/produced.
- **Schema** — columns, types, constraints; changes in them.
- **Distribution** — statistical properties: null rate, min/max, mean, cardinality, categorical frequencies.
- **Lineage** — graph of upstream/downstream dependencies; used for root cause and impact analysis.
- **Data downtime** — periods when data is missing, wrong or late; measurable as a KPI.
- **SLI / SLO** — measurable indicator and target (e.g., 99.5% of daily loads complete by 07:00).
- **Incident vs outage** — an incident is any quality/pipeline event needing response; an outage is an incident that makes the data unusable for its consumers.
- **Severity level** — classification by business impact driving response time.
- **OpenLineage** — open standard for emitting lineage/run metadata (job, run, dataset events: START, COMPLETE, FAIL).
- **Data contract** — agreed schema/semantics between producer and consumer, enforceable and observable.

### 3. How it works, step by step
1. **Inventory and tier assets.** Catalog tables, owners, consumers; classify by criticality (e.g., revenue reporting = tier 1).
2. **Collect metadata cheaply.** Pull table update times, row counts, query history, schema versions from warehouse system views (Snowflake `INFORMATION_SCHEMA`/`ACCOUNT_USAGE`, BigQuery `INFORMATION_SCHEMA`, Delta/Iceberg history).
3. **Define monitors per pillar:** freshness (expected cadence), volume (expected range), schema change (any diff), distribution (null %, value ranges, category share), referential integrity (orphan keys).
4. **Set thresholds** — static for hard rules, learned/dynamic (time-series model, rolling window with seasonality) for volume and distributions.
5. **Add lineage** (OpenLineage events from Airflow/Spark/dbt) so alerts show upstream cause and downstream blast radius.
6. **Route alerts** to one owner per alert, with severity and runbook.
7. **Manage incidents:** triage, communicate status, fix, post-mortem; track time to detect, time to resolve, data downtime.
8. **Report** via a data health page / scorecard for non-technical stakeholders.

### 4. Worked example
```sql
-- Freshness: hours since last successful load per critical table
SELECT table_name,
       DATEDIFF('hour', MAX(loaded_at), CURRENT_TIMESTAMP()) AS hours_stale
FROM   ops.load_audit
WHERE  table_name IN ('fct_orders','dim_customer')
GROUP  BY table_name
HAVING hours_stale > 26;                  -- daily job with slack

-- Volume: compare today's row count to same weekday's trailing 4-week median
WITH hist AS (
  SELECT load_date, rows_loaded
  FROM ops.load_audit WHERE table_name = 'fct_orders' AND load_date >= CURRENT_DATE - 35
)
SELECT t.rows_loaded,
       m.med,
       t.rows_loaded / NULLIF(m.med,0) AS ratio
FROM (SELECT rows_loaded FROM hist WHERE load_date = CURRENT_DATE) t,
     (SELECT MEDIAN(rows_loaded) AS med FROM hist
      WHERE DAYOFWEEK(load_date) = DAYOFWEEK(CURRENT_DATE) AND load_date < CURRENT_DATE) m;

-- Referential integrity without enforced FKs
SELECT count(*) AS orphans
FROM fct_orders f LEFT JOIN dim_customer d ON f.customer_key = d.customer_key
WHERE d.customer_key IS NULL;

-- Source-to-target reconciliation
SELECT (SELECT count(*) FROM src.orders WHERE order_date = :d)
     - (SELECT count(*) FROM dwh.fct_orders WHERE order_date = :d) AS diff;
```
How to read: a ratio under ~0.7 or over ~1.5 flags a partial load or duplicate load; non-zero orphans mean a dimension load lagged or a key mapping broke. Alert text should state *what*, *where in the lineage*, *who owns it*.

### 5. Under the hood
- **Why metadata-first:** counting rows and reading update timestamps costs almost nothing compared with profiling every column, so observability scales across thousands of tables; deep content checks are applied selectively to critical tables.
- **Anomaly detection for volume:** model expected counts with day-of-week/seasonality (e.g., Prophet-like or robust rolling quantiles); alert when the latest observation falls outside a prediction interval. Fixed thresholds break with growth or seasonality; rolling windows adapt but absorb prolonged outages into the baseline.
- **OpenLineage:** emits run events with job, run ID, input and output datasets and facets (schema, row counts, quality metrics). Consumers (Marquez, DataHub, Atlan) assemble the lineage graph.
- **Data contracts:** a producing microservice publishes schema (Avro/Protobuf/JSON Schema) and expectations in a registry; CI blocks incompatible changes; ingestion validates and reports violations as observable events.
- **PII/governance observation:** classify columns via pattern/ML scanners and tags; alert when a tagged-sensitive pattern appears in an untagged column or a new column appears in a governed table.
- **Cost/storage anomalies:** monitor bytes written per job, object counts and warehouse credits; a 10x spike usually signals a runaway join, duplicate load or missing partition filter.
- **Partner feeds:** monitor delivery timestamp, file counts, checksum, schema, and receipt acknowledgement against the contract.

### 6. Performance and design implications
- Monitoring queries should themselves be cheap: use metadata views, sample or aggregate incrementally by partition.
- Alert precision matters more than coverage: too many false positives teach people to ignore alerts (alert fatigue) — tier thresholds by dataset criticality.
- Route each alert to exactly one audience to preserve ownership.
- Make status visible: a health page with per-product status, freshness, last incident, owner, and plain-language impact.
- Measure the programme: data downtime, mean time to detect/resolve, incidents per week, % critical tables with monitors.
- dbt: surface failing tests as tagged, routed alerts with owner and model lineage, not log lines.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| Warehouse `INFORMATION_SCHEMA` / `ACCOUNT_USAGE` | Update times, row counts, query history | Metadata monitors |
| Delta `DESCRIBE HISTORY`, Iceberg `snapshots` | Commit times and operations metrics | Lake freshness/volume |
| dbt `source freshness`, tests | Freshness and rule checks | Transformation layer |
| OpenLineage integration (Airflow, Spark) | Emit lineage/run events | Lineage graph |
| Great Expectations / Soda / Deequ | Content checks | Critical tables |
| Alert routing (PagerDuty, Slack) | Notify owner | Severity-based response |
| Data catalog tags | Criticality, PII, owner | Policy-driven monitoring |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Metadata observability everywhere | You believe it replaces value-level checks |
| Deep rule-based checks on tier-1 | Rules proliferate without owners |
| ML anomaly detection | Small history/low volume — fixed rules are clearer |
| Block pipeline on failure | Business prefers last-known-good data with a warning |

### 9. Common mistakes
- Monitoring only the warehouse → problems upstream/downstream missed → extend to lake, streams, BI, ML.
- Same alert to multiple teams → low ownership/duplicate work → one owner.
- No severity model → everything pages → define levels by business impact and SLA.
- Alerts without lineage/context → slow triage → include upstream/downstream and runbook.
- Treating all tables equally → cost and noise → criticality tiers.
- No post-incident learning → recurrence → track and review.

### 10. 🎯 Interview kit
**30-second answer:** Data observability monitors the flow of data — freshness, volume, schema, distribution, lineage — mostly from metadata, while data quality monitoring queries values for validity and accuracy. I tier assets, monitor the five pillars across the full stack, route alerts to a single owner with severity and lineage context, and measure data downtime.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.

**Likely follow-ups:**
- *Incident vs outage?* → outage = consumers cannot use the data; severity from business impact.
- *Non-technical status page?* → per data product: status, freshness, last issue, owner, plain-language impact.
- *Role of OpenLineage?* → standard run/dataset lineage events feeding the graph.
- *Referential integrity without PK enforcement?* → anti-join orphan checks and trends.
- *Detect cost anomalies?* → bytes written, job runtime and credit baselines.

**Signals interviewers look for:** distinguishes observability vs quality; practical metrics; ownership and incident process.

### 11. Check yourself
- [ ] Name the five observability pillars and one query/metric for each.
- [ ] Explain why observability alone cannot catch a unit change in a column.
- [ ] Draft severity levels for tier-1 vs tier-3 datasets.
- [ ] Design the content of a stakeholder-readable data health page.
<!-- explainer:end -->

## Implementations (tools)

- [Prometheus](<../../../TOOLS/Prometheus/_index.md>)
- [AWS](<../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Intermediate] [What is the difference between a data incident and a data outage, and how do you set severity levels for data quality issues?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Data_Observability/Q-12744_What_is_the_difference_between_a_data_incident_and_a_data_ou.md>)
- [Intermediate] [Describe how you would build a data health status page that non-technical stakeholders can read.](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Data_Observability/Q-12809_Describe_how_you_would_build_a_data_health_status_page_that_.md>)
- [Deep] [What is the role of OpenLineage in a data observability stack and what events does it emit?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Data_Observability/Q-12750_What_is_the_role_of_OpenLineage_in_a_data_observability_stac.md>)
- [Deep] [How do you observe and enforce data contracts between a producing microservice and your ingestion pipeline?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Data_Observability/Q-12763_How_do_you_observe_and_enforce_data_contracts_between_a_prod.md>)
- [Deep] [Explain how you would monitor referential integrity across fact and dimension tables in a warehouse without primary key enforcement.](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Data_Observability/Q-12779_Explain_how_you_would_monitor_referential_integrity_across_f.md>)
- [Deep] [How do you observe and enforce PII/data-governance rules so sensitive columns are flagged when they appear unexpectedly?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Data_Observability/Q-12796_How_do_you_observe_and_enforce_PII_data-governance_rules_so_.md>)
- [Deep] [Explain how you would monitor the health of a data-sharing/exchange feed you provide to an external partner.](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Data_Observability/Q-12811_Explain_how_you_would_monitor_the_health_of_a_data-sharing_e.md>)
- [Deep] [What observability signals would you monitor to catch storage/cost anomalies (e.g. a job writing 10x normal data)?](<../../../PRACTICE/02_Data_Engineering/Data_Quality/Data_Observability/Q-12825_What_observability_signals_would_you_monitor_to_catch_storag.md>)

## Related concepts

- [Verifying Live Migrations](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Verifying_Live_Migrations.md>) *(Distributed Systems Core)*
- [Exactly Once into Observability Lakes](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Exactly_Once_into_Observability_Lakes.md>) *(Observability Data Lake)*
- [Hot vs Cold Telemetry](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Hot_vs_Cold_Telemetry.md>) *(Observability Data Lake)*
- [Quality SLAs and Rule Testing](<../Quality_Frameworks_and_SLAs/Quality_SLAs_and_Rule_Testing.md>) *(Quality Frameworks and SLAs)*
- [Mesh vs Central Observability Lake](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Mesh_vs_Central_Observability_Lake.md>) *(Observability Data Lake)*

## References

- *Automating Data Quality Monitoring* — Jeremy Stanley, Paige Schwartz — Data Quality Monitoring Strategies and the Role of Automation
- *Automating Data Quality Monitoring* — Jeremy Stanley, Paige Schwartz — Assessing the Business Impact of Automated Data Quality Monitoring
- *Automating Data Quality Monitoring* — Jeremy Stanley, Paige Schwartz — Assessing the Business Impact of Automated Data Quality Monitoring › Assessing Benefits to Stakeholders
- *Automating Data Quality Monitoring* — Jeremy Stanley, Paige Schwartz — Integrating Monitoring with Data Tools and Systems
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Architectural Patterns in Data Engineering › 3. Architectural Patterns in Data Engineering
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Data Quality Patterns › 10. Data Quality Patterns
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Building End-to-End Data Pipelines › 13. Building End-to-End Data Pipelines
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Why Data Quality Deserves Attention---Now › Understanding the "Rise of Data Downtime"
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Assembling the Building Blocks of a Reliable Data System › Monitoring across other parts of your stack
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Assembling the Building Blocks of a Reliable Data System › Step 4: Health check
- *Data Quality Fundamentals* — Barr Moses, Lior Gavish, Molly Vorwerck — Collecting, Cleaning, Transforming, and Testing Data › Sensor Data
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Quality Design Patterns › Problem
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Quality Design Patterns › Observability Versus Auditing
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Quality Design Patterns › Pattern: Online Observer
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Quality Design Patterns › Solution
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Monitoring
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Monitoring › Alerting
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Automated Data Quality Checks and Error Handling › Troubleshooting and Performance Tuning
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Observability and Discoverability › Chapter 7. Observability and Discoverability
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Observability and Discoverability › Lineage Observability
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Observability and Discoverability › Cost Observability
- *Data Governance with Unity Catalog on Databricks* — Kiran Sreekumar, Karthik Subbarao — Observability and Discoverability › Compute Observability

