---
id: "01_Databases_and_SQL/Database_Operations/Capacity_Planning/Capacity_Planning_and_Throughput"
level: 5
type: "concept"
topic: "01_Databases_and_SQL/Database_Operations/Capacity_Planning"
tools: ["AWS"]
---

# Capacity Planning and Throughput

**L5 Expert** · [↑ Capacity Planning](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Capacity planning is turning a business growth forecast into the four resource ceilings that actually bind a database (storage, memory/working set, IOPS/throughput, connections), adding headroom, and proving the numbers with a representative load test before the ceiling is reached.

### 1. The big picture
A database does not fail gracefully when it runs out of a resource. Disk fills and writes stop; the working set stops fitting in RAM and latency jumps by 10–100× (the "I/O cliff"); a connection limit is hit and new sessions are refused. Capacity planning is the discipline of knowing *which* of those walls you will hit first, *when*, and what it costs to move the wall.

It sits between monitoring (what is used now) and cost optimization (what you pay for). Monitoring gives the baseline; the capacity plan projects it forward using leading indicators (orders per day, active users, events per second) rather than lagging ones (CPU %). The books are blunt about its accuracy: it is "always a finger in the air", especially for new systems — but a best guess with explicit assumptions is far better than none, because it tells you what to measure and when to re-plan.

Two cloud-era corrections matter. First, "elastic" does not mean infinite: scaling a single instance up usually means a restart, so for a 24/7 workload a capacity bump is an outage or a failover, and regions can run out of a given instance type. Second, managed services move the question from "how many servers" to "how many provisioned units / what cost envelope", but the same four dimensions still decide it.

### 2. Core concepts and vocabulary
- **Throughput** — work completed per unit time (transactions/s, rows/s, MB/s). **Latency** is time per unit of work; the two trade off as utilization rises.
- **Utilization and headroom** — utilization is used ÷ available; headroom is the deliberate slack kept for spikes, growth between reviews, and failover (a surviving node must carry the failed node's load). A common rule of thumb is to plan for ≤ 60–70 % of a hard limit at peak and add ~20 % on top of storage estimates.
- **Working set** — the portion of data and indexes touched within the time window that matters; it is what must fit in memory, not total data size.
- **Leading indicator** — a business or workload metric that predicts resource demand before it shows up as load (signups, catalog size, requests/s).
- **Soft limit vs hard limit** — a soft limit is a tunable (`max_connections`, provisioned IOPS); a hard limit is architectural (maximum table or file size, partition count, log-sequence-number space, shard count that cannot be changed). Plans must list both.
- **Provisioned vs on-demand capacity** — fixed, forecast-based units (more control, cheaper when predictable) versus pay-per-use auto-scaling (less management, better for spiky or unknown load). Streaming services such as Kinesis make the same choice with shards.
- **Write amplification** — physical bytes written per logical byte (indexes, logs, page splits, compaction, replication); it makes logical size a poor predictor of I/O and disk.
- **Burst credits** — cloud storage/compute that allows short periods above baseline; the baseline, not the burst, is what you can rely on.
- **Little's Law** — concurrency = arrival rate × time in system. It links connections to throughput and latency.

### 3. How it works, step by step
1. **Define the service levels.** Target p95/p99 latency, peak throughput, availability. Capacity is always "enough for *this* SLO".
2. **Baseline the present.** Collect per-resource peaks, not averages: data + index + log + tempdb size, buffer/cache hit ratio, read/write IOPS and MB/s at peak, CPU at peak, concurrent sessions, replication lag.
3. **Pick leading indicators and relate them to resources.** For example "1 order ≈ 12 rows written ≈ 3 KB ≈ 9 IOPS". Fit this from history or from a load test; separate *organic* growth, *feature-driven* step changes and *campaign/seasonal* spikes, because mixing them makes the trend line meaningless.
4. **Project each dimension separately.**
   - *Storage*: data growth × (1 + index overhead) + log + temp/work space + backups/snapshots + bloat or fragmentation + retention changes (purges and archival bend the curve).
   - *Memory*: growth of the working set, not of total data.
   - *IOPS/throughput*: peak operations × amplification, including maintenance (vacuum, compaction, index rebuilds, backups).
   - *Connections*: services × pool size, plus failover reconnect storms.
5. **Add headroom** for growth until the next review, for failover (N+1), and for maintenance windows.
6. **Find the first binding constraint** and the date it binds; list the lead time to relieve it (restart, resharding, hardware order, quota increase).
7. **Validate with a production-representative test**: realistic data volume so plans and cache behaviour are real, realistic concurrency and skew.
8. **Operationalize**: alerts on *forecast* exhaustion ("disk full in 21 days"), a quarterly re-forecast, and an explicit list of what is automated and what needs a human.

A useful sizing sketch for a SQL Server data warehouse from the books: after loading and "exploding" data to its working form, `tempdb` and the transaction log can each need on the order of 25 % of the data-file size; if data + log + tempdb come to 1 TB, provision about 1.2 TB for unplanned growth.

### 4. Worked example
Platform growing 40 % per year; today 2 TB of data, 6 000 peak IOPS, 800 peak connections, 256 GB RAM with a 180 GB working set.

```sql
-- SQL Server: baseline database size and growth from backup history (monthly)
SELECT  database_name,
        DATEPART(YEAR,  backup_start_date) AS y,
        DATEPART(MONTH, backup_start_date) AS m,
        MAX(backup_size)/1073741824.0      AS size_gb
FROM    msdb.dbo.backupset
WHERE   type = 'D'
GROUP BY database_name, DATEPART(YEAR, backup_start_date), DATEPART(MONTH, backup_start_date)
ORDER BY database_name, y, m;
```

Projection (compound 40 %):

| | Now | +1 yr | +2 yr | +3 yr |
|---|---|---|---|---|
| Data (TB) | 2.0 | 2.8 | 3.9 | 5.5 |
| Log + temp @25 % | 0.5 | 0.7 | 1.0 | 1.4 |
| With 20 % headroom (TB) | 3.0 | 4.2 | 5.9 | 8.3 |
| Working set (GB, 30 % growth assumed) | 180 | 234 | 304 | 395 |
| Peak IOPS | 6 000 | 8 400 | 11 800 | 16 500 |

Reading it: the 256 GB host runs out of room for the working set in under a year (304 GB needed in year 2 even before OS and overhead) — memory is the *first* wall, long before disk. IOPS crosses a typical single-volume ceiling (e.g. 16 000 on common cloud block volumes) around year 3. Action: schedule a memory upgrade (needs restart → plan within a maintenance window or via failover to a larger replica) within ~9 months, and start striping/sharding design for IOPS in year 2. Alerts: page when buffer-cache hit ratio drops below baseline or page life expectancy falls sharply (the I/O-cliff signal).

### 5. Under the hood
- **Why the memory cliff is abrupt.** Cache hit ratio of 99 % vs 95 % sounds small, but if a miss costs 1 000× a hit (100 µs SSD vs 100 ns RAM), average access time rises roughly 5×. Latency climbs sharply as the working set crosses RAM.
- **Queueing.** Response time ≈ service time / (1 − utilization). At 50 % utilization you wait about 2× service time, at 90 % about 10×, which is why "average CPU 70 %" with peaks to 95 % already means tail latency pain.
- **Logical vs physical size.** MVCC systems keep old row versions until vacuum/purge; LSM stores hold multiple versions until compaction (space amplification often 1.1–2×); B-tree indexes carry fill-factor slack and fragmentation. Replication adds a full copy per replica and a log stream.
- **Maintenance as load.** Autovacuum/compaction must keep pace with the write rate; if it cannot, bloat grows and the plan's headroom silently disappears.
- **Connections are not free.** Each connection costs memory (and a process or thread); throughput peaks at a modest concurrency near the core count and then falls. Pool at the app tier or with a proxy; `max_connections` is rarely the right lever.
- **Failover capacity.** Replicas that serve reads must also absorb the primary's writes after promotion, and a cold cache after failover multiplies I/O.
- **Streaming capacity** follows the same logic with shards/partitions: throughput is per-shard, hot partition keys make one shard the ceiling, and consumer lag is the leading indicator.

### 6. Performance and design implications
- Plan on **p95/p99 peak**, not average. Averages hide the hour that breaks you.
- Measure the **real** bottleneck: wait statistics (SQL Server), `pg_stat_io`/wait events (PostgreSQL), disk queue length and latency, cache hit ratio, replication lag.
- Connection storms after a cache flush or dependency recovery are a capacity event: model them (all clients reconnecting at once) and cap with pooling, backoff and jitter.
- Separate read and write provisioning where the workloads differ; do not size a write-heavy primary for read replicas' needs.
- Multi-tenant systems need per-tenant limits; one noisy tenant consumes shared IOPS and cache.
- Telemetry itself can rival the primary workload on a large fleet; budget for it.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sp_spaceused`, `sys.dm_db_file_space_usage` | Current data/log/tempdb usage (SQL Server) | Baseline storage |
| `msdb.dbo.backupset` | Historical backup sizes → growth curve | Cheap growth trend without extra tooling |
| `sys.dm_io_virtual_file_stats` | Per-file I/O and stall times | Is storage the bottleneck? |
| `sys.dm_os_performance_counters` (Page life expectancy, Buffer cache hit ratio) | Memory pressure signals | Detect working set spill |
| `pg_total_relation_size()`, `pg_stat_database` | Table+index size, activity (PostgreSQL) | Storage and growth |
| `max_connections`, pool size, `max_server_memory` | Soft limits | Tune after sizing; never as the plan |
| Provisioned IOPS/throughput, burst balance | Cloud storage performance | Right-size vs over-provision |
| Load-test tools (`pgbench`, HammerDB, replay of captured workload) | Validate the model | Before and after each major change |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| **Provisioned capacity** — steady, forecastable load; want cost control | Traffic is spiky or unknown; under-provisioning means throttling |
| **On-demand / serverless** — unpredictable load, small team | Steady high load (usually pricier); cost envelope needs a hard cap and alarms |
| **Scale up** — simplest, no data movement | Restart needed; ceiling of largest instance; blast radius |
| **Scale out (shard/replicas)** — near-linear growth | Application and operational complexity; hot keys |
| **Autoscaling** for stateless/read tiers | Stateful primaries: scaling events cause failovers; keep humans in the loop for writer scaling and storage shrink |
| **Over-provision early** — lead time is long or the outage is costly | Cost; masks inefficiency that should be tuned |

### 9. Common mistakes
- Forecasting from average load → peak breaks the system → plan from p95/p99 peaks and the busiest week.
- Planning storage from logical data size → bloat, indexes, logs and snapshots surprise you → model physical footprint and amplification.
- Ignoring `tempdb`/log/work space → warehouse loads fail mid-run → reserve ~25 % each and test a full load.
- Treating the cloud as infinite → no quota/capacity in the region on the day → request quota early, know your scale-up downtime.
- One-time plan → drifts within a quarter → re-forecast on a schedule and alert on forecast exhaustion.
- Conflating organic growth with one-time events → trend over-extrapolates a launch spike → decompose before fitting.
- Not testing → the model is fiction → load test with realistic data size and skew.

### 10. 🎯 Interview kit
**30-second answer:** Start from the SLO, baseline peak usage of storage, working-set memory, IOPS/throughput and connections, project each with leading business indicators, add headroom including failover, find the first constraint and its lead time, validate with a representative load test, and alert on forecast exhaustion rather than current usage.

**Structure a longer answer:** clarify SLO and growth drivers → four-dimension model with headroom → soft vs hard limits and lead times → validation and automation (what is autoscaled, what stays human) → close with a scenario such as "memory binds before disk, so we upgrade the replica and fail over in a window".

**Likely follow-ups:**
- *How do you capacity-plan a serverless database?* → plan a cost envelope: model request/compute-unit consumption per business unit, set min/max capacity and budget alarms, watch for throttling.
- *How do you account for connection storms?* → Little's Law plus reconnect behaviour; pooling, jittered backoff, admission control.
- *Why is logical size a poor predictor?* → write amplification, MVCC bloat, index growth and fragmentation, replication copies.
- *Where not to autoscale?* → stateful primaries and storage that cannot shrink; scaling triggers failover or restart.
- *Soft vs hard limits?* → tunables versus architectural ceilings; hard limits need a design change and so long lead times.

**Signals interviewers look for:** thinks in dimensions and first-binding-constraint, uses leading indicators, plans for failover and cold cache, knows cloud limits, validates with tests, distinguishes autoscaling from planning.

### 11. Check yourself
- [ ] List the four resource dimensions and give one leading indicator for each.
- [ ] Explain why working-set size, not data size, decides memory needs and describe the I/O cliff.
- [ ] Build a three-year projection at 40 % growth with 20 % headroom and name the first constraint.
- [ ] Give two soft limits and two hard limits for your favourite engine.
- [ ] Describe how a cache flush can turn into a connection storm and how to size for it.
- [ ] State which parts of a database capacity plan you would never fully automate and why.
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)

## Practice questions

- [Intermediate] [What's your framework for capacity planning across dev/staging/prod so pre-prod is representative enough to catch capacity problems yet not cripplingly expensive?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Capacity_Planning/Q-19202_What_s_your_framework_for_capacity_planning_across_dev_stagi.md>)
- [Deep] [How would you capacity-plan a serverless/autoscaling database (Aurora Serverless v2, DynamoDB on-demand) where you're planning cost envelopes rather than fixed instances?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Capacity_Planning/Q-19169_How_would_you_capacity-plan_a_serverless_autoscaling_databas.md>)
- [Deep] [Describe how you'd forecast and provision Kafka/queue capacity (partitions, retention, consumer lag) as a first-class part of database capacity planning.](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Capacity_Planning/Q-19186_Describe_how_you_d_forecast_and_provision_Kafka_queue_capaci.md>)
- [Deep] [How do you model the capacity headroom for connection storms and thundering herds after a dependency recovers or a cache flushes?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Capacity_Planning/Q-19206_How_do_you_model_the_capacity_headroom_for_connection_storms.md>)
- [Deep] [How do you plan capacity for a document store (MongoDB/Couchbase) where document size growth, working-set fit, and oplog window each constrain scale differently?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Capacity_Planning/Q-19232_How_do_you_plan_capacity_for_a_document_store_MongoDB_Couchb.md>)
- [Expert] [Walk me through how you build a multi-year database capacity model for a platform growing 40% YoY across compute, storage, IOPS, and connections — what leading indicators do you anchor it to?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Capacity_Planning/Q-19155_Walk_me_through_how_you_build_a_multi-year_database_capacity.md>)
- [Expert] [How do you build a capacity plan that a SRE team can operationalize as autoscaling policies, and where do you deliberately keep humans in the loop instead of autoscaling a database?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Capacity_Planning/Q-19181_How_do_you_build_a_capacity_plan_that_a_SRE_team_can_operati.md>)
- [Expert] [How do you build a capacity model that distinguishes soft limits (tunable) from hard limits (architectural ceilings like max table size, LSN wraparound, partition count)?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Capacity_Planning/Q-19218_How_do_you_build_a_capacity_model_that_distinguishes_soft_li.md>)

## Related concepts

- [Database Cost Optimization](<../../Database_Strategy/Cost_Optimization/Database_Cost_Optimization.md>) *(Cost Optimization)*
- [Database Vendor and Platform Choices](<../../Database_Strategy/Vendor_and_Tooling/Database_Vendor_and_Platform_Choices.md>) *(Vendor and Tooling)*
- [Org-wide Backpressure](<../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Rate_Limiter_Storage/Org-wide_Backpressure.md>) *(System Design Core)*
- [Choosing a Database for the Workload](<../../Database_Strategy/Database_Selection/Choosing_a_Database_for_the_Workload.md>) *(Database Selection)*
- [Separating Compute and Storage](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Separating_Compute_and_Storage.md>) *(Cloud Fundamentals)*

## References

- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Best Practices for Choosing Data Stream Capacity Mode
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Capacity Reservations
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Google DiRT Disaster Recovery Testing › Capacity management
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Instance and database management › 9.1 #43 Autoshrinking databases
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Instance and database management › 9.6 #48 Failing to capacity plan
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Operational Visibility › Utilization
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Infrastructure Engineering › User resource limits
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Infrastructure Engineering › Storage capacity
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Infrastructure Engineering › Storage availability
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — The Big Picture Designing and Implementing a Lakehouse Platform › Design Questionnaire
- *Designing Data-Intensive Applications* — Martin Kleppmann — Reliable, Scalable, and Maintainable Applications › Scalability
- *Designing Data-Intensive Applications* — Martin Kleppmann — Reliable, Scalable, and Maintainable Applications › Describing Performance
- *Designing Data-Intensive Applications* — Martin Kleppmann — Reliable, Scalable, and Maintainable Applications › Approaches for Coping with Load

