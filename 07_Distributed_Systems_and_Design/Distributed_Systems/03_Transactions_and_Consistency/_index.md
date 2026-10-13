---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/03_Transactions_and_Consistency"
level: 3
type: "curriculum"
curriculum: "data_engineering"
---

# Transactions and Consistency

**L3 Practitioner** · [↑ Distributed Systems](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Replication makes data durable and fast to read, but multiple copies force you to choose what a reader may observe (a consistency model) and how multi-object changes become atomic (transactions, atomic commit, consensus) — each stronger guarantee costs latency, availability or both.

### 1. The big picture
Once data lives on several machines, three questions appear in every design:

1. **What may a reader see?** If two clients read and write different copies at the same time, which histories are acceptable? That contract is a *consistency model*.
2. **How do several changes happen as one?** Moving money between accounts touches two records, possibly on two servers. That is a *transaction*; across servers it needs *atomic commit*.
3. **How do nodes agree on anything at all?** Who is the leader, which transaction commits first, who got the last seat? That is *consensus*.

This block's children go through them in order: ACID and isolation (single-node transactions, snapshot isolation/MVCC, write skew), serializability, linearizability and ordering, distributed transactions (2PC and friends), and consensus with leader election. This parent lesson is the map: how the pieces connect and when each guarantee is worth paying for. It builds on the Distributed Failures block — consensus is exactly the tool that turns unreliable failure detection into a reliable decision.

### 2. Core concepts and vocabulary
- **Replication** — keeping several copies of data for fault tolerance, read scaling and locality (geo-replication reduces latency and survives datacenter loss).
- **Consistency model** — a contract that says which values reads may return given concurrent operations on multiple copies. Weaker models allow more histories and need less coordination.
- **Single-operation vs multi-operation models** — linearizability/sequential/causal/eventual describe individual reads and writes; transaction isolation levels describe groups of operations.
- **Linearizability** — every operation appears to take effect atomically at one instant between its start and end; the system behaves like a single copy with real-time ordering ("strong consistency", also "external consistency" when applied to transactions).
- **Serializability** — concurrent transactions behave as if executed one after another in some order; no real-time constraint.
- **Eventual consistency** — if writes stop, replicas converge; reads may meanwhile be stale.
- **Causal consistency** — operations that are causally related are seen in order by everyone.
- **Transaction / ACID** — atomic group of operations; atomicity, consistency (application invariants), isolation, durability.
- **Atomic commit** — all participants commit or all abort.
- **Two-phase commit (2PC)** — coordinator asks participants to prepare, then commits; blocks if the coordinator dies at the wrong moment.
- **Consensus** — getting nodes to agree on one value: *uniform agreement*, *integrity* (decide once), *validity* (value was proposed), *termination* (non-crashed nodes decide).
- **Total order broadcast** — all nodes deliver the same messages in the same order; equivalent to repeated consensus.
- **Timeliness vs integrity** — seeing up-to-date data vs not being corrupt.
- **Compensating transaction** — a later action that corrects a temporary violation (an apology workflow).
- **CAP/PACELC** — under partition choose consistency or availability; otherwise latency versus consistency.

### 3. How it works, step by step
**A ladder of guarantees (stronger is costlier)**
| Level | What you get | Typical mechanism | Cost |
|---|---|---|---|
| Eventual | Replicas converge when writes stop | Async replication, anti-entropy, CRDTs | Cheapest; stale and out-of-order reads |
| Read-your-writes / monotonic reads / session | Sensible per-client behavior | Sticky routing, version tracking | Low |
| Causal | Causally related ops ordered | Version vectors / dependency tracking | Moderate |
| Snapshot isolation (txn) | Consistent point-in-time view; no dirty/non-repeatable reads; write skew possible | MVCC | Moderate |
| Serializable (txn) | As-if-serial transactions | 2PL, SSI, actual serial execution | Higher |
| Linearizable | Single-copy illusion with real-time order | Consensus / leader with quorum | Latency + unavailability under partition |
| Strict serializable / external consistency | Serializable *and* real-time order | Spanner: Paxos + 2PC + TrueTime | Highest |

**How a multi-partition transaction commits (the 2PC shape)**
1. Client sends operations to a coordinator; participants execute and take locks.
2. **Prepare:** coordinator asks every participant "can you commit?"; each durably logs a promise and votes yes/no.
3. **Decide:** if all vote yes, the coordinator logs *commit* then tells everyone; any no (or timeout) → abort.
4. Participants apply or roll back and release locks.
5. If the coordinator crashes after prepare, participants are *in doubt* and hold locks until it returns — the blocking problem. Fixes: replicate the coordinator's state with consensus (Spanner runs 2PC over Paxos groups), or avoid 2PC by ordering transactions deterministically (Calvin).

**How consensus unblocks it**
1. Nodes need an agreed sequence of decisions (the log).
2. A leader proposes; followers accept; a majority acknowledgment commits the entry.
3. If the leader is suspected dead (via a failure detector), a new leader is elected with a higher term/ballot; majority intersection guarantees it learns every committed value.
4. FLP says deterministic consensus can't be guaranteed in a purely asynchronous system, so algorithms (Paxos, Raft, ZAB, Viewstamped Replication) keep **safety** always and get **liveness** from timeouts/failure detection when the system is partially synchronous.

### 4. Worked example
An order service writes to two stores. Choose guarantees per operation instead of "one strong setting for everything":

```sql
-- Inside one database: serializable transaction for the invariant
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
  UPDATE inventory SET qty = qty - 1 WHERE sku = 'A17' AND qty > 0;
  INSERT INTO orders (order_id, sku, user_id) VALUES (...);
COMMIT;
```

```text
-- Across services/partitions: avoid 2PC where possible
order-service  --(event: OrderPlaced, key=order_id)-->  log/queue
payment-service consumes, charges idempotently (key=order_id)
shipping-service consumes, creates shipment idempotently
failure -> retry; if payment declined -> compensating event: OrderCancelled
```

Reasoning: the inventory/order invariant (never oversell) is local to one database, so a serializable transaction is cheap and correct. The cross-service workflow uses an ordered log with idempotent consumers and compensation, which gives integrity without a distributed lock across services; users may briefly see "processing" (a timeliness violation, not an integrity one).

A database-level consensus example: with `etcd`/ZooKeeper you can build a lock/leader-election primitive; reads from followers may be stale unless you ask for a linearizable (quorum) read:

```bash
etcdctl get /config/db --consistency=l     # linearizable (default)
etcdctl get /config/db --consistency=s     # serializable/local read, may be stale, faster
```

### 5. Under the hood
- **Replication and consensus are siblings.** Updating several copies atomically *is* the consensus problem, so strongly consistent replication always pays for coordination. That is why systems expose weaker models: they permit divergence in exchange for latency and availability.
- **State vs operation view.** One can describe consistency by allowed *states* of copies, or by constraints on the *order of operations* seen from outside; operation-based models are what clients can observe.
- **No global clock → no natural order.** Every node has its own view of time, "special relativity for data". Models recover order with leaders, logical clocks, or bounded-uncertainty clocks (TrueTime with commit-wait gives external consistency, equivalent in real-time terms to linearizability: if T1 commits before T2 starts, T1's timestamp is smaller).
- **Spanner vs Calvin.** Spanner: Paxos for log replication, 2PC across shards, TrueTime for ordering; multi-partition transactions cost an extra 2PC round. Calvin: agree on a deterministic transaction order first (a sequencing layer + consensus), then execute without 2PC.
- **Uniqueness requires consensus.** Enforcing a traditional uniqueness constraint means funneling all conflicting writes through one deciding node/partition. Many applications can instead accept temporary violation plus an apology (overbooking, overdrafts, oversold stock), which removes the coordination cost.
- **Timeliness vs integrity.** "Eventual consistency" violates timeliness (fixes itself by waiting); integrity violations (lost or corrupt data, wrong derived state) are permanent and need detection and repair. Atomicity, durability, idempotent processing and end-to-end checks protect integrity even when timeliness is relaxed.
- **Consensus properties:** agreement and integrity and validity are safety; termination is liveness. 2PC fails termination because it waits for a crashed coordinator; consensus algorithms need a majority, so they tolerate *f* failures with 2*f*+1 nodes.

### 6. Performance and design implications
- Each cross-node coordination step adds a network round trip; geo-distributed strong consistency commonly means tens to hundreds of milliseconds per commit.
- Linearizable reads must go through the leader or a quorum — they don't scale out like follower reads. Offer relaxed read options (bounded staleness, session consistency) for read-heavy paths.
- Majority quorums protect safety but reduce availability: the minority side of a partition cannot make progress.
- Keep transactions short and local to a partition; design keys so that the common transaction touches one shard.
- Prefer idempotent, ordered logs + compensation (sagas) for long business processes across services; reserve 2PC/consensus for small, critical invariants.
- Monitor replication lag, leader changes and commit latency; they are the operational signs of the consistency/availability trade-off.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SET TRANSACTION ISOLATION LEVEL ...` | Chooses isolation for a session | Match anomalies you can tolerate |
| `ConsistencyLevel` (Cassandra: `ONE`, `QUORUM`, `ALL`, `LOCAL_QUORUM`) | Per-request replica count | Tunable consistency (R+W>N for strong) |
| `acks=all`, `min.insync.replicas` (Kafka) | Wait for in-sync replicas | Durability vs latency |
| `readConcern` / `writeConcern` (MongoDB) | Visibility and durability level | Choose majority for safe reads/writes |
| etcd/ZooKeeper linearizable vs stale reads | Quorum read vs local read | Coordination data vs performance |
| 2PC / XA (`BEGIN DISTRIBUTED TRANSACTION` in SQL Server MSDTC) | Cross-resource atomic commit | Small, critical multi-resource updates |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Serializable/linearizable — money, inventory, uniqueness, leader election | High-volume, latency-critical reads where staleness is harmless |
| Snapshot isolation — most OLTP | Invariants spanning rows that SI can't protect (write skew) |
| Eventual/causal — feeds, counters, caches, analytics ingestion | Anything needing read-your-writes without session handling |
| 2PC — few participants, short transactions, fast reliable network | Long transactions, many services, third-party participants |
| Sagas / compensation — long-running business workflows | You need isolation between steps (intermediate states are visible) |
| Consensus-backed metadata store (etcd, ZK) | Putting bulk data through consensus (it's low-throughput) |

### 9. Common mistakes
- Assuming "ACID" or "consistent" means the same everywhere → surprises across engines → read the exact isolation and consistency definitions.
- Using a distributed transaction across microservices by default → lock holding, coordinator blocking, availability loss → events + idempotency + compensation.
- Reading from followers right after writing → stale or missing data → read-your-writes routing or leader reads.
- Believing quorum reads/writes are always linearizable → sloppy quorums, clock skew, concurrent writes break it → verify the system's actual guarantee.
- Confusing serializable with linearizable → real-time order issues → know the difference (transaction isolation vs single-object recency).
- Ignoring that timeliness and integrity are separate → "eventual consistency" used as an excuse for lost updates → design integrity checks and idempotence.

### 10. 🎯 Interview kit
**30-second answer:** Replication gives availability and speed but forces a choice about what readers may see. Consistency models run from eventual to linearizable, transactions group operations atomically (snapshot isolation to serializable), and spanning partitions needs atomic commit — usually 2PC, made fault-tolerant by running it over consensus as Spanner does. Stronger guarantees cost latency and availability, so pick per operation.
**Structure a longer answer:** clarify invariants and failure assumptions → consistency model for reads → isolation level for transactions → cross-partition coordination (2PC vs log/sagas/deterministic ordering) → consensus for leader/metadata → state the trade-off (latency, availability under partition) with a concrete scenario.
**Likely follow-ups:**
- *Serializable vs linearizable?* → multi-object isolation vs single-object real-time recency; strict serializable is both.
- *Why does 2PC block?* → coordinator crash after prepare leaves participants in doubt.
- *What does FLP say, and how do real systems get around it?* → no guaranteed termination in pure async; use timeouts/failure detectors for liveness, keep safety unconditional.
- *When can you relax uniqueness/consistency?* → when an apology/compensation is cheap and bounded.
- *How does Spanner order transactions?* → TrueTime intervals, commit wait, Paxos groups, 2PC.
**Signals interviewers look for:** you tie each guarantee to a concrete cost, distinguish timeliness from integrity, and choose weaker, cheaper mechanisms where the business allows.

### 11. Check yourself
- [ ] Rank eventual, causal, snapshot isolation, serializable and linearizable by strength and give one cost for each.
- [ ] Explain why atomically updating replicas is equivalent to consensus.
- [ ] Walk through 2PC and show exactly where it blocks.
- [ ] State the four consensus properties and classify them as safety or liveness.
- [ ] Explain how Spanner achieves external consistency.
- [ ] Give an example where a compensating transaction is better than a distributed lock.
<!-- explainer:end -->

## In this section (study in order)

1. [ACID and Weak Isolation Levels](<ACID_and_Isolation/_index.md>) (L4 Advanced)
2. [Serializability Techniques](<Serializability/_index.md>) (L4 Advanced)
3. [Linearizability and Ordering](<Linearizability_and_Ordering/_index.md>) (L4 Advanced)
4. [Distributed Transactions and Two-Phase Commit](<Distributed_Transactions/_index.md>) (L4 Advanced)
5. [Consensus Algorithms](<Consensus/_index.md>) (L4 Advanced)

## References

- *Database Internals* — Alex Petrov — Replication and Consistency › Chapter 11. Replication and Consistency
- *Database Internals* — Alex Petrov — Distributed Transactions › Chapter 13. Distributed Transactions
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Eventual Versus Strong Consistency
- *Designing Data-Intensive Applications* — Martin Kleppmann — Distributed Transactions and Consensus
- *Designing Data-Intensive Applications* — Martin Kleppmann — J › L
- *Database Internals* — Alex Petrov — Replication and Consistency › Consistency Models
- *Database Internals* — Alex Petrov — Further Reading › Appendix A. Bibliography
- *Designing Data-Intensive Applications* — Martin Kleppmann — Fault-Tolerant Consensus
- *Database Internals* — Alex Petrov — Distributed Transactions › Distributed Transactions with Spanner
- *Designing Data-Intensive Applications* — Martin Kleppmann — Enforcing Constraints › Timeliness and Integrity

