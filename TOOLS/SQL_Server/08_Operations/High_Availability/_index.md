---
id: "TOOLS/SQL_Server/08_Operations/High_Availability"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# High Availability

**L4 Advanced** · [↑ Operations and Administration](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** High availability (HA) keeps the service running through local failures with automatic failover; disaster recovery (DR) restores service after losing a site; they use different designs, cost and failure scenarios, and neither replaces backups.

### 1. The big picture
Business owners say "make it highly available" without defining what failures they mean. The DBA's job is to translate requirements (RTO, RPO, the failures to survive) into a topology, and to make sure everyone understands what is and isn't covered. The most common mistakes are conceptual: calling an in-datacenter HA cluster "DR", picking Availability Groups for every workload, and loading clusters so heavily they cannot absorb a failover.

### 2. Core concepts and vocabulary
- **High availability (HA)** — automatic failover to redundant hardware, usually in the same data center, so latency is low and failover quick: a **hot standby**.
- **Disaster recovery (DR)** — recovering from a critical incident (corruption, dead server, loss of a data center or availability zone). The simplest form is backup/restore; faster forms keep a **warm standby** in another location, kept synchronized.
- **Failover Cluster Instance (FCI)** — a Windows failover cluster in which a SQL Server *instance* moves between nodes; storage is shared and stored once. Instance-level, single-site in practice.
- **Availability Group (AG)** — database-level replication of log records to up to several replicas. Can serve HA (synchronous commit, automatic failover), DR (asynchronous, manual failover) and read scale-out.
- **Synchronous commit** — primary waits for the secondary to harden the log before committing: no data loss, added latency.
- **Asynchronous commit** — primary doesn't wait: faster, possible data loss.
- **Log shipping** — copy and restore log backups on a schedule to a standby; simple, `FULL` model required.
- **Active/active cluster** — every node hosts production databases and nodes fail over to each other; it is a way to use the hardware, *not* load balancing for writes.
- **Single point of failure (SPOF)** — any component whose failure takes the service down (server, rack, switch, data center, storage).
- **Quorum** — the vote that determines which cluster members may keep running after a partition.

### 3. How it works, step by step
**Selecting a design**
1. State requirements: RTO, RPO, which failures must be survived (server, rack, data center, region), and whether automatic failover is required.
2. List the SPOFs in the proposed design and check each against those failures.
3. Match technology to need:
   - Local, automatic, instance-level, no extra copy of data → **FCI**.
   - Database-level, readable secondaries, HA and/or DR → **AG**.
   - Cheap DR, tolerate minutes of loss → **log shipping** or AG async.
   - Cloud → built-in zone redundancy, geo-replication, failover groups.
4. Test the failover and recovery procedures; document who does what.
5. Capacity-plan for the *failed-over* state.

**What each failure needs**

| Failure | Covered by |
|---|---|
| Server/OS crash | FCI or AG (HA) |
| Rack/switch failure | HA spanning racks |
| Data center loss | DR in another site (AG async, log shipping, restore) |
| Accidental delete/corruption | Backups, point-in-time restore (replicas copy the mistake) |
| Region outage (cloud) | Geo-replication/failover groups |

### 4. Worked example
The data warehouse that couldn't use an AG:

1. A critical warehouse in `SIMPLE` recovery, a heavy nightly ETL. Business wants HA with automatic failover.
2. Team configures an AG with **synchronous commit**. ETL now misses its window: each commit waits for the secondary to harden the log, so commits queue.
3. Switch to **asynchronous commit**: automatic failover is lost, but ETL is still too slow.
4. Root cause: an AG requires `FULL` recovery. The ETL now logs every page allocation instead of extents; far more log I/O.
5. Conclusion: an FCI gives HA with a single copy of data on shared storage, no change of recovery model and no replica synchronization cost. For DR, ship backups offsite (the DR requirement is a day anyway).

Quick AG health check:

```sql
SELECT ag.name AS ag_name, r.replica_server_name, rs.role_desc,
       rs.synchronization_health_desc, d.database_name,
       drs.synchronization_state_desc, drs.log_send_queue_size, drs.redo_queue_size
FROM sys.availability_groups ag
JOIN sys.availability_replicas r ON r.group_id = ag.group_id
JOIN sys.dm_hadr_availability_replica_states rs ON rs.replica_id = r.replica_id
JOIN sys.dm_hadr_database_replica_states drs ON drs.replica_id = r.replica_id
JOIN sys.availability_databases_cluster d ON d.group_id = ag.group_id AND d.group_database_id = drs.group_database_id;
```

Growing `log_send_queue_size` means potential data loss on failover (async) or stalled commits (sync).

### 5. Under the hood
- **FCI**: shared storage is attached to exactly one node; on failure the cluster brings the SQL resources up on another node, runs crash recovery, and clients reconnect via the virtual network name. Downtime is the detection plus recovery time; there's one copy of the data, so no replication overhead and no recovery-model requirement.
- **AG**: primary sends log blocks to secondaries, which harden them and redo them continuously. It sits on a Windows Server Failover Cluster (or a cluster-less configuration in some cases) for health detection and listener management. Operation is at *database* level, so only critical databases can be included, but it requires `FULL` recovery, adds log I/O, and secondary replicas cost licences and storage.
- **Synchronous vs. asynchronous**: RPO of zero vs. possible loss, depending on network and the secondary's speed; an undersized secondary builds a send/redo queue.
- **Active/active**: two nodes each primary for different AGs/instances; after a failover one node must carry both loads. If steady-state load already uses most of each node, the survivor is overloaded: timeouts, lost monitoring, even cluster instability.
- **Peer-to-peer replication** is *not* a load-balancing or HA tool: no cluster front, applications connect to nodes directly, conflict handling is rudimentary and stops the distribution agent. Replication is not suitable as a DR tool.
- **Azure SQL Database/Managed Instance**: HA is built in: quorum-based replicas across fault domains, optional zone redundancy, transparent automatic failover. For DR, active geo-replication (up to four readable secondaries) and auto-failover groups across paired regions, including whole elastic pools.

### 6. Performance and design implications
- **Synchronous commit adds commit latency** (`HADR_SYNC_COMMIT` waits) proportional to network round trip + secondary log flush. Keep sync replicas close and well provisioned; use async across distance.
- **`FULL` recovery cost** applies whenever AG/log shipping are used; very write-heavy ETL warehouses may prefer FCI or a cloud MPP service.
- **Headroom**: size each node for the *post-failover* load, or limit what runs on each. Active/active saves money but accepts degraded performance after failover, and must be revisited as load grows.
- **Secondaries can offload** read-only queries and backups, but need licensing and monitoring.
- **Monitor queues**, not just "synchronized" state.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE AVAILABILITY GROUP` / `ALTER AVAILABILITY GROUP … FAILOVER` | Define and fail over an AG | Build; planned/forced failover |
| `AVAILABILITY_MODE = SYNCHRONOUS_COMMIT / ASYNCHRONOUS_COMMIT` | Commit behavior per replica | HA vs. DR replicas |
| `FAILOVER_MODE = AUTOMATIC / MANUAL` | Failover policy | Match requirement |
| `sys.dm_hadr_database_replica_states` | Queues, lag, state | Health and RPO monitoring |
| `sp_help_log_shipping_*`, log shipping jobs | Manage log shipping | Simple DR |
| `SERVERPROPERTY('IsClustered')`, `sys.dm_os_cluster_nodes` | Cluster info | Inventory |
| Failover cluster manager, `Get-Cluster*` | Cluster administration | Cluster checks |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| FCI: instance-level HA, single data copy, `SIMPLE` models, big write loads | Need DR across sites without SAN replication; shared-storage SPOF |
| AG sync: zero data loss locally, readable secondaries | Heavy bulk loads; high network latency |
| AG async: cross-site DR | Requirement for zero data loss or automatic failover |
| Log shipping: cheap, simple DR | Need low RPO or automatic failover |
| Cloud-built-in HA/geo-replication | Need features/versions not available there |

### 9. Common mistakes
- **Calling HA "DR"** → the business thinks a data center loss is covered → separate HA and DR requirements and test both.
- **Never testing DR** → plan fails during a real disaster → rehearse failover/restore on a schedule.
- **AG for everything** → ETL slowdown, forced `FULL` model → consider FCI/other technologies.
- **Overloaded active/active cluster** → survivor collapses after failover → capacity-plan the failover state.
- **Treating active/active as load balancing** → write-scaling expectations are wrong → only one primary per database.
- **Using replication as DR** → high complexity, fragile → use AG, log shipping or backup/restore.
- **HA without backups** → corruption/delete replicated → keep tested backups (see Backup and Restore).

### 10. 🎯 Interview kit
**30-second answer:** HA is about automatic failover from local failures using hot standby in the same data center (FCI or synchronous AG). DR is about recovering from a site-level disaster using a warm standby elsewhere or backups (async AG, log shipping). They aren't interchangeable, and neither protects against logical corruption. That's what backups are for.

**Structure a longer answer:** requirements (RTO/RPO, failure scenarios) → list SPOFs → compare FCI/AG/log shipping/cloud → note recovery-model and performance consequences → capacity for failover → test and document.

**Likely follow-ups:**
- *FCI vs. AG?* → Instance vs. database level; shared storage vs. separate copies; AG needs `FULL` and replicates log, supports readable secondaries and DR.
- *Why did synchronous AG slow our ETL?* → Commit waits on the secondary hardening the log; plus `FULL` logging volume.
- *Is active/active load balancing?* → No; one writable copy per database.
- *How would you design HA + DR?* → Sync AG with auto-failover locally plus async AG to a remote site with manual failover.
- *What can't an AG protect against?* → Logical errors: a bad `DELETE` is replicated.

**Signals interviewers look for:** you separate HA and DR, tie choices to RTO/RPO, are aware of costs/limits (recovery model, latency, headroom), and insist on testing.

### 11. Check yourself
- [ ] Define HA and DR and give a failure that only DR covers.
- [ ] Compare FCI and AG on data copies, scope, recovery model and DR capability.
- [ ] Explain how synchronous commit affects latency and RPO.
- [ ] Describe what happens to an active/active two-node cluster that is 70% loaded per node during failover.
- [ ] Explain why replication isn't a DR strategy.
- [ ] Sketch a topology giving local automatic failover and remote manual DR.
<!-- explainer:end -->

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Geo-Replication and Failover Groups
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Availability › 13.5 #93 Overloading a cluster
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Availability › 13.4 #92 Assuming availability groups are always the right answer
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Availability › 13.1 #89 Confusing HA and DR
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Availability › 13 Availability
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › High Availability (HA) Architecture
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Business Continuity in Elastic Pools

