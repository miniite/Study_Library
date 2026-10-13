---
id: "01_Databases_and_SQL/Database_Strategy/Cloud_Databases/Cloud_Database_Architecture"
level: 5
type: "concept"
topic: "01_Databases_and_SQL/Database_Strategy/Cloud_Databases"
tools: ["AWS", "Azure", "PostgreSQL"]
---

# Cloud Database Architecture

**L5 Expert** · [↑ Cloud Databases](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A cloud database is a managed service whose real design choices are *how much of the stack you hand over* (VM, managed instance, fully managed, serverless) and *how its compute, storage, replication and billing are decoupled* — and every one of those choices changes your failure modes, consistency guarantees and bill.

### 1. The big picture

Running a database in the cloud is not "the same database on someone else's server". Cloud providers split the old monolith (one machine owning CPU, RAM, disk and the replication story) into separately scaled, separately billed layers: compute, storage, networking, backup, and the control plane that patches and fails over. Your job as an architect moves from *operating* the engine to *choosing the shape of the service* and *designing around its limits*: connection ceilings, throttling (HTTP 429 in Cosmos DB), replica lag, cold starts, quotas and egress charges.

The first decision is the **responsibility spectrum**. For SQL Server on Azure this runs from SQL Server on an Azure VM (you manage the OS and the engine; every legacy feature works) through Azure SQL Managed Instance (near-complete on-premises compatibility, Microsoft runs the patching and HA) to Azure SQL Database (fully managed, single database or elastic pool, ideal for SaaS and transactional workloads) and, for analytics, Azure Synapse. The same ladder exists on AWS (EC2-hosted engine → RDS → Aurora → serverless variants). The rule of thumb: pick the highest rung that still supports the features you need, because every rung you climb removes an operational chore and adds a constraint.

This lesson sits under *Database Strategy*; its siblings cover cost, workload-driven selection and vendor choice. Here we focus on the *architecture* of cloud databases: decoupled storage, replicas, multi-region, connection management and failure testing.

### 2. Core concepts and vocabulary

- **Managed service** — the provider handles provisioning, patching, backups and failover; you keep schema, queries, access control and capacity decisions.
- **IaaS vs PaaS database** — IaaS = a VM you run the engine on (full control, full chore list); PaaS = the engine is a service (less control, less toil).
- **Compute/storage decoupling** — the storage layer is a distributed, replicated log/page service separate from the instances that run queries. Aurora is the canonical example: replicas read the same shared storage volume rather than replaying their own copy.
- **Writer / reader endpoints** — a cluster exposes one write target and one or more read targets; the application (or a proxy) chooses.
- **Read replica** — a copy serving reads; may be asynchronous with some **replica lag**.
- **Multi-AZ** — synchronous or storage-level redundancy across availability zones in one region; protects against a zone or host failure; failover typically within minutes or less.
- **Multi-region / global database** — cross-region replication for disaster recovery (DR) and low-latency reads; protects against a regional outage but has larger RPO/RTO trade-offs.
- **RPO / RTO** — recovery point objective (how much data you may lose) and recovery time objective (how long you may be down).
- **Throttling / request units (RU)** — in Cosmos DB, every operation costs RUs; exceed the provisioned budget and you get 429 responses.
- **Connection ceiling** — each engine/instance size supports a maximum number of concurrent connections; serverless fleets can hit it in seconds.
- **Serverless database** — capacity scales automatically (e.g., Aurora Serverless v2 scales in ACUs, Aurora Capacity Units) and may pause or scale to a minimum.
- **Egress** — charges for data leaving a region or cloud.

### 3. How it works, step by step

**Choosing the rung (target platform):**

1. List the features you cannot give up (cross-database queries, SQL Agent, CLR, linked servers, OS-level access).
2. Map to the rung: needs OS/legacy feature → VM; needs near-full compatibility but not OS → Managed Instance; greenfield or SaaS multi-tenant → single database/elastic pool; large-scale reporting → warehouse/analytics service.
3. Decide who owns maintenance: the more you hand over, the less you can time patches or tweak engine settings.
4. Prototype the riskiest workload on the chosen rung before committing.

**What happens on a write in a decoupled-storage cluster (Aurora-style):**

1. The client sends the write to the writer instance via the cluster endpoint.
2. The writer produces redo log records and ships them to the storage fleet, which spreads them across multiple AZs.
3. The write is acknowledged once a quorum of storage nodes has durably recorded it.
4. Reader instances attach to the same storage and receive cache-invalidation/log information from the writer; they apply it to their buffer pools, usually within milliseconds.
5. If the writer fails, one reader is promoted; because storage is shared, no bulk data copy is needed, so failover is fast.

This is why Aurora "replica lag" is small but never zero: a read right after a write on a replica can still be stale, so read-your-writes needs an explicit design (route that session to the writer, or use a session-consistency feature).

**Read/write split decision:**

1. Measure read:write ratio and whether read queries are heavy analytics or point lookups.
2. If reads only need to scale, add replicas in the same cluster behind the reader endpoint.
3. If analytic scans hurt OLTP latency, offload to a warehouse/query service or a zero-ETL replica rather than more OLTP replicas.
4. Split into separate clusters only when the workloads need different instance classes, parameter groups, maintenance windows or blast-radius isolation — and accept that cross-cluster consistency becomes your problem.

### 4. Worked example

Scenario: a B2B SaaS on a managed PostgreSQL-compatible cluster; a surge of Lambda/serverless invocations opens thousands of connections and exhausts the connection ceiling.

```sql
-- On PostgreSQL-compatible engines: see who holds connections
SELECT state, count(*) AS conns
FROM pg_stat_activity
GROUP BY state
ORDER BY conns DESC;

-- Compare against the ceiling
SHOW max_connections;
```

How to read it: a large `idle` count means connections are held by application instances that are not working — the signature of a missing pool. The fix is layered, not a bigger instance:

1. **Pool in front of the database** (a managed proxy or a pooler such as PgBouncer) so thousands of short-lived clients share tens of server connections.
2. **Queue at the application edge** so surges wait instead of opening more connections (back-pressure).
3. **Load-shed** non-critical traffic first (reports, batch refresh) when saturation metrics cross a threshold.
4. **Circuit-break**: when connection acquisition times out repeatedly, fail fast instead of piling retries on the database.

Tune retries with exponential backoff and jitter; otherwise throttled clients retry in lockstep and create a second surge.

For Cosmos DB the same idea appears as 429 handling: clients must honor the retry-after hint, and a bursty microservice should have its own container or its own RU budget so it cannot starve the shared account. Ownership of that policy belongs with a platform team that sets per-service quotas and alerts.

### 5. Under the hood

- **Shared-storage replicas.** Because replicas read the same volume, adding a replica adds compute, not a data copy. Storage cost does not multiply; replica lag stays low; promotion is fast. The trade-off: you cannot tune storage per replica, and the writer is still a single point of write throughput.
- **Storage auto-growth is not capacity planning.** Storage grows automatically, but IOPS, throughput, connection counts, instance memory and cost do not. "We never need to worry about capacity" is dangerously incomplete: you still own working-set-to-memory ratio, connection limits and spend growth.
- **Global databases.** Cross-region replication is asynchronous at the storage or log level, so the secondary's RPO is typically seconds, not zero; a regional failover is a deliberate, sometimes manual, promotion with RTO of minutes. Multi-region *writes* (Cosmos multi-region writes, DynamoDB Global Tables) instead accept concurrent writes in several regions and need a conflict-resolution policy — typically last-writer-wins by timestamp or a custom merge — meaning some writes silently lose.
- **Consistency menus.** Cosmos DB offers a spectrum from strong through bounded staleness, session and consistent-prefix to eventual; weaker levels cost fewer RUs and less latency. The partition key you choose interacts with this: queries that span partitions pay coordination cost regardless of level.
- **Serverless scaling.** Scaling in ACUs or on-demand capacity is not instantaneous; a pause/resume or a rapid scale-up has latency, and scaling the instance does not raise connection limits as fast as demand rises.
- **Proxy layers.** Connection proxies multiplex many client sessions onto few server sessions. Session-level state (temp tables, prepared statements, `SET` options) can break multiplexing, so "pinning" occurs and reduces the benefit.

### 6. Performance and design implications

- Separate *latency-sensitive OLTP* from *scan-heavy analytics*; replicas share the same engine, so a heavy report on a replica is fine for the writer but can still starve other readers.
- Watch the right signals: replica lag, connection count vs ceiling, buffer cache hit ratio, IOPS and burst balance on smaller classes, 429 rate (Cosmos), and CPU credits on burstable classes.
- Right-size with *percentile* and *burst-aware* metrics, not averages; a database idle 95% of the time may legitimately need headroom for one batch window.
- Test DR for real: backup existence is not restorability. Schedule region-failover game days and restore drills; measure actual RTO/RPO and fix the runbook.
- Plan upgrades (major-version, engine patches) with a blue/green or replica-promotion pattern so cutover is minutes, and validate extensions and drivers first.
- Keep the roadmap conservative: depend on generally available features, treat preview features as experiments, and keep an exit path.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `SHOW max_connections;` / `pg_stat_activity` | Connection ceiling and who holds connections | Diagnosing connection storms |
| `SHOW GLOBAL VARIABLES LIKE 'max_connections';` (MySQL) | Same for MySQL-family engines | Aurora MySQL sizing |
| Reader/writer endpoint | Route reads vs writes | Read scaling, read/write split |
| Managed proxy (e.g., RDS Proxy) or PgBouncer | Pool and multiplex connections | Serverless/Lambda fleets, many app instances |
| Provisioned RU / autoscale max RU (Cosmos) | Throughput budget | Capping a noisy service |
| Retry policy with backoff + jitter | Handles 429s and transient errors | Every cloud DB client |
| Point-in-time restore / snapshot restore | Rewind or clone | DR drills, bad-deploy recovery |
| Global database / replica promotion | Cross-region failover | Region-level DR |
| Performance Insights / Enhanced Monitoring (AWS), Query Performance Insight (Azure) | Wait and load analysis | Capacity and cost decisions |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| **VM-hosted engine** — you need OS access or unsupported legacy features | You do not want to patch, back up and fail over yourself |
| **Managed Instance / managed engine** — lift-and-shift with high compatibility | You need engine settings or extensions the service forbids |
| **Single database / elastic pool** — many small tenant databases with variable load | One tenant can monopolize pool resources (noisy neighbor) |
| **Serverless / scale-to-zero** — spiky or mostly idle databases | Latency-sensitive workloads that cannot tolerate warm-up, or connection-heavy fleets |
| **Same cluster for reads and writes** — simple, cheap, strong read-your-writes via writer | Heavy analytics hurting OLTP; different maintenance or tuning needs |
| **Separate clusters** — isolate blast radius and tune independently | You need consistent cross-cluster reads; ops overhead is real |
| **Global DB with single writer** — DR and local reads, simple semantics | You need low-latency writes in every region |
| **Multi-region writes** — local writes everywhere | You cannot accept conflict-resolution outcomes (e.g., last-writer-wins) |

### 9. Common mistakes

- Treating a managed service as "no ops" → capacity, connection and cost incidents → keep ownership of limits, quotas and budgets.
- Assuming auto-scaling storage means auto-scaling everything → IOPS/connection/memory exhaustion → monitor and set alarms before the ceiling.
- Splitting clusters unnecessarily → more consistency headaches and cost → split only for a stated isolation or tuning reason.
- Ignoring replica lag in read-your-writes paths → users see stale data → route post-write reads to the writer or use session consistency.
- Letting serverless clients connect directly → connection storms → put a proxy/pooler in front and cap concurrency.
- Never testing failover → the runbook fails when needed → run game days and restore drills.
- Retrying without jitter → synchronized retry storms → exponential backoff with jitter and a retry budget.
- Forgetting VPC and networking (subnets, security groups, private endpoints, cross-AZ and cross-region transfer) → surprise latency and egress cost → include networking in the design review.
- Betting a roadmap on preview features → forced rework → require GA status or an exit plan.

### 10. 🎯 Interview kit

**30-second answer:** "I pick the rung of the responsibility ladder that supports my required features with the least operational burden — managed engine over VM unless I need OS or legacy features. Then I design around the service's limits: decoupled storage gives fast replicas and failover but not unlimited connections or zero lag, so I add pooling, backpressure and explicit read-your-writes routing, and I test DR rather than assume it."

**Structure a longer answer:** clarify workload, RPO/RTO, consistency and compliance → pick the service rung → describe the mechanism (writer, shared storage, replicas, regions) → state trade-offs (cost, lag, conflicts, lock-in) → close with a real scenario such as a traffic surge or regional failover.

**Likely follow-ups:**
- *Why can you still see stale reads on Aurora despite storage-level replication?* → replicas apply log/invalidations asynchronously; lag is small but non-zero, so read-your-writes needs the writer or a session guarantee.
- *Multi-AZ vs multi-region?* → Multi-AZ handles host/zone failures with near-zero RPO and fast RTO; multi-region handles region loss with seconds of RPO and minutes of RTO and more cost.
- *How do you keep a bursty service from starving a shared Cosmos account?* → per-service containers or RU budgets, 429-aware clients with backoff, and platform-owned quotas.
- *How would you migrate hundreds of self-managed MySQL nodes to Aurora?* → assess and tier by RPO/RTO, pilot, replicate with CDC, cut over in waves with rollback gates, keep the source until validation passes.
- *Which global write option and what conflict model?* → single-writer global DB for simple semantics; multi-region writes only when local write latency is essential and last-writer-wins or a custom merge is acceptable.

**Signals interviewers look for:** you name failure modes (connection ceiling, throttling, lag), tie decisions to RPO/RTO and cost, and mention testing and rollback.

### 11. Check yourself

- [ ] Explain the four rungs of the managed-SQL ladder and name a feature that forces each.
- [ ] Describe how a write is acknowledged in a shared-storage cluster and why replica lag is small but non-zero.
- [ ] Design connection handling for a serverless fleet hitting a managed database (pool, queue, shed, break).
- [ ] Contrast Multi-AZ, a global single-writer database and multi-region writes by RPO, RTO and conflict handling.
- [ ] Draw a phased migration plan with rollback gates for a large self-managed fleet.
- [ ] Explain why "storage auto-scales, so capacity is solved" is wrong.
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)
- [Azure](<../../../TOOLS/Azure/_index.md>)
- [PostgreSQL](<../../../TOOLS/PostgreSQL/_index.md>)

## Practice questions

- [Deep] [How do you evaluate whether to keep write and read workloads on the same Aurora cluster or split them across purpose-tuned clusters, and how does that affect your consistency guarantees?](<../../../PRACTICE/01_Databases_and_SQL/Database_Strategy/Cloud_Databases/Q-07929_How_do_you_evaluate_whether_to_keep_write_and_read_workloads.md>)
- [Deep] [What's your framework for deciding reserved instances versus savings plans versus on-demand across a fluctuating RDS/Aurora fleet, and how do you avoid over-committing the org?](<../../../PRACTICE/01_Databases_and_SQL/Database_Strategy/Cloud_Databases/Q-07955_What_s_your_framework_for_deciding_reserved_instances_versus.md>)
- [Deep] [How do you architect quota, rate-limiting, and throttling policy for Cosmos DB (429 handling) so a bursty microservice can't starve the shared account — and who owns that policy org-wide?](<../../../PRACTICE/01_Databases_and_SQL/Database_Strategy/Cloud_Databases/Q-07982_How_do_you_architect_quota_rate-limiting_and_throttling_poli.md>)
- [Deep] [How do you architect for graceful degradation when Aurora hits its connection ceiling under a traffic surge — queueing, load-shedding, circuit-breaking at the data tier?](<../../../PRACTICE/01_Databases_and_SQL/Database_Strategy/Cloud_Databases/Q-08004_How_do_you_architect_for_graceful_degradation_when_Aurora_hi.md>)
- [Deep] [Walk me through how you would build a multi-year strategy to migrate a 400-node self-managed MySQL fleet onto Aurora while keeping every tier under its RPO/RTO — what phases and rollback gates would you define?](<../../../PRACTICE/01_Databases_and_SQL/Database_Strategy/Cloud_Databases/Q-08027_Walk_me_through_how_you_would_build_a_multi-year_strategy_to.md>)
- [Expert] [What's your strategy for testing DR — do you run real region-failover game days on Aurora Global Database, and how do you convince leadership the disruption is worth it?](<../../../PRACTICE/01_Databases_and_SQL/Database_Strategy/Cloud_Databases/Q-07949_What_s_your_strategy_for_testing_DR_do_you_run_real_region-f.md>)
- [Expert] [How do you build a multi-year data-platform roadmap that anticipates Aurora and Cosmos feature evolution without betting the company on unreleased or preview features?](<../../../PRACTICE/01_Databases_and_SQL/Database_Strategy/Cloud_Databases/Q-07970_How_do_you_build_a_multi-year_data-platform_roadmap_that_ant.md>)
- [Expert] [How would you architect a globally distributed write workload — would you reach for Aurora Global Database, Cosmos multi-region writes, or DynamoDB Global Tables, and what conflict-resolution model do you accept in each?](<../../../PRACTICE/01_Databases_and_SQL/Database_Strategy/Cloud_Databases/Q-07994_How_would_you_architect_a_globally_distributed_write_workloa.md>)

## Related concepts

- [Cross-cloud Data Contracts](<../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Cross-cloud_Data_Contracts.md>) *(Multi Cloud Data)*
- [Choosing a Database for the Workload](<../Database_Selection/Choosing_a_Database_for_the_Workload.md>) *(Database Selection)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Best Practices
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Hybrid Cloud Architectures
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Future Enhancements and Trends
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › Understanding Cloud Platforms
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Prerequisite Knowledge for Aspiring Data Engineers › Conclusion
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Overview of AWS Analytics and Auxiliary Services › Amazon Athena
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Overview of AWS Analytics and Auxiliary Services › Conclusion
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Ingesting Data Using Zero-ETL Integrations › Supported Targets for AWS DMS
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Compute Engines for Lakehouse Architectures › Ecosystem Support
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — Lakehouse in the Real World › Reference Architectures
- *Practical Lakehouse Architecture* — Gaurav Ashok Thalpati — H › I
- *Azure Data Fundamentals* — Michael John Pena — Azure Cosmos DB › Chapter 8. Azure Cosmos DB
- *Azure Data Fundamentals* — Michael John Pena — Azure Cosmos DB › Coverage of Curriculum Objectives
- *Azure Data Fundamentals* — Michael John Pena — Azure Cosmos DB › Exam Tip
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Storage Design Patterns › Examples
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Threading and Concurrency Understanding the 2020 Amazon Kinesis Outage › Scaling Is Not a Magic Bullet

