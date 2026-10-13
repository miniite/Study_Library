---
id: "01_Databases_and_SQL/Database_Operations/High_Availability/HA_Failover_Design"
level: 5
type: "concept"
topic: "01_Databases_and_SQL/Database_Operations/High_Availability"
tools: []
---

# HA Failover Design

**L5 Expert** · [↑ High Availability](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** HA failover design is choosing how much data loss (RPO) and downtime (RTO) you accept, then building replication, failure detection, leader election with fencing, and client rerouting so a node failure becomes a short, safe, automatic role change instead of an outage or a split-brain.

### 1. The big picture
High availability (HA) keeps the service running through the failure of a component — a node, a disk, an availability zone. Disaster recovery (DR) restores service after a larger loss — a region, a corruption, an operator error that replication faithfully copied. They overlap but are not the same: HA is about *redundancy and fast takeover*; DR is about *restoring from a separate copy of the data*. A common and costly mistake is to claim DR because you have HA replicas, then discover that a bad `DELETE` replicated everywhere in milliseconds.

Every HA design negotiates between three things: **data safety** (synchronous replication loses nothing but adds commit latency and can block writes), **availability** (automatic failover is fast but risks false positives and split-brain), and **cost/complexity** (more replicas, witnesses, orchestrators, tests). The technology comes second: Windows failover clustering plus availability groups, Postgres with Patroni, MySQL Group Replication or Galera, a managed service that hides it all — all implement the same pipeline: *detect → decide → fence → promote → reroute → rebuild*.

Platforms differ in how much they hide. Azure SQL Database, for example, builds HA in using quorum-based replicas across fault domains, zone-redundant replicas in higher tiers, and automatic failover that is transparent to applications — you do not configure clustering or mirroring yourself. On self-managed systems every step below is yours.

### 2. Core concepts and vocabulary
- **RPO (recovery point objective)** — maximum tolerable data loss, measured in time or transactions. **RTO (recovery time objective)** — maximum tolerable time until service is restored.
- **Availability** — fraction of time the service works; 99.9 % ≈ 8.8 h/yr down, 99.99 % ≈ 53 min/yr. Measure *application-observed* availability, not node uptime.
- **Synchronous / semi-synchronous / asynchronous replication** — the primary waits for the replica to harden (or receive) the commit / waits for at least one of N / does not wait.
- **Failover vs switchover** — unplanned promotion because of failure versus planned, graceful role swap (patching). Switchover can be lossless; failover may not be.
- **Quorum** — a majority of voting members that must agree; prevents two partitions both acting as primary.
- **Witness / arbiter / tie-breaker** — a lightweight voter that holds no data but breaks ties, typically placed in a third failure domain.
- **Fencing (STONITH)** — making sure the old primary cannot write: power it off, revoke storage/network, set read-only.
- **Split-brain** — two nodes accept writes as primary and histories diverge.
- **Flapping** — repeated failovers caused by a threshold hovering near its trigger.
- **VIP / DNS / proxy routing** — mechanisms to point clients at the current primary.
- **In-doubt transaction** — a distributed (2PC) transaction prepared on a participant whose coordinator was lost; it holds locks until resolved.

### 3. How it works, step by step
**Failover pipeline**
1. **Replicate.** Primary ships log records (WAL/redo/binlog/transaction log) to replicas. Mode decides what a "committed" transaction means: synchronous = hardened on the replica before the client gets an ack.
2. **Detect.** Heartbeats and health checks from more than one vantage point (cluster nodes, replica's view of the primary, an external probe). A single missed heartbeat must not trigger failover; use a timeout of several intervals (e.g. 3 × 5 s) to ride out GC pauses and brief network blips.
3. **Decide.** The controller (cluster manager, Patroni, orchestrator) needs *quorum*: a majority of voters must agree the primary is gone. Without a majority, do nothing — an unavailable cluster is better than two primaries.
4. **Fence.** Ensure the old primary cannot accept writes — before promoting anyone. A paused-then-resumed VM or a network-isolated primary may still think it is primary and serve clients with stale connections.
5. **Choose the candidate.** The replica with the most log applied (largest LSN/GTID set), subject to lag and locality rules. A lag ceiling avoids promoting a replica that is too far behind.
6. **Promote.** Replica finishes applying received log, recovers (undo uncommitted), becomes writable, and begins a new timeline/epoch.
7. **Reroute.** Update VIP/DNS/proxy/service discovery; kill stale connections; clients reconnect with retry and backoff.
8. **Re-protect.** Reattach other replicas to the new primary, rebuild the failed node as a replica (rewind or re-clone), and only then consider failback in a planned window.

**RPO vs RTO decision table**

| Replication | RPO | Commit latency | Failover |
|---|---|---|---|
| Synchronous, automatic | ~0 | + replica round-trip (and blocks if replica is down) | Automatic safe |
| Semi-sync (≥1 of N) | ~0 while ≥1 healthy | Small | Automatic |
| Asynchronous, automatic | Seconds–minutes of lag | None | Automatic but may lose data |
| Asynchronous, manual | Same, plus human time | None | Slow RTO |

### 4. Worked example
A write-heavy fintech service, three nodes across two zones plus a witness in a third zone, Postgres with synchronous replication to one standby (`ANY 1`) and an asynchronous second standby.

```sql
-- primary: require acknowledgement from at least 1 of the listed standbys
ALTER SYSTEM SET synchronous_standby_names = 'ANY 1 (replicaA, replicaB)';
ALTER SYSTEM SET synchronous_commit = 'remote_apply';  -- or on/remote_write
SELECT pg_reload_conf();

-- watch replication health
SELECT application_name, state, sync_state,
       pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn) AS replay_lag_bytes,
       write_lag, flush_lag, replay_lag
FROM   pg_stat_replication;
```

What happens when the primary's zone goes dark: the witness and the surviving zone's two nodes form a quorum; the controller waits the failure timeout, fences the dead node via the cloud API (or relies on its own lease expiring), picks the standby with the highest flushed LSN, promotes it, flips the proxy, and clients reconnect. Because at least one standby had acknowledged every commit, RPO is zero; RTO is detection timeout (~15–30 s) + promotion + client reconnect, typically under a minute.

Now the failure the design must survive: the *standby* dies. With a single synchronous standby, commits block. `ANY 1 (replicaA, replicaB)` with two candidates keeps writes flowing as long as one is alive. For a one-standby topology the controller must deliberately *degrade* (drop to async, alert loudly) or accept write unavailability — a decision the business has to make in advance, not at 3 a.m.

### 5. Under the hood
- **Why two nodes cannot be safe.** With two voters, a partition leaves each side thinking it has half the votes; either both stay up (split-brain) or both stop. A third voter — the witness, ideally in a third failure domain — makes a clear majority possible. For distributed data stores the equivalent is a replication factor of 3 with quorum reads/writes (R + W > N) or a consensus protocol (Raft/Paxos) electing a leader per term.
- **Leader leases and epochs.** A new leader carries a higher term/epoch/timeline; replicas and storage reject writes from stale terms. This is the software fence that backs the physical one.
- **Synchronous costs.** A synchronous commit adds one network round-trip (≈ 0.5–2 ms in-zone, 5–50 ms cross-region) plus a disk flush on the replica to *every* commit. Large batch/ETL loads queue commits behind the replica; the book's example is a warehouse ETL missing its window after availability groups were set to synchronous with automatic failover. Asynchronous mode fixes throughput but forces manual failover or accepted data loss — a trade-off, not a free setting.
- **The orchestrator is a SPOF unless built otherwise.** The failover controller must itself be HA: run several, back them with a consensus store (etcd/ZooKeeper/Consul or the cluster's own quorum), and use a leader lock so two controllers cannot duel.
- **Client-side recovery often dominates RTO.** The DB can promote in seconds while clients wait on: established TCP connections that never see a FIN (keepalives/timeouts), DNS TTL and resolver caching, pool entries holding dead sockets, and drivers that do not retry. Design for: short TCP keepalive/connect timeouts, validating pooled connections, retry with jittered backoff, and a routing layer (proxy/VIP) that moves faster than DNS.
- **In-doubt transactions.** If a database participates in 2PC, a promote can leave prepared transactions whose coordinator is gone; the new primary must recover prepared-transaction state (it is in the log) and a resolver must commit or roll back; sagas with idempotent compensations avoid the problem.
- **Cold standby.** After promotion buffer caches and plan caches are cold; a thundering herd of reconnects can overload the new primary. Warm standbys (read traffic, cache preload) and connection admission limits reduce this.

### 6. Performance and design implications
- Choose replication mode by *workload*: OLTP with money → synchronous/quorum; bulk ETL, analytics → asynchronous with a documented RPO, or exclude the bulk path.
- Keep voters and replicas in separate failure domains (zone/rack/power); more replicas in one zone buys nothing against zone loss.
- Set a **maximum replica lag** for promotion eligibility and for read routing; add hysteresis (consecutive failed checks, minimum time between failovers) to prevent flapping.
- Measure leading indicators of trouble: replication lag, WAL/log send queue, disk latency on replicas, quorum membership changes; lagging indicators (failover events) only tell you afterwards.
- Count failovers against SLOs using *application-observed* availability.
- Run game days: kill the primary, partition the network, fill the disk, fail the orchestrator. Untested HA is a hypothesis.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `synchronous_standby_names`, `synchronous_commit` (PostgreSQL) | Defines which standbys must ack and how far | Zero-RPO design |
| `pg_stat_replication`, `pg_promote()`, `pg_rewind` | Observe, promote, resync an old primary | Failover and re-attach |
| Availability group `AVAILABILITY_MODE = SYNCHRONOUS_COMMIT / ASYNCHRONOUS_COMMIT`, `FAILOVER_MODE = AUTOMATIC / MANUAL` (SQL Server) | Per-replica sync and failover behaviour | HA vs DR replicas |
| `sys.dm_hadr_database_replica_states` | Log send queue, redo queue, synchronization state | Lag monitoring |
| Semi-sync plugin / Group Replication / Galera | MySQL-family options with different consistency and write-conflict models | Choose by write pattern |
| Cluster quorum / witness config | Voting and tie-breaking | Any two-site design |
| Proxy health checks (HAProxy, ProxySQL, PgBouncer, cloud LB) | Route to current primary and remove failed nodes | Fast client rerouting |
| TCP keepalive, connect timeout, pool validation query | Client failure detection | Shorten client-side RTO |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| **Synchronous replication** — no data loss is mandatory (regulated finance) | High-volume batch loads; cross-region latency; single standby that blocks writes if it dies |
| **Asynchronous + automatic failover** — availability matters most, small loss acceptable | Auditors require provable no-loss; you must reconcile lost tail |
| **Manual failover** — risky automation, rare failures, small teams | Strict RTO; on-call fatigue |
| **Availability groups / equivalent** — need readable secondaries and per-database failover | You only need cheap DR; log shipping or backups may suffice |
| **Single large reliable instance** — small business, tolerates restore-time outage | Planned patching and failures are common; RTO/RPO targets are tight |
| **Multi-region active-passive** — regional DR | Needs data-loss decision and tested runbook; cross-region sync is slow |

### 9. Common mistakes
- Confusing HA with DR → corruption or deletion replicates → keep independent backups/point-in-time recovery and test restores.
- Two-node cluster without a witness → split-brain or stuck cluster → add a third voter in another failure domain.
- Promoting before fencing → two writers → fence first, always.
- Hair-trigger failure detection → flapping, failover storms → require consecutive failures, a minimum interval, lag eligibility rules.
- Assuming DNS changes are instant → clients keep hitting the dead node → short TTL, proxy/VIP, connection validation, tested retries.
- Synchronous commit with one standby → standby outage halts writes → multiple candidates or defined degrade-to-async policy.
- Never testing failover → the first real one fails → scheduled game days and periodic switchover.
- Treating "all nodes up" as healthy → replication broken silently → alert on lag and quorum, not just node liveness.

### 10. 🎯 Interview kit
**30-second answer:** Define RPO and RTO first. Use quorum-based replication across failure domains with a witness, synchronous (or semi-sync) where no data loss is allowed, health checks with hysteresis, fence the old primary before promoting the most up-to-date replica, and reroute clients through a proxy/VIP with retries. Keep backups separate because HA is not DR, and rehearse failover.

**Structure a longer answer:** clarify RPO/RTO and workload → replication mode and topology → detection, quorum, fencing, promotion → client routing and recovery → monitoring, testing, failback → close with a scenario (zone loss, standby loss, or a stuck 2PC).

**Likely follow-ups:**
- *Prevent flapping?* → multiple failed probes, grace period, lag threshold for promotion, cooldown between failovers.
- *How do you stop two orchestrators dueling?* → consensus-backed leader lock/lease, fencing tokens, epochs.
- *Why can a successful failover still produce an outage?* → TCP state, DNS TTL, pool caching, non-retrying clients.
- *Auditor wants provable no-loss?* → synchronous with acknowledged commit evidence (LSN/GTID comparison), failover logs, tested drills, immutable audit trail.
- *Run HA on Kubernetes?* → operator, anti-affinity across zones, storage that survives pod reschedule, graceful shutdown, PodDisruptionBudgets; underestimated: fencing and storage attach delay.

**Signals interviewers look for:** speaks in RPO/RTO, knows split-brain and fencing, separates HA from DR, treats the client path as part of the design, quantifies trade-offs, mentions testing.

### 11. Check yourself
- [ ] Draw the failover pipeline and name what can go wrong at each step.
- [ ] Explain why a two-node cluster is unsafe and how a witness fixes it.
- [ ] Compare synchronous, semi-sync and asynchronous replication on RPO, latency and blocking.
- [ ] List four client-side reasons a promoted database still looks down.
- [ ] Describe what you do when the synchronous standby dies.
- [ ] Explain the difference between HA and DR with an example that breaks one but not the other.
<!-- explainer:end -->

## Practice questions

- [Deep] [Walk me through how you'd set a maximum acceptable replication lag threshold for automated failover, and what heuristics prevent flapping when the standby is briefly behind.](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/High_Availability/Q-19356_Walk_me_through_how_you_d_set_a_maximum_acceptable_replicati.md>)
- [Deep] [How do TCP connection state, DNS TTLs, and connection-pool caching sabotage a technically-successful failover? Describe the full client-side recovery path you'd design for.](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/High_Availability/Q-19383_How_do_TCP_connection_state_DNS_TTLs_and_connection-pool_cac.md>)
- [Deep] [A probe: why not just use a single very large, very reliable database instance and skip HA complexity entirely? Argue both sides for a mid-size company.](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/High_Availability/Q-19409_A_probe_why_not_just_use_a_single_very_large_very_reliable_d.md>)
- [Deep] [How do you decide replication factor and consistency level for a globally-replicated NoSQL store balancing HA, latency, and storage cost across regions? Walk through a concrete tuning example.](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/High_Availability/Q-19442_How_do_you_decide_replication_factor_and_consistency_level_f.md>)
- [Expert] [Contrast MySQL Group Replication, Galera Cluster, and semi-sync replication with an external orchestrator. Which would you standardize on for a write-heavy fintech workload and why?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/High_Availability/Q-19361_Contrast_MySQL_Group_Replication_Galera_Cluster_and_semi-syn.md>)
- [Expert] [Explain how you'd design failover for a database that participates in distributed transactions (2PC/Saga) with other systems. What happens to in-doubt transactions during a promote?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/High_Availability/Q-19384_Explain_how_you_d_design_failover_for_a_database_that_partic.md>)
- [Expert] [Your automated failover controller itself needs to be HA. How do you design the orchestrator so it's not a SPOF, and how do you prevent two orchestrators from dueling?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/High_Availability/Q-19405_Your_automated_failover_controller_itself_needs_to_be_HA_How.md>)
- [Expert] [How does the choice of failover strategy change for a database backing regulated financial reporting where auditors require provable no-data-loss? What evidence do you need to produce?](<../../../PRACTICE/01_Databases_and_SQL/Database_Operations/High_Availability/Q-19427_How_does_the_choice_of_failover_strategy_change_for_a_databa.md>)

## Related concepts

- [Zero Downtime Sharding Migrations](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Zero_Downtime_Sharding_Migrations.md>) *(Distributed Systems Core)*
- [Quorums and Consensus](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Quorums_and_Consensus.md>) *(Distributed Systems Core)*
- [Replication Topologies and Lag](<../Replication/Failback_After_Failover/Replication_Topologies_and_Lag.md>) *(Replication)*
- [Slow Dependencies and Load Shedding](<../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Deadline_Propagation/Slow_Dependencies_and_Load_Shedding.md>) *(Distributed Systems Core)*
- [Read Your Writes with Replicas](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Read_Your_Writes_with_Replicas.md>) *(Cloud Fundamentals)*
- [Availability Zone Resilience](<../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Availability_Zone_Resilience/_index.md>) *(Cloud Fundamentals)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › High Availability (HA) Architecture
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Geo-Replication and Failover Groups
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Business Continuity in Elastic Pools
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Availability › 13 Availability
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Availability › 13.1 #89 Confusing HA and DR
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Availability › 13.2 #90 Failing to architect for the requirements
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Availability › 13.4 #92 Assuming availability groups are always the right answer
- *Database Internals* — Alex Petrov — Replication and Consistency › Achieving Availability
- *Chaos Engineering: System Resiliency in Practice* — Casey Rosenthal, Nora Jones — Chaos Engineering on a Database › Robustness and Stability
- *Designing Data-Intensive Applications* — Martin Kleppmann — Setting Up New Followers › Handling Node Outages
- *Designing Data-Intensive Applications* — Martin Kleppmann — Setting Up New Followers › Leader failure: Failover
- *Hadoop: The Definitive Guide* — Tom White — ZooKeeper › Implementation

