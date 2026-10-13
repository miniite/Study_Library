---
id: "02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Separating_OLTP_and_Analytics"
level: 4
type: "concept"
topic: "02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP"
tools: []
---

# Separating OLTP and Analytics

**L4 Advanced** · [↑ OLTP vs OLAP](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Give analytics its own copy of the data, on its own infrastructure, so heavy scans never compete with customer transactions — and accept a controlled delay in exchange.

### 1. The big picture
Early in a company's life the production database answers everything. As data grows and the business asks historical questions, analytic queries start scanning big tables on the same server that runs checkout. The result: slow transactions, lock waits, cache churn, and occasionally an outage. Separating OLTP from analytics is the standard remedy and an inflection point in a company's data maturity.

"Separate" is a spectrum. At one end is a read replica of the same engine; at the other is a full warehouse/lakehouse fed by pipelines from many sources. The right point on the spectrum depends on data volume, freshness needs, number of sources, and team capacity. Whatever you choose, two things change: **data is copied** (so there is latency and a pipeline to operate) and **the analytic copy can be modeled differently** (star schema, columnar, history).

### 2. Core concepts and vocabulary
- **Read replica / readable secondary** — a continuously synchronized copy of the OLTP database that serves read-only queries (e.g., an Always On readable secondary).
- **ETL / ELT** — Extract–Transform–Load (transform before loading) vs Extract–Load–Transform (load raw, transform inside the warehouse).
- **Data warehouse** — an integrated, modeled, historical store designed for analytics.
- **Data lake / lakehouse** — object-storage-based store of raw and curated files, optionally with table-format ACID (Delta, Iceberg).
- **Staging area** — landing zone where extracted data sits before being merged.
- **Data latency** — delay between a change in OLTP and its visibility in analytics.
- **Blast radius** — how much breaks when one component fails; separation reduces the analytic blast radius onto OLTP.
- **Coupling** — dependence between systems (e.g., warehouse querying production directly vs consuming a change stream).
- **Workload isolation** — keeping resources (CPU, memory, I/O) separate per workload.

### 3. How it works, step by step
The decision ladder, from least to most disruptive:

1. **Tune first** — add indexes, fix bad queries, schedule reports off-hours. Cheap, but doesn't scale.
2. **Resource governance** — cap analytics CPU/memory (e.g., Resource Governor). Protects OLTP, but analytics and OLTP still share the same data pages and storage.
3. **Operational analytics on the same table** — add a nonclustered columnstore index on the OLTP table so analytic queries scan columnar structures. Good for modest workloads; adds write overhead.
4. **Readable secondary / replica** — offload read queries to a replica. Same schema, near-real-time, no modeling gains.
5. **Replication to a reporting database** — can carry its own indexes/columnstore.
6. **Warehouse / lake with ETL or ELT** — full separation: different schema, history, many sources, own compute. Most effective, most disruptive.
7. **Streaming/CDC backbone** — near-real-time feed from log-based change capture.

Typical pipeline for step 6:
1. Extract changes from the OLTP source (incremental by watermark, or CDC).
2. Land in staging (files/tables).
3. Validate and transform (dedupe, type, conform).
4. Merge into dimensions/facts idempotently.
5. Publish; update freshness metadata; alert on SLA breach.

### 4. Worked example
Offloading with a nonclustered columnstore, then deciding it isn't enough.

```sql
-- Step 3: operational analytics on the OLTP table
CREATE NONCLUSTERED COLUMNSTORE INDEX ncci_Orders
ON dbo.Orders (OrderDate, CustomerId, Status, Total);

-- Analytic query now scans the columnstore, not the B-tree:
SELECT YEAR(OrderDate) AS Yr, MONTH(OrderDate) AS Mo, SUM(Total) AS Revenue
FROM dbo.Orders
GROUP BY YEAR(OrderDate), MONTH(OrderDate);
```

Watch: (a) every INSERT/UPDATE now also maintains the columnstore — check write latency on the OLTP side; (b) the query still shares the same server's memory and CPU. If checkout latency rises during reporting, move to a readable secondary or a warehouse.

Warehouse-side load, made idempotent so a rerun doesn't double-count:

```sql
MERGE fact.Orders AS t
USING stg.Orders_Batch AS s
   ON t.OrderId = s.OrderId
WHEN MATCHED AND s.RowVersion > t.RowVersion THEN UPDATE SET Total = s.Total, Status = s.Status
WHEN NOT MATCHED THEN INSERT (OrderId, CustomerKey, DateKey, Total, Status, RowVersion)
     VALUES (s.OrderId, s.CustomerKey, s.DateKey, s.Total, s.Status, s.RowVersion);
```

The natural key `OrderId` makes the merge repeatable.

### 5. Under the hood
- **Analytics tolerates latency, not inaccuracy.** The book framing: OLAP prioritizes query performance over *immediate transaction consistency*, but analytical accuracy still matters. This allows pre-aggregation, compression, caching and batch loads that OLTP couldn't use.
- **Replica vs warehouse:** a replica gives the same normalized schema — joins remain heavy and history is whatever the source retains. A warehouse reshapes the data (star schema, SCDs), integrates sources and keeps history.
- **The organizational angle:** separating systems also separates teams. OLTP engineers change schemas freely; analytics breaks silently. This is why data contracts and schema-change processes matter — the main data-quality failures come from the silo between the two sides.
- **Failure modes introduced:** pipeline lag, partial loads, duplicate loads, schema drift, deletes that don't propagate, reconciliation mismatches. Each needs a guard (idempotent loads, row-count/checksum reconciliation, freshness alerts, contract tests).

### 6. Performance and design implications
- Read replicas: replication lag is the number to watch; long-running reporting queries on a replica can delay log apply and cause bloat/conflicts on some engines.
- Warehouse: cost is compute + storage; schedule heavy transforms; use incremental models; partition facts.
- Freshness SLA: define per dataset (e.g., "orders ≤ 15 min", "finance ≤ T+1 08:00") and monitor against it with dependency-aware alerts.
- Network/extraction cost on the OLTP side: incremental extraction with an indexed watermark column is cheap; full-table scans nightly are not.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| Always On readable secondary | Read-only replica with near-real-time data | Offload reporting, same schema |
| Transactional replication | Pushes changes to a subscriber DB | Reporting DB with its own indexes |
| CDC / Change Tracking | Captures row changes | Incremental extraction |
| `NONCLUSTERED COLUMNSTORE INDEX` | Columnar copy over OLTP table | Light operational analytics |
| `MERGE` / partition switch | Idempotent upsert / atomic load | Warehouse load |
| Freshness checks (max load time) | Detect stale data | Pipeline monitoring |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Replica: need near-real-time reads, same schema, small team | Reports need history, multi-source joins, or heavy modeling |
| Warehouse + ELT: many sources, large history, BI at scale | Data tiny; pipeline overhead exceeds benefit |
| Columnstore on OLTP table: moderate analytics, no budget for new infra | Write-heavy tables with strict latency; analytics will grow |
| Streaming CDC: freshness in seconds–minutes needed | A nightly batch satisfies the business |

### 9. Common mistakes
- Pointing BI tools at the production primary → outage risk → replica or warehouse.
- Building a pipeline with no freshness monitoring → stale dashboards unnoticed → SLA alerts.
- Assuming replica = warehouse → still slow, still no history → model for analytics.
- Extracting by `updated_at` when some writes don't touch it → missed changes → CDC/rowversion.
- Ignoring deletes → ghost rows in analytics → tombstones/CDC deletes.

### 10. 🎯 Interview kit
**30-second answer:** Analytic scans compete with transactions for CPU, memory, I/O and locks, so we give analytics its own copy — a replica for quick wins, or a warehouse/lakehouse fed by ETL/ELT or CDC for full separation. The trade-off is added latency and a pipeline to operate in return for protection of OLTP and freedom to model data for analysis.

**Structure a longer answer:** clarify freshness/volume → describe ladder (tune → govern → columnstore → replica → warehouse → streaming) → pick based on constraints → name pipeline risks and guards → give a scenario.

**Likely follow-ups:**
- *Replica or warehouse?* → Replica if same schema/near-real-time suffices; warehouse for history, integration, modeling.
- *How do you keep the copy correct?* → Idempotent merges, reconciliation counts, contracts, freshness checks.
- *Blast radius of nightly pull from replicas vs streaming backbone?* → Batch is simple/loosely coupled but stale and spiky; streaming is fresh but couples more components and needs expertise.
- *Alert on freshness without false alarms?* → Compare to expected cadence; use dependency-aware, per-dataset SLAs.

**Signals interviewers look for:** structured options, awareness of latency and failure modes, and organizational (contract) thinking.

### 11. Check yourself
- [ ] Name five ways to separate analytics from OLTP, ordered by disruption.
- [ ] Explain what a replica does *not* give you that a warehouse does.
- [ ] Design an idempotent incremental load for orders.
- [ ] Write a freshness SLA for two datasets with different cadences.
- [ ] List three failure modes introduced by separation and their guards.
<!-- explainer:end -->

## Go deeper

- [Separating OLTP and Reporting](<Separating_OLTP_and_Reporting.md>)
- [Feeding OLAP from OLTP](<Feeding_OLAP_from_OLTP.md>)

## Practice questions

- [Basic] [Explain why a warehouse is often described as 'the single place to answer business questions'.](<../../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34910_Explain_why_a_warehouse_is_often_described_as_the_single_pla.md>)
- [Intermediate] [Explain why you'd choose ELT into a modern cloud warehouse over classic ETL when the OLTP source dumps are large and transformations change frequently.](<../../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-35015_Explain_why_you_d_choose_ELT_into_a_modern_cloud_warehouse_o.md>)
- [Intermediate] [Your OLAP dashboard shows yesterday's data missing until noon — walk through how ETL SLAs, dependency DAGs, and freshness monitoring would catch that earlier.](<../../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34789_Your_OLAP_dashboard_shows_yesterday_s_data_missing_until_noo.md>)
- [Deep] [A dashboard shows a 'live orders today' counter that's expensive to recompute in OLAP each request — design an incremental/cached serving pattern that stays correct across late events.](<../../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-35040_A_dashboard_shows_a_live_orders_today_counter_that_s_expensi.md>)
- [Deep] [Under what conditions would running analytics on a logical replica of the OLTP DB be acceptable, and when does that break down at scale?](<../../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34716_Under_what_conditions_would_running_analytics_on_a_logical_r.md>)
- [Deep] [A candidate says 'we'll just use SNAPSHOT isolation on the OLTP primary for reports so they see consistent data' — as bar-raiser, what deeper trade-offs do you probe about version-store bloat and long-running reads?](<../../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34820_A_candidate_says_we_ll_just_use_SNAPSHOT_isolation_on_the_OL.md>)
- [Expert] [Explain how MVCC bloat in an OLTP Postgres can silently degrade both transactional and any replica-based reporting, and how you'd tune vacuum for a mixed load.](<../../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34739_Explain_how_MVCC_bloat_in_an_OLTP_Postgres_can_silently_degr.md>)
- [Expert] [How do you architect analytics on a heavily-sharded, multi-tenant OLTP system so one tenant's giant OLAP export never starves smaller tenants' transactional path?](<../../../../PRACTICE/02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Q-34835_How_do_you_architect_analytics_on_a_heavily-sharded_multi-te.md>)

## Related concepts

- [Real-time OLAP Serving Layer](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Real-time_OLAP_Serving_Layer.md>) *(Data Architecture)*
- [Choosing a Database for the Workload](<../../../../01_Databases_and_SQL/Database_Strategy/Database_Selection/Choosing_a_Database_for_the_Workload.md>) *(Database Selection)*
- [Rollups vs On-demand Queries](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Rollups_vs_On-demand_Queries.md>) *(Data Architecture)*
- [Warehouse as System of Record](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Warehouse_as_System_of_Record.md>) *(Data Architecture)*
- [Quality Attribute Trade-off Scenarios](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Quality_Attribute_Trade-off_Scenarios.md>) *(Trade off Analysis)*
- [OLTP vs Star Schema Performance](<../../Dimensional_Modeling/OLTP_vs_Star_Schema_Performance.md>) *(Dimensional Modeling)*

## References

- *Azure Data Fundamentals* — Michael John Pena — Understanding Data Workloads › Note
- *Azure Data Fundamentals* — Michael John Pena — Understanding Data Workloads › The OLAP foundation
- *Azure Data Fundamentals* — Michael John Pena — Understanding Data Workloads › Query performance over transaction consistency
- *Azure Data Fundamentals* — Michael John Pena — Understanding Data Workloads › Historical data focus
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — transaCtional vs. analytiC Workloads RANSA TIONAL VS. NALYTI
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — NoN-Clustered ColumNstore INdexes oN rowstore tables - LUSTERED COLUM STORE! DEXESO OWSTORE › Separate OLAP and OLTP Processes
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Ordered Columnstore Indexes › Are Real-Time Operational Analytics Required?
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Data Quality Isn't About Pristine Data › A Brief Summary of OLTP and OLAP
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Data Quality Isn't About Pristine Data › Translation Issues Between OLTP and OLAP Data Worldviews
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Implementing Data Contracts › Mock Data Contract Scenario Intro
- *Data Contracts: Developing Production-Grade Pipelines at Scale* — Chad Sanderson, Mark Freeman — Implementing Data Contracts › Data Contract Violation Scenario
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — The Big Picture Designing and Implementing a Lakehouse Platform › Choosing the Right Architecture
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Chapter 9. Lakehouse of the Future › HTAP
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Use Cases of Azure SQL Database
- *Data Engineering Best Practices* — Richard J. Schiller, David Larochelle — Data factory core › OLTP/OLAP data (Gold zone)

