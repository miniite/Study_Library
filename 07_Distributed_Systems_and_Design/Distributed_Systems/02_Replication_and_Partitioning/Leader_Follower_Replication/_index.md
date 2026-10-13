---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/02_Replication_and_Partitioning/Leader_Follower_Replication"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Leader-Follower Replication

**L4 Advanced** · [↑ Replication and Partitioning](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** In leader-follower replication all writes go to one leader, which streams an ordered change log to read-only followers; the key decisions are synchronous vs asynchronous shipping, how to add followers safely, and how to fail over when nodes die.

### 1. The big picture
Leader-based (active/passive, master-slave) replication is the default in PostgreSQL, MySQL, Oracle Data Guard, SQL Server Availability Groups, MongoDB, and also in Kafka partitions and RabbitMQ queues. One replica is the **leader**; clients send every write there. The leader writes locally and ships the change to its **followers**, which apply changes in the same order. Reads can be served by the leader or any follower, which scales read throughput.

Because there is exactly one writer per partition, there are no write conflicts and ordering is trivial. The costs: writes cannot scale beyond the leader, followers may lag, and a leader failure requires a failover procedure that is subtle and dangerous.

### 2. Core concepts and vocabulary
- **Leader / primary / master** — the replica that accepts writes.
- **Follower / secondary / replica / hot standby** — read-only copy applying the leader's log.
- **Replication log (change stream)** — ordered record of changes (see Replication Logs).
- **Synchronous follower** — leader waits for its acknowledgement before confirming the write.
- **Asynchronous follower** — leader does not wait.
- **Semi-synchronous** — one follower synchronous, rest asynchronous.
- **Log sequence number (LSN) / binlog coordinates** — a position in the replication log.
- **Catch-up recovery** — a restarted follower requests the log from its last processed position.
- **Failover** — promoting a follower to leader after leader failure.
- **Split brain** — two nodes both believe they are the leader.
- **Chain replication** — synchronous variant where writes pass along a chain of replicas.

### 3. How it works, step by step
**Synchronous vs asynchronous write path**
1. Client sends the update to the leader.
2. Leader persists it and sends the change to followers.
3. Synchronous follower acknowledges; leader then confirms to the client and makes the write visible.
4. Asynchronous followers apply later; there is no delivery or latency guarantee (usually well under a second, sometimes minutes during recovery or overload).

Fully synchronous replication is impractical: one unavailable follower would block all writes. In practice at most one follower is synchronous, guaranteeing that at least two nodes (leader + that follower) have the latest data; if it slows, an asynchronous follower is promoted to synchronous. Fully asynchronous maximizes availability and throughput but a leader crash can lose acknowledged writes.

**Setting up a new follower without downtime**
1. Take a consistent snapshot of the leader (no global lock).
2. Copy the snapshot to the new node.
3. The follower connects to the leader and requests all changes after the snapshot's exact log position (LSN / binlog coordinates).
4. When the backlog is processed it has *caught up* and follows the live stream.

**Handling node outages**
- *Follower failure*: it knows the last transaction it processed from its local log; after restart it requests changes since then (catch-up recovery).
- *Leader failure (failover)*:
  1. Detect failure (typically timeouts, e.g., no heartbeat for 30 s).
  2. Choose a new leader (election; best candidate = most up-to-date replica; or a controller appoints).
  3. Reconfigure clients and followers to use it; the old leader, if it returns, must become a follower.

### 4. Worked example
```sql
-- PostgreSQL: one synchronous standby, others async
-- on the primary (postgresql.conf)
synchronous_commit = on
synchronous_standby_names = 'FIRST 1 (standby_a, standby_b)'

-- see replication state
SELECT application_name, state, sync_state,
       pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn) AS lag_bytes
FROM   pg_stat_replication;
```
Reading the result: `sync_state` shows `sync` for the chosen standby and `potential` or `async` for others; `lag_bytes` growing means the follower is falling behind. If `standby_a` dies, `standby_b` is promoted to `sync` automatically, so writes keep flowing with two-node durability.

### 5. Under the hood
- **Failover hazards**: with asynchronous replication the new leader may lack writes the old leader acknowledged; if the old leader returns, those writes conflict and are usually discarded, violating durability and possibly corrupting external systems (e.g., reused auto-increment IDs leaked to a cache). Two nodes believing they are leader (split brain) can corrupt data; fencing (STONITH) and epoch/term numbers prevent a stale leader from acting.
- **Timeout choice**: too short causes unnecessary failovers during load spikes (making things worse); too long delays recovery.
- **Election algorithms**: ring algorithm (circulates the live set around a ring, highest rank wins, but a partitioned ring elects multiple leaders and has no safety), bully algorithm, and consensus-based election (Raft, ZooKeeper/Zab) that guarantee a single leader per term via majority votes.
- **Consensus link**: reliable leader change is a consensus problem; many systems delegate it to a coordination service.
- **Chain replication**: writes go head→tail, reads from tail, giving strong consistency and good throughput (used in Azure Storage).

### 6. Performance and design implications
- Read scaling with replicas works for read-heavy workloads, but only if you accept stale reads from asynchronous followers.
- Synchronous replication adds a network round trip (and the follower's fsync) to every commit; cross-region sync is usually too slow.
- Monitor replication lag in bytes and seconds; alarm before it exceeds your freshness SLO.
- Plan runbooks and test failover regularly; automated failover without fencing can cause more outages than it fixes.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `synchronous_standby_names` (PostgreSQL) | Defines synchronous standbys | Zero-data-loss requirement |
| `rpl_semi_sync_master_enabled` (MySQL) | Semi-synchronous replication | Durability with bounded latency |
| `pg_basebackup` / `xtrabackup` | Take snapshot to seed a follower | Adding a replica |
| `pg_promote()` / `SET GLOBAL read_only` | Promote / fence | Manual failover |
| Kafka `acks=all`, `min.insync.replicas` | ISR-based synchronous guarantee | Durable topic writes |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Synchronous (one follower): cannot lose committed data | Follower is remote/high latency |
| Asynchronous: throughput and availability matter, small loss window acceptable | Money or irreplaceable data without other protection |
| Automatic failover: strict RTO | You cannot fence the old leader reliably |
| Read replicas: read-heavy, tolerates staleness | Needs read-your-writes without extra routing logic |

### 9. Common mistakes
- Making every follower synchronous → single outage halts writes → one sync, rest async.
- Copying data files for a new follower → inconsistent snapshot → use proper snapshot + log position.
- Aggressive failover timeout → flapping leaders → tune and add hysteresis.
- Reviving the old leader with unreplicated writes → conflicting history → demote and rewind or rebuild it.
- No fencing → split brain.

### 10. 🎯 Interview kit
**30-second answer:** One leader takes all writes and streams an ordered log to followers. Synchronous followers give durability at the price of availability; asynchronous ones give speed with potential data loss on failover. New followers start from a snapshot plus log position; failover needs failure detection, election and fencing.

**Structure a longer answer:** write path → sync/async trade-off → follower bootstrap → failure modes (follower vs leader) → split brain and consensus → real example (Postgres or Kafka ISR).

**Likely follow-ups:**
- *What can go wrong in failover?* → lost writes, split brain, conflicting auto-increment keys, wrong timeouts.
- *Why not all synchronous?* → availability collapses.
- *How do you add a replica live?* → consistent snapshot + replay from recorded LSN.

**Signals interviewers look for:** precise sync/async consequences, fencing/epoch awareness, knowledge that leader election is consensus.

### 11. Check yourself
- [ ] Draw the message flow with one synchronous and one asynchronous follower.
- [ ] List the steps to add a follower with no downtime.
- [ ] Explain three failover hazards.
- [ ] Describe why the ring algorithm lacks safety under partitions.
<!-- explainer:end -->

## In this section (study in order)

1. [Replication Log Implementations](<Replication_Logs/_index.md>) (L5 Expert)
2. [Replication Lag and Read Guarantees](<Replication_Lag/_index.md>) (L5 Expert)

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Setting Up New Followers › Handling Node Outages
- *Designing Data-Intensive Applications* — Martin Kleppmann — Setting Up New Followers › Leader failure: Failover
- *Designing Data-Intensive Applications* — Martin Kleppmann — Setting Up New Followers
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 5. Replication › Synchronous Versus Asynchronous Replication
- *Designing Data-Intensive Applications* — Martin Kleppmann — Fault-Tolerant Consensus › Single-leader replication and consensus
- *Designing Data-Intensive Applications* — Martin Kleppmann — M
- *Database Internals* — Alex Petrov — Leader Election › Ring Algorithm

