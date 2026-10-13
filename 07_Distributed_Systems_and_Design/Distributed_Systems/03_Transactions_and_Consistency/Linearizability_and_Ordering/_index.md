---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/03_Transactions_and_Consistency/Linearizability_and_Ordering"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Linearizability and Ordering

**L4 Advanced** · [↑ Transactions and Consistency](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Linearizability makes a replicated system behave as if there were a single copy of the data and every operation took effect atomically at one instant — a very strong, very expensive guarantee closely tied to ordering and consensus.

### 1. The big picture
When data is replicated, different clients can see different values for the same key at the same time. *Consistency models* describe what values a read is allowed to return. At the strong end sits **linearizability** (also called atomic, strong, or external consistency): once a write completes, every subsequent read, from any client, sees it. It's the model that makes a distributed store feel like a single variable.

It matters for locks and leader election (everyone must agree who holds the lock), uniqueness constraints (usernames, bank balances), and cross-channel coordination (a file uploaded, then a message sent about it — the worker must see the file). But it is slow and unavailable during partitions. Weaker models — sequential and causal consistency — give up the real-time guarantee but keep most of the intuition, and are far cheaper.

### 2. Core concepts and vocabulary
- **Register** — the simplest object: a single value you can read or write; consistency models are usually defined over registers.
- **Operation interval** — an operation starts (invocation) and ends (response); it may overlap with others.
- **Linearization point** — the instant between invocation and response when the operation appears to take effect.
- **Recency guarantee** — a read after a completed write must return that write (or a newer one).
- **Linearizable** — all operations can be placed on a single timeline, consistent with real time and with a legal sequential history.
- **Sequential consistency** — there is a single global order consistent with each process's own program order, but not necessarily real time.
- **Causal consistency** — operations that are causally related (happens-before) are seen in that order by everyone; concurrent ones may differ.
- **Total order vs partial order** — linearizability implies a total order; causality is only partial.
- **Compare-and-set (CAS)** — atomic "set to new if current equals old"; the basic linearizable primitive.
- **Total order broadcast** — all nodes deliver the same messages in the same order; equivalent to consensus.
- **CAP** — under a network partition you must choose between linearizability and availability.
- **Lamport timestamp** — (counter, node ID) pair giving a total order consistent with causality.

### 3. How it works, step by step
**What linearizable means.**
1. Each operation takes real time: it has a start and an end.
2. The system must pick for each operation one instant inside its interval.
3. All instants together form a legal single-copy history (a read returns the latest write before it).
4. Once any read has returned the new value, no later read may return the old one — the value cannot "flip back".

**Implementing it.**
| Replication scheme | Linearizable? |
|---|---|
| Single-leader, reads from leader or synchronous followers | Potentially (stale if the node isn't really the leader) |
| Consensus (Raft, Paxos, ZooKeeper's Zab) | Yes, with care (reads must go through the leader/quorum or use leases) |
| Multi-leader | No — concurrent writes on different leaders |
| Leaderless with quorums (w + r > n) | Usually not, even with strict quorums (variable delays, sloppy quorums, clock skew) |

**Ordering ladder.**
1. *Linearizability* — total order, real-time.
2. *Sequential consistency* — total order, no real-time; same-process order preserved.
3. *Causal consistency* — partial order; the strongest model that stays available during partitions.
4. *Eventual consistency* — replicas converge eventually; no ordering promised.

### 4. Worked example
Two clients, one register `x = 0`, replicas A (leader) and B (follower):
```
Client 1: write(x,1)  |---------|         completes at t=10
Client 2:                         read(x) |---| at t=12 -> must return 1
Client 3:                    read(x) |----------| overlaps the write -> 0 or 1
                                                     but if it returned 1, no later read may return 0
```
Compare-and-set to implement a lock safely:
```sql
-- in a linearizable store (etcd txn, ZooKeeper, Spanner)
UPDATE locks SET owner = 'worker-7' WHERE name = 'job-42' AND owner IS NULL;
-- exactly one concurrent caller sees "1 row updated"
```
With etcd you'd write `etcdctl txn` with `compare: create_revision("lock") = 0`, which is only safe because the store is linearizable. On an eventually consistent store, two workers could both "win".

### 5. Under the hood
- **Cost is latency, not just partitions.** Even RAM on a multicore CPU isn't linearizable without fences: each core has caches and store buffers. Dropping linearizability there is purely for speed. Attiya and Welch proved that linearizable reads/writes need response time at least proportional to the uncertainty of network delay, so there's no clever algorithm that avoids this.
- **CAP, correctly.** With two data centers and a cut link, single-leader linearizable setups make the follower side unavailable (can't reach the leader); multi-leader remains available but not linearizable. CAP is only about *partitions*; the latency cost applies always.
- **Linearizability is composable (local):** if each object is linearizable, the system of objects is. Operations *spanning* objects still need extra coordination (transactions). **Sequential consistency isn't composable.**
- **Linearizability vs serializability.** Serializability: transactions behave as if in some serial order (isolation, multi-object, no real-time bound). Linearizability: recency on a single object. A database can be one, both (*strict serializability* — Spanner), or neither.
- **Consensus link.** Implementing a linearizable register, a lock, a uniqueness constraint, and total order broadcast are all equivalent to consensus. Total order broadcast + sequence numbers gives a linearizable log; a log-based store can apply writes and check uniqueness deterministically.
- **Lamport timestamps** give a total order consistent with causality but you only know the order *after* seeing all operations, so they can't enforce a uniqueness check in real time — you need total order broadcast.
- **RIFL (retries).** To get exactly-once linearizable RPCs, tag each request with (client ID leased from a service, monotonically increasing sequence) and persist completion records so retried requests return the original result rather than re-executing.
- **Lease-based reads.** A Multi-Paxos/Raft leader may serve reads locally during a time lease; this is a performance optimisation that relies on bounded clock drift.

### 6. Performance and design implications
- Use linearizable operations only where required (locks, leader election, unique IDs, config). Serve other reads from replicas or caches.
- Each linearizable write costs at least one quorum round trip; cross-region this is tens to hundreds of milliseconds.
- Prefer causal consistency (session guarantees: read-your-writes, monotonic reads) for user-facing feeds.
- Test with a linearizability checker (e.g. Jepsen's Knossos) if you build or choose a store.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| etcd `--consistency=l` (default) vs `s` | Linearizable vs serializable (stale) reads | Config reads that can be stale |
| ZooKeeper `sync()` before read | Get a linearizable read from a follower | Rare strong reads |
| MongoDB `readConcern: "linearizable"` | Linearizable reads on primary | Critical single-doc reads |
| Cassandra `SERIAL` / LWT | Paxos-based CAS | Uniqueness checks |
| DynamoDB `ConsistentRead=true` | Strongly consistent read (single-region) | Read-after-write |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Locks, leader election, unique constraints, fencing tokens | High-volume reads of tolerant data |
| Causal consistency: social feeds, collaborative apps | You must enforce a cross-user invariant |
| Eventual consistency: counters, caches, analytics | Users must see their own writes immediately |

### 9. Common mistakes
- Equating "quorum reads and writes" with linearizable → stale reads → use consensus-backed operations or read-repair-with-care.
- Mixing up linearizability and serializability → wrong guarantees assumed → check recency vs isolation separately.
- Reading from a former leader → stale data → leases, or a quorum read.
- Using wall-clock timestamps for ordering → lost updates by skew → logical clocks.

### 10. 🎯 Interview kit
**30-second answer:** Linearizability means the system acts like one copy of the data: each operation appears atomic at some point between its start and end, and after a write completes everyone reads it. It needs coordination (consensus), costs latency, and is unavailable under partitions; weaker models like causal consistency avoid that.

**Structure a longer answer:** define via single-copy behaviour → example with two clients → where it's needed (locks, uniqueness) → cost/CAP → weaker alternatives.

**Likely follow-ups:**
- *Linearizable vs serializable?* → Recency on one object vs isolation across many.
- *Sequential vs linearizable?* → No real-time order; not composable.
- *Why aren't quorums enough?* → Variable delays and concurrent writes allow stale reads.
- *Is linearizability needed for causality?* → No; causal consistency is the strongest available model.

**Signals interviewers look for:** correct distinction from serializability, mention of consensus equivalence, and ability to pick the weakest sufficient model.

### 11. Check yourself
- [ ] Draw a timeline where a read overlaps a write and list the allowed results.
- [ ] Explain why multi-leader and leaderless replication aren't linearizable.
- [ ] State the relationship among linearizability, causal consistency and sequential consistency.
- [ ] Explain why implementing a unique-username check requires consensus.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — J › L
- *Designing Data-Intensive Applications* — Martin Kleppmann — Scope of ordering guarantee › Implementing total order broadcast using linearizable storage
- *Designing Data-Intensive Applications* — Martin Kleppmann — Ordering Guarantees › Linearizability is stronger than causal consistency
- *Designing Data-Intensive Applications* — Martin Kleppmann — Scope of ordering guarantee › Implementing linearizable storage using total order broadcast
- *Database Internals* — Alex Petrov — Replication and Consistency › Cost of linearizability
- *Designing Data-Intensive Applications* — Martin Kleppmann — The Cost of Linearizability
- *Database Internals* — Alex Petrov — Replication and Consistency › Sequential Consistency
- *Designing Data-Intensive Applications* — Martin Kleppmann — Ordering Guarantees
- *Designing Data-Intensive Applications* — Martin Kleppmann — Sequence Number Ordering › Lamport timestamps
- *Designing Data-Intensive Applications* — Martin Kleppmann — Sequence Number Ordering › Timestamp ordering is not sufficient

