---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/02_Replication_and_Partitioning/Leader_Follower_Replication/Replication_Lag"
level: 5
type: "curriculum"
curriculum: "data_engineering"
---

# Replication Lag and Read Guarantees

**L5 Expert** · [↑ Leader-Follower Replication](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Asynchronous followers lag behind the leader, so reads from them can be stale; consistency models (from eventual to linearizable) and session guarantees (read-your-writes, monotonic reads, consistent prefix) define how much staleness an application can see.

### 1. The big picture
Leader-based replication with read replicas is attractive for read-heavy workloads: add followers to scale reads. But followers are asynchronous, so a read served by a lagging follower returns old data. If the system is idle the lag shrinks to nothing (*eventual consistency*), but "eventually" has no time bound, and under load or after recovery it can be seconds or minutes. The application must be designed for that, or the database must provide stronger guarantees. This lesson maps the spectrum of consistency models and the practical anomalies caused by lag.

### 2. Core concepts and vocabulary
- **Replication lag** — delay between a write on the leader and its visibility on a follower.
- **Strict consistency** — every write instantly visible everywhere; requires a global clock; a theoretical ideal, impossible in practice.
- **Linearizability** — the strongest practical model: each operation appears to take effect atomically at one instant between its start and end; once any read returns a value, all later reads return that value or newer. Behaves as if there were a single copy.
- **Sequential consistency** — all processes see operations in the same order, but not necessarily real-time order.
- **Eventual consistency** — if no new updates arrive, all replicas eventually return the last value.
- **Tunable consistency** — per-operation choice of replica counts (r, w, n).
- **Read-your-writes** — a user always sees their own writes.
- **Monotonic reads** — successive reads never go back in time.
- **Consistent prefix reads** — reads see writes in causal order.

### 3. How it works, step by step
Anomalies from lag and their fixes:
1. **Reading your own write**: user posts a comment (to leader) and refreshes (served by lagging follower) → comment vanishes. Fix: read-your-writes — route reads of user-modified data to the leader (e.g., own profile), or for a window after a write; or track the last write timestamp/LSN and only use replicas caught up beyond it.
2. **Time moving backward**: two successive reads hit different replicas, the second one more lagged. Fix: monotonic reads — pin each user to one replica (hash of user ID).
3. **Causality violations**: a reply appears before the question because different partitions replicate at different speeds. Fix: consistent prefix reads — write causally related data to the same partition, or track causality explicitly.
4. **Cross-device**: the user writes on laptop and reads on phone; session stickiness fails, so route by user not by device.

### 4. Worked example
```sql
-- read-your-writes with LSN tracking (PostgreSQL concept)
-- after a write on the leader:
SELECT pg_current_wal_lsn();        -- e.g. 0/3000148   (store in the user's session)
-- before a follower read, check the replica has replayed past that point:
SELECT pg_last_wal_replay_lsn() >= '0/3000148'::pg_lsn AS caught_up;
-- if false: fall back to the leader or wait briefly
```
Reading the result: `caught_up = true` means the follower has applied the user's own write, so reading from it cannot hide it. Many applications wrap this check in a proxy or driver.

### 5. Under the hood
- **Linearizability vs serializability**: linearizability is a recency guarantee on a *single object*; serializability is an isolation property on *multi-object transactions*. They are different; strict serializability combines both.
- **Linearizability cost**: needs coordination (consensus/leader) so it has latency cost and unavailability during partitions (the CAP trade-off). Single-leader replication with reads from the leader is linearizable if implemented carefully; asynchronous replicas are not.
- **Linearization point and overlap**: concurrent operations may be ordered either way, but once a read returns the new value all later reads must too; a "compare-and-swap" or lock service uses this.
- **Eventual consistency from the server side** uses tunable consistency: choose consistency level per operation, such as ONE, QUORUM, ALL. Convergence is helped by anti-entropy, read repair and conflict resolution (LWW, vector clocks).
- **Causal consistency** sits between eventual and linearizable: preserves cause→effect order without a global total order and stays available during partitions.
- **Monitoring**: lag can be measured in bytes (LSN difference) or seconds (heartbeat table, `pg_stat_replication.replay_lag`, `Seconds_Behind_Master`).

### 6. Performance and design implications
- Routing reads to the leader for recently written data reduces lag anomalies but reduces read scaling; choose per feature.
- Stronger models cost latency (round trips to quorum or leader) and availability.
- Large transactions, long-running queries on followers (replay conflicts), and slow single-threaded apply all inflate lag.
- Avoid solving lag in application code ad hoc; centralize in the data-access layer or use a database that offers it as a feature (transactions in a single leader solve many such problems).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `pg_stat_replication.replay_lag` | Replica lag time | Monitoring |
| `SHOW REPLICA STATUS` → `Seconds_Behind_Source` | MySQL lag | Monitoring/alerts |
| `synchronous_commit = remote_apply` | Wait until replica applied | Read-your-writes from standby |
| Cassandra consistency levels `QUORUM`, `LOCAL_QUORUM`, `ALL` | Tune r/w | Per-query freshness |
| MongoDB `readConcern: "majority"`, `readPreference` | Read freshness and routing | Avoid stale/rolled back reads |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Eventual consistency: social feeds, counters, catalogues | Financial balances, inventory, uniqueness |
| Read-your-writes + monotonic reads: typical user-facing apps | Strictly cross-user ordering requirements |
| Linearizable: locks, leader election, unique usernames | High-latency geo-distributed hot paths |
| Causal consistency: collaborative/ordered data | Need global total order |

### 9. Common mistakes
- Assuming a follower read reflects a just-committed write → "my data disappeared" bugs → route or check LSN.
- Load-balancing each request randomly across replicas → non-monotonic reads → sticky by user.
- Treating "eventual" as "fast" → hours-long divergence after failures → monitor lag and set alerts.
- Confusing linearizability with serializability.

### 10. 🎯 Interview kit
**30-second answer:** Async followers lag, so reads can be stale. Eventual consistency promises convergence only without new writes. Applications need session guarantees (read-your-writes, monotonic reads, consistent prefix); the strongest single-object model is linearizability, which costs coordination and availability.

**Structure a longer answer:** cause of lag → anomalies and fixes → consistency spectrum → CAP/latency trade-off → measurement.

**Likely follow-ups:**
- *Linearizability vs serializability?* → recency of single object vs isolation of transactions.
- *How do you implement read-your-writes?* → leader reads for recent writes or LSN/timestamp tracking.
- *What is tunable consistency?* → choose r and w per request.

**Signals interviewers look for:** precise model definitions and mapping each anomaly to a concrete remedy.

### 11. Check yourself
- [ ] Describe three lag anomalies and one fix each.
- [ ] Distinguish linearizability from serializability.
- [ ] Explain why strict consistency cannot be implemented.
- [ ] Show how to use an LSN to guarantee read-your-writes.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Problems with Replication Lag › Monotonic Reads
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 9. Consistency and Consensus › Consistency Guarantees
- *Database Internals* — Alex Petrov — Replication and Consistency › Strict Consistency
- *Designing Data-Intensive Applications* — Martin Kleppmann — Problems with Replication Lag › Consistent Prefix Reads
- *Designing Data-Intensive Applications* — Martin Kleppmann — Solutions for Replication Lag
- *Database Internals* — Alex Petrov — Replication and Consistency › Eventual Consistency
- *Designing Data-Intensive Applications* — Martin Kleppmann — Problems with Replication Lag › Reading Your Own Writes
- *Database Internals* — Alex Petrov — Anti-Entropy and Dissemination › Read Repair
- *Database Internals* — Alex Petrov — Replication and Consistency › Witness Replicas
- *Database Internals* — Alex Petrov — Replication and Consistency › Tunable Consistency

