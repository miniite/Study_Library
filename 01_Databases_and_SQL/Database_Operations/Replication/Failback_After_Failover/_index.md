---
id: "01_Databases_and_SQL/Database_Operations/Replication/Failback_After_Failover"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Operations/Replication"
tools: ["MySQL"]
---

# Failback After Failover

**L4 Advanced** · [↑ Replication](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Failover moves writes to a replica; failback moves them back to the original primary only after you have proven the old primary holds no writes the new one lacks, and stopped it from accepting writes while you prove it.

### 1. The big picture
Failover is the emergency half; failback is the controlled half and is where data gets quietly damaged. When the old primary comes back, it is not a neutral spare: it may hold **committed transactions the new primary never received** (asynchronous replication), it may still believe it is the primary and accept writes from clients with stale connections (**split-brain**), and its data directory has diverged from the new primary's timeline.

The safe default is not to "fail back" at all in the sense of swapping roles instantly. Instead, treat the old primary as a *broken replica*: fence it, reconcile or discard its divergent tail, rebuild or re-attach it as a replica of the new primary, let it catch up, and only then — optionally, in a planned maintenance window — perform an ordinary controlled switchover. This lesson builds on the replication modes in the topologies lesson and underpins the schema-change/failover lesson that follows.

### 2. Core concepts and vocabulary
- **Failover** — promoting a replica to primary because the primary failed (unplanned) or is being maintained (planned, often called *switchover*).
- **Failback** — returning the primary role to the original node.
- **Fencing (STONITH)** — making sure the old primary cannot accept writes: power off, revoke network/storage access, set `read_only`, remove from the load balancer.
- **Split-brain** — two nodes both accept writes as primary; their histories diverge.
- **Errant (divergent) transaction** — a transaction that exists on the old primary but not on the new one; with GTIDs it appears as GTIDs in the old primary's executed set that the new primary lacks.
- **RPO / RTO** — recovery point objective (how much data you may lose) and recovery time objective (how long you may be down).
- **Timeline** — in log-based systems, the branch of history after a promotion; the old primary is on a different branch.
- **Rewind / re-clone** — either roll the old primary's divergent tail back (e.g. `pg_rewind`, flashback tooling) or discard its data and rebuild from a backup/snapshot of the new primary.

### 3. How it works, step by step
**Phase A — at failover (so failback is possible later)**
1. Detect failure with more than one signal (heartbeat + replica's view + external probe) to avoid false positives.
2. **Fence** the old primary before promoting anything. Promotion without fencing is how split-brain is born.
3. Pick the most advanced replica (largest executed GTID set / log position). Let it finish applying its relay log; only then promote it.
4. Repoint the other replicas and the application endpoint to the new primary.
5. Record the old primary's last known GTID/log position — this is the evidence for step C.

**Phase B — when the old primary returns**
1. Keep it fenced and read-only. Start it with replication disabled and clients blocked.
2. Compare its executed log position/GTID set with the new primary's.
3. Decide per case:
   - *Old ⊆ new:* nothing divergent. Attach as a replica; it simply catches up.
   - *Old has extras (errant transactions):* export those transactions (read its binlog/WAL), have the business decide whether to re-apply them on the new primary (reconcile, with conflict checks) or drop them, then rebuild/rewind.
4. Rebuild: restore from a recent backup or clone from the new primary, and set it to replicate from there.

**Phase C — planned failback (optional)**
1. Confirm the old node is a healthy replica with lag ≈ 0.
2. Announce a window; lower connection limits; stop writes on the current primary (`read_only`, or drain the app).
3. Wait until the replica's executed set equals the primary's.
4. Promote the old node, repoint clients and replicas, and turn the interim primary into a replica.
5. Verify with checksums and application smoke tests; then reopen.

### 4. Worked example
Errant-transaction detection with GTIDs (MySQL):

```sql
-- On the NEW primary (promoted yesterday)
SELECT @@GLOBAL.gtid_executed;
-- 3E11FA47-...:1-98200, 7B91C0AA-...:1-120

-- On the OLD primary, once it is back (fenced, read_only=ON)
SELECT @@GLOBAL.gtid_executed;
-- 3E11FA47-...:1-98215

-- What does the old one have that the new one does not?
SELECT GTID_SUBTRACT('3E11FA47-...:1-98215', '3E11FA47-...:1-98200') AS errant;
-- 3E11FA47-...:98201-98215
```

Fifteen transactions (98201–98215) were committed on the old primary after the last position the new primary received. Next:
1. Decode them: `mysqlbinlog --include-gtids='3E11FA47-...:98201-98215' <binlog>`.
2. Classify: are they orders/payments (must be re-applied or compensated) or session noise?
3. Apply reconciled changes on the **new** primary through the normal write path.
4. Rebuild the old node from a fresh backup of the new primary and `CHANGE REPLICATION SOURCE TO … SOURCE_AUTO_POSITION=1`.

Never "just start replication" on the old primary: the replica will either refuse (GTIDs not in the source's history) or, worse with file/position replication, run and diverge silently.

### 5. Under the hood
- **Why the tail exists.** In async mode the primary acknowledges a commit before shipping it. A crash at that instant leaves committed-and-acknowledged data only on the dead node. Semi-sync narrows the window to "received by one replica", but its silent fallback to async during a network blip reopens it.
- **Why recovery of the commit point is hard.** Distributed commit protocols face the same shape of problem: in two-phase commit, if the coordinator dies after collecting votes but before broadcasting the decision, participants are *uncertain* — they hold locks and cannot decide alone; they must learn the outcome from peers' logs or a backup coordinator. Replication failback has an analogous "uncertain tail" that you resolve by consulting the logs on both sides, not by guessing.
- **Timelines.** PostgreSQL gives the promoted node a new timeline ID; the old primary's WAL beyond the branch point is invalid on the new timeline, which is why `pg_rewind` can undo it by copying only changed blocks.
- **Quorum systems** (Raft/Paxos-based clusters, Galera, Always On with quorum) avoid the problem by requiring a majority to commit: a minority partition cannot accept writes, so a returning node has no unique committed tail. The cost is write availability in a minority partition.

### 6. Performance and design implications
- A full re-clone of a multi-TB node can take hours; keep recent snapshots/backups and a tested restore time so RTO for "rebuild old primary" is a known number.
- Replication catching up after rebuild competes with production reads; throttle clone/restore I/O and schedule failback off-peak.
- Fencing mechanisms must be **independent of the thing that failed**: a script that SSHes into a dead host cannot fence it. Use out-of-band management or storage/network-level isolation.
- Rehearse: game-day failover and failback is the only way to know the timing and the gaps in the runbook.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `read_only`, `super_read_only` | Prevent writes on returning node | Immediately on recovery |
| `GTID_SUBTRACT(a, b)` | GTIDs in `a` not in `b` | Finding errant transactions |
| `mysqlbinlog --include-gtids` | Decode specific transactions | Reconciliation |
| `SOURCE_AUTO_POSITION=1` | GTID-based re-attachment | Re-pointing replicas after promotion |
| `pg_rewind` | Rewind diverged PG node to common point | Fast re-join of old PG primary |
| `pg_basebackup` / XtraBackup / snapshot | Rebuild from new primary | When divergence is too large |
| Orchestrator / Patroni / managed-service failover | Automate detection, fencing, promotion | Fleet operations |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Re-clone the old node | Dataset so large that rebuild exceeds your RTO and no snapshot exists |
| Rewind (`pg_rewind`) | WAL needed for rewind has been removed; checksums/`wal_log_hints` not enabled |
| Stay on the new primary permanently | Hardware/region of the new primary is unsuitable (smaller box, wrong region) |
| Planned switchover back | Mid-peak traffic, or replica lag not provably zero |
| Re-apply errant transactions | You cannot detect conflicts with writes made on the new primary since |

### 9. Common mistakes
- Promoting before fencing → two writers → fence first, always.
- Promoting a replica that hasn't finished its relay log → lost committed data → compare executed sets before promote.
- Letting the old primary rejoin automatically at boot → it re-announces as primary → boot into read-only/disabled replication.
- Using replication as the backup of the failed node's data → corruption and mistakes were copied too → keep real backups and PITR.
- Failing back immediately "to restore normal" → second outage for no benefit → treat it as a planned change.
- Forgetting application-side state (caches, queued jobs, sequences) that already moved on.

### 10. 🎯 Interview kit
**30-second answer:** Fence the old primary, compare its log position with the new one, handle any errant transactions deliberately, rebuild it as a replica, let it catch up, then do a planned, zero-lag switchover if you still want the original topology.

**Structure a longer answer:** clarify (RPO, sync mode, tooling) → core mechanism (fence → compare → reconcile → rebuild → switchover) → trade-offs (re-clone vs rewind; stay vs return) → real scenario (payments cannot lose committed tx: semi-sync + GTID audit + reconcile through application path).

**Likely follow-ups:**
- *How do you prevent split-brain?* → fencing independent of the failed node, single authority (quorum or external coordinator), clients behind a controlled endpoint.
- *What replication topology for no loss of committed transactions?* → synchronous/quorum commit across at least two failure domains, accepting latency and availability trade-offs; prove with fault-injection tests and log comparison.
- *How do you find missing transactions after failover?* → GTID set subtraction or log position comparison, decode binlog.
- *Why not restore from the replica?* → replication is not a backup.

**Signals interviewers look for:** fencing-first mindset, awareness of the uncertain tail, refusal to auto-rejoin, rehearsed runbook.

### 11. Check yourself
- [ ] List the exact order: fence, choose candidate, catch up, promote, repoint — and say what breaks if you swap two.
- [ ] Show how to compute errant transactions with GTID sets.
- [ ] Explain why an async primary can hold acknowledged transactions the replica lacks.
- [ ] Compare re-clone, rewind and "re-apply the tail" for rejoining an old primary.
- [ ] Describe a planned failback with zero lost writes and no split-brain at any step.
<!-- explainer:end -->

## Go deeper

- [Schema Changes and Failover](<Schema_Changes_and_Failover.md>)
- [Replication Topologies and Lag](<Replication_Topologies_and_Lag.md>)

## Implementations (tools)

- [MySQL](<../../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Intermediate] [Walk me through what happens end-to-end when a write commits on the master and is replicated to a slave in an asynchronous setup.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Replication/Q-21169_Walk_me_through_what_happens_end-to-end_when_a_write_commits.md>)
- [Intermediate] [How do you detect and recover from a replica whose SQL thread has stopped due to a duplicate key error?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Replication/Q-21181_How_do_you_detect_and_recover_from_a_replica_whose_SQL_threa.md>)
- [Intermediate] [A slave fell far behind and you need to rebuild it. Walk through re-cloning it from a consistent backup without stopping the master.](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Replication/Q-21186_A_slave_fell_far_behind_and_you_need_to_rebuild_it_Walk_thro.md>)
- [Deep] [How do you promote a slave to master safely during a planned master maintenance window without losing writes?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Replication/Q-21174_How_do_you_promote_a_slave_to_master_safely_during_a_planned.md>)
- [Deep] [During a network partition between master and replica, what state does the I/O thread enter and how does it recover?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Replication/Q-21213_During_a_network_partition_between_master_and_replica_what_s.md>)
- [Deep] [What consistency guarantees does a synchronous replica give you, and what is the availability cost if that replica goes down?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Replication/Q-21239_What_consistency_guarantees_does_a_synchronous_replica_give_.md>)
- [Deep] [A replica keeps disconnecting with 'Got fatal error 1236 from master'. What are the common root causes and how do you fix each?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Replication/Q-21251_A_replica_keeps_disconnecting_with_Got_fatal_error_1236_from.md>)
- [Deep] [How do you handle replication of a partitioned table where partition maintenance (DROP PARTITION) runs on the master?](<../../../../PRACTICE/01_Databases_and_SQL/Database_Operations/Replication/Q-21262_How_do_you_handle_replication_of_a_partitioned_table_where_p.md>)

## Related concepts

- [Consistent Backups and Restore Safety](<../../Backup_and_Recovery/Consistent_Backups_and_Restore_Safety/_index.md>) *(Backup and Recovery)*
- [Zero Downtime Sharding Migrations](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Zero_Downtime_Sharding_Migrations.md>) *(Distributed Systems Core)*
- [HA Failover Design](<../../High_Availability/HA_Failover_Design.md>) *(High Availability)*

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Multi-Leader Replication Topologies › Writing to the Database When a Node Is Down
- *Designing Data-Intensive Applications* — Martin Kleppmann — Don't confuse 2PC and 2PL › Recovering from coordinator failure
- *Database Reliability Engineering* — Laine Campbell, Charity Majors — Backup and Recovery › Replication Is Not a Backup!

