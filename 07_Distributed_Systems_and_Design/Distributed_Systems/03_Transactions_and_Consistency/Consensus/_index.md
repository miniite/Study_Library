---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/03_Transactions_and_Consistency/Consensus"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Consensus Algorithms

**L4 Advanced** · [↑ Transactions and Consistency](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Consensus lets a group of nodes agree on a single value (or an ordered log of values) despite crashes and message loss, as long as a majority stays reachable — Paxos and Raft are the canonical algorithms.

### 1. The big picture
Many hard distributed problems reduce to "get everyone to agree": who is the leader, did this transaction commit, what is the next entry in the log, who owns the lock. Solving this ad hoc leads to split brain and lost data. **Consensus** gives a formal solution: a set of processes each *propose* values, and the algorithm guarantees they *decide* on one.

The ingredients are always the same: a **leader** to sequence decisions, **quorums** (majorities) so any two decisions overlap in at least one node, **numbered rounds** (terms/epochs/ballots) so stale leaders are recognised, and a **durable log**. Real systems use consensus underneath: etcd and Consul (Raft), ZooKeeper (Zab), Spanner/Chubby (Paxos), Kafka's KRaft, CockroachDB (Raft). The related topics are Leader Election, Distributed Transactions and Linearizability.

### 2. Core concepts and vocabulary
- **Consensus properties** — *agreement* (no two nodes decide differently), *validity* (the decided value was proposed), *termination/liveness* (non-faulty nodes eventually decide), *integrity* (decide at most once).
- **FLP result** — in a purely asynchronous system, no deterministic algorithm can guarantee termination if even one node may crash; practical systems use timeouts/randomness to get liveness.
- **Quorum** — a majority (⌊n/2⌋ + 1); a cluster of 2f+1 tolerates f crash failures.
- **Proposer / acceptor / learner** — Paxos roles: suggest values, vote, and store outcomes.
- **Proposal number** — unique, increasing ballot (often (counter, node ID)).
- **Multi-Paxos** — Paxos with a stable *distinguished proposer* (leader) that skips the prepare phase for subsequent decisions.
- **Replicated log / state machine** — each node applies the same commands in the same order, getting the same state.
- **Term (Raft)** — a numbered period with at most one leader.
- **Candidate / leader / follower** — Raft roles.
- **Lease** — time-bounded promise that no other leader will be accepted; used to serve fast reads.
- **Byzantine fault** — node behaves arbitrarily or maliciously; requires 3f+1 nodes (PBFT).

### 3. How it works, step by step
**Classic Paxos (single value)**
1. *Prepare*: a proposer picks a number *n* and sends `Prepare(n)` to acceptors.
2. An acceptor that hasn't promised a higher number replies `Promise(n)`, including any value it already accepted. It promises to ignore lower-numbered proposals.
3. With promises from a majority, the proposer picks the value from the highest-numbered accepted proposal reported, or its own value if none.
4. *Accept*: it sends `Accept(n, v)`; acceptors accept unless they promised a higher number.
5. With a majority of accepts, *v* is **chosen**; learners are told.

**Multi-Paxos** elects a stable leader once; subsequent log slots need only the accept round (one round trip). Leader changes re-run prepare for all open slots.

**Raft**
1. All nodes start as followers; if a follower hears nothing from a leader for a *randomised election timeout* it becomes a candidate, increments its term, votes for itself and requests votes.
2. Each node votes for at most one candidate per term, and only if the candidate's log is at least as up to date (higher last term, or same term and longer log). A majority wins; a split vote just times out and retries.
3. The leader receives client commands, appends to its log, and sends `AppendEntries` to followers.
4. When a majority has persisted an entry, the leader marks it **committed** (also committing all earlier entries), applies it, and tells followers in later messages.
5. Followers that diverge are overwritten to match the leader's log.

### 4. Worked example
A 3-node Raft cluster {A, B, C}, A is leader in term 4, client sends `x = 8`:
```
A: log [..., (t4,i7: x=8)]  -> AppendEntries to B, C
B: persists, acks            -> majority (A+B) reached
A: commits i7, applies x=8, replies to client
C: partitioned, never saw i7

A crashes. B and C time out. C's log is shorter (no i7).
C requests B's vote in term 5 -> B denies (C's log is behind).
B becomes candidate, C votes yes -> B leader term 5. Entry i7 survives.
```
Safe operations you can try:
```bash
# etcd cluster health and leader
etcdctl endpoint status --cluster -w table
# a linearizable write/read through the Raft log
etcdctl put x 8
etcdctl get x --consistency=l
```
`endpoint status` shows `IS LEADER`, `RAFT TERM`, `RAFT INDEX`; if the term climbs rapidly the cluster is flapping elections.

### 5. Under the hood
- **Why quorums work:** any two majorities intersect, so a later leader always sees every committed entry in at least one voter. That is why a candidate with an out-of-date log can't win in Raft.
- **Safety vs liveness:** safety (never two different decisions) holds always; liveness needs a majority reachable and some stable leader. Dueling proposers in Paxos can livelock; Raft's randomised timeouts and Multi-Paxos's leader address this.
- **Persistence rules:** acceptors must durably record promises and accepted values *before* replying; Raft persists `currentTerm`, `votedFor` and log entries before answering.
- **Reads:** a naive read from the leader can be stale if a new leader exists. Solutions: send reads through the log, confirm leadership with a quorum heartbeat (ReadIndex), or use leases bounded by clock drift (a performance optimisation that depends on clocks).
- **Log compaction:** applied prefix is replaced by a snapshot, the log truncated, and slow followers catch up by installing snapshots.
- **Membership changes** must keep majorities overlapping (joint consensus or single-server changes).
- **Generalised view (write-once registers):** a client writes only into write-once registers in rounds; phase 1 ensures safety by sealing older registers with *nil* and discovering values; phase 2 writes. This describes the same safety argument without roles.
- **Byzantine consensus** (PBFT, blockchains) tolerates lying nodes but needs 3f+1 replicas and more message rounds; only used across mutually distrusting parties.
- **Fast Paxos / EPaxos** cut a round trip when there are no conflicts at the cost of larger quorums.

### 6. Performance and design implications
- Latency of a commit ≈ one round trip to a majority (plus disk fsync). Place replicas to keep the nearest majority close.
- Throughput is bounded by the leader; batching and pipelining entries are the main optimisations.
- Use odd cluster sizes (3 or 5); 7+ increases latency without much extra safety. Never split across just two failure domains.
- Monitor: leader changes, commit latency, fsync latency, follower lag, term number.
- Clock-skew-sensitive leases must have conservative bounds.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `etcdctl endpoint status/health` | Show leader, term, index | Debugging elections |
| Raft `election-timeout`, `heartbeat-interval` (etcd) | Failure detection speed | Tune for WAN/slow disks |
| ZooKeeper `tickTime`, `initLimit`, `syncLimit` | Timeouts for ZAB | Large ensembles |
| `snapshot-count` (etcd) | When to compact the log | Memory/disk tuning |
| Kafka `controller.quorum.voters` (KRaft) | Raft quorum members | Metadata quorum |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Raft: you want an understandable, well-documented algorithm | Need extreme throughput across many leaders — shard it |
| Multi-Paxos / Paxos variants: existing library or leaderless fast paths | Implementing it from scratch (subtle) |
| Byzantine consensus: adversarial, multi-organisation | Single trusted operator — adds cost for no benefit |
| Use consensus service (etcd/ZooKeeper) for coordination | Putting bulk data through it — keep only metadata |

### 9. Common mistakes
- Even-sized clusters → no extra fault tolerance → use 3, 5.
- Spanning two data centers evenly → loses majority if one DC fails → three sites or place majority in one.
- Too-short election timeouts → constant elections under GC pauses → tune above network + fsync jitter.
- Treating lease reads as unconditionally safe → stale reads with clock drift → bound drift or use ReadIndex.
- Storing large values in etcd/ZooKeeper → leader overload.

### 10. 🎯 Interview kit
**30-second answer:** Consensus is getting a cluster to agree on a value despite crashes, using majority quorums so that any two decisions overlap. Paxos does it with prepare/accept phases and numbered proposals; Raft makes leadership central, with terms, leader election, and a replicated log committed once a majority has it.

**Structure a longer answer:** problem and properties → quorum intersection → Paxos phases or Raft flow → failure handling (leader crash, partition) → cost and cluster sizing.

**Likely follow-ups:**
- *Why odd cluster sizes?* → n=4 tolerates 1 failure, same as n=3, but needs a larger majority.
- *What stops a stale leader from committing?* → Higher term; followers reject its AppendEntries and it can't reach a majority.
- *Difference between Paxos and Raft?* → Raft specifies leader election and log management; Paxos is a core agreement protocol with more freedom (and ambiguity).
- *What does FLP say?* → No guaranteed termination in fully asynchronous systems; practical systems rely on timeouts.

**Signals interviewers look for:** quorum-intersection reasoning, correct handling of safety vs liveness, awareness of stale-read pitfalls.

### 11. Check yourself
- [ ] State consensus properties and explain FLP in one sentence.
- [ ] Walk through a Raft election and log replication with a leader crash.
- [ ] Explain why a candidate with a shorter log can't be elected.
- [ ] Describe two ways to make leader reads linearizable.
- [ ] Say how many failures a 5-node cluster tolerates and why.
<!-- explainer:end -->

## In this section (study in order)

1. [Leader Election](<Leader_Election/_index.md>) (L5 Expert)

## References

- *Database Internals* — Alex Petrov — Raft
- *Designing Data-Intensive Applications* — Martin Kleppmann — Fault-Tolerant Consensus › Consensus algorithms and total order broadcast
- *Database Internals* — Alex Petrov — Paxos
- *Designing Data-Intensive Applications* — Martin Kleppmann — Distributed Transactions and Consensus
- *Database Internals* — Alex Petrov — Fast Paxos › Generalized Paxos algorithm
- *Database Internals* — Alex Petrov — Fast Paxos › Generalized Solution to Consensus
- *Designing Data-Intensive Applications* — Martin Kleppmann — Scope of ordering guarantee › Using total order broadcast
- *Designing Data-Intensive Applications* — Martin Kleppmann — U › V
- *Database Internals* — Alex Petrov — Paxos › Multi-Paxos
- *Database Internals* — Alex Petrov — Raft › Byzantine Consensus

