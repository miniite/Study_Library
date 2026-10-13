---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/02_Replication_and_Partitioning/Leaderless_Replication"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Leaderless Replication and Quorums

**L4 Advanced** · [↑ Replication and Partitioning](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Leaderless (Dynamo-style) replication lets any replica accept writes; clients or coordinators send each write to several replicas and read from several, and quorum rules (w + r > n) plus read repair and anti-entropy make the copies converge.

### 1. The big picture
Single-leader and multi-leader designs rely on one node (or one per region) to order writes. Leaderless systems drop that idea: there is no failover because there is no leader. Amazon's Dynamo made the style fashionable; Cassandra, Riak and Voldemort follow it. A client (or a coordinator on its behalf) sends a write to all *n* replicas in parallel and considers it successful once *w* acknowledge; reads query *r* replicas and take the newest value. Availability is excellent: a node can be rebooted with no failover event. The price is weaker, subtler consistency, since there is no single order of writes.

### 2. Core concepts and vocabulary
- **n** — number of replicas holding each key; **w** — acknowledgements required for a write; **r** — replicas consulted for a read.
- **Quorum condition** — `w + r > n`: read and write sets overlap, so a read sees at least one up-to-date node.
- **Version number / timestamp / vector clock** — used to decide which value is newer.
- **Read repair** — a reader writes the newer value back to stale replicas it noticed.
- **Anti-entropy process** — background comparison and copying between replicas.
- **Sloppy quorum** — writes go to any reachable nodes, not necessarily the designated *n*, with **hinted handoff** to return data later.
- **Last-write-wins (LWW)** — conflict resolution by timestamp, discarding losers.

### 3. How it works, step by step
**Write with a node down** (n=3, w=2):
1. Client sends write to all three replicas in parallel.
2. Two accept, one is down and misses it.
3. After two *ok* replies the write is successful; the client ignores the third.

**Read after the node returns** (r=2):
1. Client queries replicas in parallel.
2. Replies carry version numbers: say v7 from two replicas, v6 from the returned one.
3. Client takes v7 and performs **read repair** by writing v7 to the stale replica.
4. Rarely read keys get fixed later by the anti-entropy process.

### 4. Worked example
```text
n=5, w=3, r=3  (w + r = 6 > 5)  -> tolerates 2 unavailable nodes
n=3, w=3, r=1                   -> fast reads, but one down node blocks all writes
n=3, w=1, r=1  (w + r = 2 <= 3) -> fastest and most available, but stale reads likely

CQL (Cassandra) example:
CONSISTENCY QUORUM;
INSERT INTO users (id, name) VALUES (1234, 'Ada');   -- w = QUORUM
SELECT name FROM users WHERE id = 1234;              -- r = QUORUM
```
Reading it: with RF=3, QUORUM is 2 for both, so the sets always overlap and the SELECT returns the newest acknowledged write. At `ONE`/`ONE` the read may hit the lagging node.

### 5. Under the hood
- **Why overlap works**: any set of w nodes and any set of r nodes with w + r > n share at least one node, which holds the latest acknowledged write.
- **Limits of quorum consistency**: even with w + r > n stale reads can occur when (a) a sloppy quorum puts writes on different nodes than reads query; (b) concurrent writes are resolved by timestamps (LWW) and clocks skew, silently losing a write; (c) a write succeeded on fewer than w replicas, fails, but is not rolled back on those where it landed; (d) a node with a new value is restored from a replica with an old one, dropping below w; (e) unlucky timing between a read and a write in flight. Quorums therefore give "probably fresh", not linearizability.
- **Monitoring**: leaderless stores usually have no replication-lag metric because there is no ordered log; staleness is probabilistic and can only be estimated.
- **Sloppy quorums and hinted handoff** improve write availability during partitions but offer no overlap guarantee until hints replay.
- **Detecting concurrent writes** needs version vectors or CRDT-style merges; LWW is simple but lossy.
- **Multi-datacenter**: n includes replicas in all DCs; local-quorum variants keep latency low by waiting only for local replicas.

### 6. Performance and design implications
- Latency is the slowest of the *w* (or *r*) fastest replies; higher w/r raises tail latency (see percentiles), lower values raise staleness.
- Failures cost nothing visible until fewer than w (or r) nodes are reachable.
- Many replicas answering every read costs bandwidth; digest reads reduce it.
- Anti-entropy and hinted handoff add background load; plan repair windows.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CONSISTENCY ONE/QUORUM/LOCAL_QUORUM/ALL` (cqlsh) | Sets r/w per request | Per-query trade-off |
| Replication factor (keyspace) | Sets *n* | Durability planning |
| `nodetool repair` | Anti-entropy | Regular maintenance |
| Riak `n_val`, `r`, `w`, `dw` | Quorum parameters | Dynamo-style config |
| Lightweight transactions (`IF NOT EXISTS`) | Compare-and-set via Paxos | When you need linearizable updates |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Always-on availability, multi-DC writes | You need strict serializability or multi-key transactions |
| Workload tolerates eventual consistency (counters, sessions, feeds) | Application cannot handle conflicting versions |
| `w + r > n` with majorities | Low-latency demands where r=w=1 stale reads are acceptable (then be explicit) |
| LWW for simplicity | Data where lost updates matter (use CRDTs or app merge) |

### 9. Common mistakes
- Assuming `w + r > n` means linearizable → anomalies still possible → use CAS/consensus where needed.
- Mixing sloppy quorums with a consistency expectation → stale reads → avoid or accept.
- Relying on wall-clock timestamps for LWW → clock skew drops writes → use logical clocks or single-writer keys.
- Skipping repair → deleted data resurrects → schedule anti-entropy.

### 10. 🎯 Interview kit
**30-second answer:** Leaderless replication writes to n replicas and succeeds after w acks; reads ask r replicas and pick the newest version. With w + r > n the read and write sets overlap. Read repair and anti-entropy converge stragglers; the design favours availability over strong consistency.

**Structure a longer answer:** compare with leader models → write/read path → quorum math → repair mechanisms → failure/edge cases → when to choose.

**Likely follow-ups:**
- *Is quorum read strongly consistent?* → no; sloppy quorums, concurrent writes, clock skew and partial failures allow anomalies.
- *What values of w and r would you choose for read-heavy loads?* → w=n, r=1, accepting write unavailability on a single failure; or quorum both.
- *How do you resolve conflicts?* → LWW, version vectors with app merge, or CRDTs.

**Signals interviewers look for:** quorum arithmetic fluency, knowledge of edge cases, awareness of conflict resolution cost.

### 11. Check yourself
- [ ] For n=5 choose w and r tolerating two failures and explain.
- [ ] Walk through read repair with versions 6 and 7.
- [ ] Give three scenarios where w + r > n still reads stale data.
- [ ] Explain the role of hinted handoff and its consistency cost.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Multi-Leader Replication Topologies › Read repair and anti-entropy
- *Designing Data-Intensive Applications* — Martin Kleppmann — Detecting Concurrent Writes
- *Designing Data-Intensive Applications* — Martin Kleppmann — Multi-Leader Replication Topologies › Leaderless Replication
- *Designing Data-Intensive Applications* — Martin Kleppmann — Multi-Leader Replication Topologies › Quorums for reading and writing
- *Designing Data-Intensive Applications* — Martin Kleppmann — Limitations of Quorum Consistency › Sloppy Quorums and Hinted Handoff
- *Designing Data-Intensive Applications* — Martin Kleppmann — Multi-Leader Replication Topologies › Writing to the Database When a Node Is Down
- *Designing Data-Intensive Applications* — Martin Kleppmann — J › L
- *Designing Data-Intensive Applications* — Martin Kleppmann — Limitations of Quorum Consistency
- *Designing Data-Intensive Applications* — Martin Kleppmann — Detecting Concurrent Writes › Version vectors
- *Hadoop: The Definitive Guide* — Tom White — ZooKeeper › Warning

