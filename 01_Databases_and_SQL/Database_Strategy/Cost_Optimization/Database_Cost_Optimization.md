---
id: "01_Databases_and_SQL/Database_Strategy/Cost_Optimization/Database_Cost_Optimization"
level: 5
type: "concept"
topic: "01_Databases_and_SQL/Database_Strategy/Cost_Optimization"
tools: ["AWS"]
---

# Database Cost Optimization

**L5 Expert** · [↑ Cost Optimization](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Database cost is the product of compute, storage, data movement and licensing — you cut it by first making the workload cheaper to run (queries, indexes, data layout), then right-sizing and tiering, and only then buying commitments, while governing sprawl so savings stay saved.

### 1. The big picture

Cloud and on-premises databases both burn money, but cloud makes the burn *visible, granular and easy to inflate*: a click can provision a 96-vCPU instance, and a forgotten staging cluster bills every hour. A common root cause is the **lift-and-shift mindset**: teams copy on-premises specifications (sized for the worst day of the year) straight into cloud resources and pay for idle headroom permanently.

Cost optimization is not a one-off cut. It is a loop: *measure → attribute → fix the workload → right-size → tier → commit → govern*. The order matters. Buying a reserved-capacity discount on a badly tuned, oversized instance locks in the waste; tuning first shrinks the thing you then commit to.

Within *Database Strategy*, this lesson pairs with cloud architecture (what you are running), selection (which engine the workload really needs) and vendor choice (which pricing model you accept).

### 2. Core concepts and vocabulary

- **Compute cost** — CPU time, memory and cluster nodes. Inefficient queries, bad joins and over-provisioned nodes raise it.
- **Storage cost** — primary data, indexes, replicas, snapshots and backups. Grows with redundant copies, long retention and no compression or tiering.
- **Data transfer / egress** — charges for moving data between regions, clouds or out to clients and BI tools; easy to overlook until multi-region or high-frequency API access appears.
- **Licensing / managed-service fees** — per-core licenses, reserved capacity, per-query pricing or tiered service levels.
- **Right-sizing** — matching instance size and tier to measured demand, not guessed peak.
- **Reserved instances / savings plans / committed-use** — a discount in exchange for a usage commitment over a term; **on-demand** is the flexible, full-price option.
- **Serverless / consumption pricing** — pay per unit of use (per second of capacity, per request); cheap when idle, potentially expensive and opaque when steady.
- **Tiered storage** — hot (fast, expensive) vs warm vs cold/archive storage.
- **Database sprawl / zombie databases** — forgotten replicas, staging clusters and snapshots that still bill.
- **FinOps tagging / cost attribution** — labeling resources to teams so spend has an owner.
- **Noisy neighbor** — one workload consuming shared capacity and forcing everyone to over-provision.

### 3. How it works, step by step

A defensible plan to cut spend (say 30% in two quarters) without an SLA regression:

1. **Baseline and attribute.** Break the bill down by service, environment, team and database. Tag everything; untagged spend is the first finding.
2. **Remove pure waste (lowest risk).** Delete orphaned snapshots, abandoned replicas, idle non-production clusters; schedule dev/test to stop out of hours; shorten retention that no one needs.
3. **Fix the workload.** Profile the top consumers: long-running queries, missing or unused indexes, full scans, excessive shuffles, `SELECT *` and N+1 patterns. Use partition pruning, caching or materialized views where results are reused. This reduces CPU and IO and enables a smaller instance.
4. **Right-size.** Use utilization over a long window, including the peak batch window. Move to a smaller class or lower tier; use burstable or serverless options for variable workloads.
5. **Tier and compress data.** Move cold data to cheaper storage or object storage; compress where CPU headroom exists; archive history out of the hot database.
6. **Commit what is stable.** Apply reserved capacity or savings plans only to the steady baseline, leaving spiky demand on-demand or serverless.
7. **Govern.** Add guardrails — cost review for large instance classes, budgets and alerts, mandatory tags, a decommissioning workflow — so engineers can still move fast.
8. **Review monthly.** Re-baseline; cost drifts back up without owners.

Levers to *deliberately avoid* when SLAs are fixed: over-aggressive right-sizing based on averages, removing replicas that provide HA, cutting backup retention below compliance minimums, and consolidating instances without considering blast radius.

### 4. Worked example

Goal: find indexes that cost write and storage overhead but are never used (SQL Server; the same idea exists in every engine).

```sql
-- Indexes with writes but no reads since the last instance restart
SELECT  OBJECT_NAME(i.object_id)  AS table_name,
        i.name                    AS index_name,
        s.user_updates,
        s.user_seeks + s.user_scans + s.user_lookups AS user_reads
FROM    sys.indexes i
JOIN    sys.dm_db_index_usage_stats s
        ON s.object_id = i.object_id AND s.index_id = i.index_id
       AND s.database_id = DB_ID()
WHERE   i.is_primary_key = 0 AND i.is_unique_constraint = 0
  AND   s.user_seeks + s.user_scans + s.user_lookups = 0
ORDER BY s.user_updates DESC;
```

How to read it: high `user_updates` with zero reads means every insert/update pays to maintain the index for no benefit — extra CPU, log and storage. Caveats: counters reset on restart, so confirm the observation window covers month-end or quarterly jobs; unique and constraint-backing indexes are excluded on purpose. Disable first, monitor, then drop.

Second example, a quick cost attribution check of the biggest tables:

```sql
SELECT TOP 20 t.name, SUM(ps.used_page_count) * 8 / 1024 AS used_mb
FROM sys.dm_db_partition_stats ps
JOIN sys.tables t ON t.object_id = ps.object_id
GROUP BY t.name
ORDER BY used_mb DESC;
```

Large tables that are mostly historical are candidates for partitioning with archival or compression.

### 5. Under the hood

- **Why query tuning saves money.** Cloud billing tracks the resources a query consumes (CPU seconds, IO, scanned bytes, RUs). A scan that reads a whole table instead of a partition multiplies cost linearly with table size; pruning and indexes convert that to near-constant cost.
- **Maintenance still matters.** Up-to-date statistics keep plans efficient (the optimizer chooses cheaper plans with accurate cardinality estimates). Run statistics updates *before* index maintenance, because an index rebuild already refreshes its statistics; avoid auto-close and auto-shrink settings, which cost performance.
- **Compression trade-off.** Row/page/columnar compression reduces storage and IO but spends CPU on compress/decompress and can change plan choices; decide codec per table based on read/write mix and available CPU headroom.
- **Compute/storage decoupling.** It lets you scale each independently, but moves spend into a per-second consumption line that can be harder to govern; it lowers TCO mainly for variable or bursty demand and rarely for steady 24×7 load.
- **Lakehouse cost traps.** Small files inflate metadata and per-request costs; without compaction, scans slow and bills grow; metadata scaling and storage-versioning (time travel retention) add hidden storage.
- **Egress.** Charged per byte leaving a network boundary. Multi-cloud designs pay it on every cross-cloud read, which is why placement and caching decisions dominate multi-cloud TCO.
- **Provisioned IOPS vs general-purpose volumes.** IOPS, throughput and instance class are coupled; moving to a cheaper volume type only saves money if latency SLOs still hold.

### 6. Performance and design implications

- Right-sizing from *peak* alone overspends; from *average* alone risks outages. Use percentile utilization plus known batch windows, and test autoscaling latency.
- Overly sensitive autoscaling thresholds cause constant scaling churn and cost; add cooldowns.
- Over-indexing is a cost problem as real as under-indexing: each index adds write amplification and storage.
- Denormalization reduces join compute but can raise storage and update cost at large scale; evaluate total cost, not just query time.
- Hot-for-compliance data: keep it queryable but cheap — tiered storage, caching, and archival with documented retrieval time.
- Consolidation onto fewer or larger instances saves per-instance overhead but widens the blast radius and may re-create noisy-neighbor contention that forces over-provisioning; use resource governance and workload isolation.
- Metrics to look for: CPU and memory over time, IO wait, cost per query/tenant/transaction, idle hours, storage growth rate, unused index count, orphan resource count.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `sys.dm_db_index_usage_stats` | Index reads vs writes | Finding unused indexes |
| Query Store / `pg_stat_statements` | Top queries by resources | Finding the workload to fix |
| `EXPLAIN` / actual plans | Shows scans, joins, shuffles | Validating a tuning change |
| Partitioning + pruning | Limits data scanned | Large time-based tables |
| Data compression (row/page/columnstore) | Smaller storage and IO | Cold or read-heavy tables |
| Auto-pause / serverless tier | Scale to zero when idle | Dev/test, intermittent workloads |
| Reserved capacity / savings plan | Discount for commitment | Stable baseline only |
| Budgets, cost alerts, tags | Visibility and ownership | Always |
| Storage lifecycle policies | Move data to cold tiers | Backups, logs, history |
| Lakehouse compaction / `OPTIMIZE` | Merge small files | Lake/warehouse tables |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| **Reserved / committed capacity** — baseline is steady and multi-year | You may migrate engines or clouds; commitments become lock-in |
| **On-demand** — new or uncertain workloads | Long-running steady load (you overpay) |
| **Serverless / scale-to-zero** — spiky or near-idle databases | Latency-sensitive wake-ups, steady high load, or runaway query bills |
| **Consolidate instances** — many small, low-use databases | Tenants with conflicting SLAs; blast radius matters |
| **Aggressive compression** — CPU headroom, IO- or storage-bound | CPU-bound OLTP with heavy updates |
| **Cold-tier archive** — rarely accessed, tolerant of retrieval delay | Compliance queries needing instant access |
| **Buy observability/cost tooling** — small org, fast need | Large estate where custom attribution pays back |

### 9. Common mistakes

- Lift-and-shift sizing → permanent overprovisioning → right-size from measured utilization.
- Committing before tuning → lock-in of waste → tune and right-size, then commit the baseline.
- Judging over-provisioning by CPU average → outage during the rare batch window → include peak windows and IO/wait metrics.
- Ignoring autoscaling latency → timeouts during bursts → keep minimum capacity or pre-scale.
- Dropping indexes after a short observation → breaks quarter-end jobs → observe a full business cycle, disable before drop.
- Overlooking egress and cross-AZ transfer → surprise bills → model data movement in architecture reviews.
- No decommissioning discipline → zombie databases consume a meaningful share of budget → tags, owner, expiry dates and a retire workflow.
- Cutting backup/retention below compliance floors → regulatory exposure → treat compliance as a hard constraint.
- Trusting consumption pricing without a cap → unbounded cost → budgets, quotas and alerts.

### 10. 🎯 Interview kit

**30-second answer:** "I treat cost as compute plus storage plus data movement plus licensing. I attribute spend first, remove waste, tune the workload so it needs less hardware, right-size using percentile and batch-window data, tier cold data, and only then commit reserved capacity on the steady baseline — with tagging, budgets and a decommissioning process so it stays fixed."

**Structure a longer answer:** clarify SLA and growth → baseline and attribute → ordered levers by risk → trade-offs (commitment vs flexibility, consolidation vs blast radius) → close with an example such as a 30% reduction plan.

**Likely follow-ups:**
- *Reserved vs on-demand across a fluctuating fleet?* → commit only the floor of usage; keep the spiky remainder on-demand or serverless; review quarterly to avoid over-commitment.
- *How do you know a database is truly over-provisioned?* → long-window percentiles for CPU, memory, IO and wait, plus known peak events; idle-looking but needed headroom shows periodic spikes.
- *When does serverless cost more?* → steady high utilization, chatty connections that prevent pausing, or unbounded scans billed per use.
- *How do you stop zombie databases?* → mandatory owner/expiry tags, automated idle detection, approval-based retirement with a final backup.
- *Lakehouse cost traps?* → small files, missing compaction, metadata growth, long time-travel retention.

**Signals interviewers look for:** ordered levers, explicit risk trade-offs, SLA awareness and governance, not just "turn things off".

### 11. Check yourself

- [ ] List the four cost components and one lever for each.
- [ ] Write the query that finds unused indexes and state two caveats.
- [ ] Sequence a 30% cut plan and name two levers you would avoid.
- [ ] Explain why committing before tuning is a mistake.
- [ ] Decide serverless vs provisioned for a 40%-idle, latency-sensitive database.
- [ ] Describe a decommissioning workflow that does not break hidden dependencies.
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Deep] [How do you decide when the storage-vs-compute decoupling of Snowflake or Aurora Serverless v2 genuinely lowers TCO versus when it just shifts spend into an opaque per-second line item you can't govern?](<../../../PRACTICE/01_Databases_and_SQL/Database_Strategy/Cost_Optimization/Q-19253_How_do_you_decide_when_the_storage-vs-compute_decoupling_of_.md>)
- [Deep] [Why is data egress often the silent killer of a multi-cloud database strategy, and how do you architect around it without becoming a single-vendor hostage?](<../../../PRACTICE/01_Databases_and_SQL/Database_Strategy/Cost_Optimization/Q-19272_Why_is_data_egress_often_the_silent_killer_of_a_multi-cloud_.md>)
- [Deep] [What metrics would convince you a database is genuinely over-provisioned versus one that just looks idle but needs the headroom for a rare critical batch window?](<../../../PRACTICE/01_Databases_and_SQL/Database_Strategy/Cost_Optimization/Q-19294_What_metrics_would_convince_you_a_database_is_genuinely_over.md>)
- [Deep] [How do you handle cost optimization for a database that must stay hot for compliance/regulatory query access even though it's rarely touched?](<../../../PRACTICE/01_Databases_and_SQL/Database_Strategy/Cost_Optimization/Q-19317_How_do_you_handle_cost_optimization_for_a_database_that_must.md>)
- [Deep] [What's your take on 'database sprawl' as a cost problem — how do you build a decommissioning discipline that retires datastores as aggressively as teams stand them up?](<../../../PRACTICE/01_Databases_and_SQL/Database_Strategy/Cost_Optimization/Q-19338_What_s_your_take_on_database_sprawl_as_a_cost_problem_how_do.md>)
- [Expert] [You're asked to cut database spend 30% in two quarters without an SLA regression. How do you sequence the levers, and which do you deliberately avoid because they trade cost for risk?](<../../../PRACTICE/01_Databases_and_SQL/Database_Strategy/Cost_Optimization/Q-19266_You_re_asked_to_cut_database_spend_30_in_two_quarters_withou.md>)
- [Expert] [When a data lakehouse promises to unify warehouse + lake cost, what are the concrete cost traps (small-file problem, compaction, metadata scaling) you'd interrogate?](<../../../PRACTICE/01_Databases_and_SQL/Database_Strategy/Cost_Optimization/Q-19292_When_a_data_lakehouse_promises_to_unify_warehouse_lake_cost_.md>)
- [Expert] [When you consolidate onto a single warehouse to get volume discounts, how do you prevent the resulting noisy-neighbor contention from forcing you to over-provision and erase the savings?](<../../../PRACTICE/01_Databases_and_SQL/Database_Strategy/Cost_Optimization/Q-19318_When_you_consolidate_onto_a_single_warehouse_to_get_volume_d.md>)

## Related concepts

- [Governing Data Platform Costs](<../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Governing_Data_Platform_Costs.md>) *(FinOps)*
- [Commitments and Spot Capacity](<../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Commitments_and_Spot_Capacity.md>) *(FinOps)*
- [Database Vendor and Platform Choices](<../Vendor_and_Tooling/Database_Vendor_and_Platform_Choices.md>) *(Vendor and Tooling)*
- [Small Files and Compaction](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Small_Files_and_Compaction.md>) *(Data Architecture)*
- [Rollups vs On-demand Queries](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Rollups_vs_On-demand_Queries.md>) *(Data Architecture)*
- [Choosing a Database for the Workload](<../Database_Selection/Choosing_a_Database_for_the_Workload.md>) *(Database Selection)*

## References

- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Scalability and Performance Optimization › 12. Scalability and Performance Optimization
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Operationalizing Data Pipelines › 14. Operationalizing Data Pipelines
- *Azure Data Fundamentals* — Michael John Pena — Data Storage Fundamentals › Cost efficiency through scale
- *Azure Data Fundamentals* — Michael John Pena — Data Storage Fundamentals › Considerations for Cloud Storage
- *Azure Data Fundamentals* — Michael John Pena — Data Storage Fundamentals › Identifying the Hidden Costs in Cloud Storage: What to Watch For
- *Azure Data Fundamentals* — Michael John Pena — Azure SQL Services and Open Source Options › Simplifying Cost Considerations
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 2 Execution Plan Generation And The Query Optimizer
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 21 Intelligent Query Processing
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — load customer names › Chapter 23 A Query Tuning Methodology
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 9: Execution Plans and Query Store › Structure
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Service Tiers and Compute Models
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Cost Models: DTU vs. vCore
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Elastic Pools and Scaling Options
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Service-Level Management › Cost/efficiency indicators
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Risk Management › Availability of Resources
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Infrastructure Engineering › Benefits of Physical Servers
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Data Storage, Indexing, and Replication › Multileader use cases
- *Data Engineering with Medallion Architecture* — Miki Eto — Scaling Data Platforms with Optimization › Related topics
- *Data Engineering with Medallion Architecture* — Miki Eto — Scaling Data Platforms with Optimization › Key considerations
- *Data Engineering with Medallion Architecture* — Miki Eto — Scaling Data Platforms with Optimization › Process or steps
- *Data Engineering with Medallion Architecture* — Miki Eto — Scaling Data Platforms with Optimization › Cautions and limits

