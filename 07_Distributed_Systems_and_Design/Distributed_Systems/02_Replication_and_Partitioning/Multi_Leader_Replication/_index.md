---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/02_Replication_and_Partitioning/Multi_Leader_Replication"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Multi-Leader Replication

**L4 Advanced** · [↑ Replication and Partitioning](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Multi-leader replication lets several nodes accept writes (typically one per datacenter or per device) and replicate them to each other asynchronously, buying local write latency and offline tolerance at the price of write conflicts you must detect and resolve.

### 1. The big picture
With a single leader, every write funnels through one node; if it is in another continent, every write pays that latency, and if the link between datacenters breaks, writes stall. Multi-leader (master-master, active-active) replication gives each datacenter its own leader. Each leader acts as a follower to the others: a write is committed locally, then shipped asynchronously across the wide-area link.

Inside a datacenter normal single-leader replication still applies. Multi-leader is mostly a *retrofitted* feature in databases, so it interacts badly with auto-increment keys, triggers and integrity constraints; it is widely regarded as dangerous territory to use only when the benefits are needed.

### 2. Core concepts and vocabulary
- **Multi-leader / active-active** — more than one node accepts writes.
- **Write conflict** — two leaders concurrently modify the same record and both succeed locally.
- **Conflict avoidance** — routing all writes for a record through the same leader.
- **Last write wins (LWW)** — resolve by highest timestamp or ID; loses data.
- **Convergence** — all replicas must end with the same value regardless of arrival order.
- **Replication topology** — which leaders send to which: circular, star, all-to-all.
- **CRDT / operational transformation** — data structures/algorithms that merge concurrent edits automatically.
- **Offline-first client** — each device keeps a local leader and syncs later.

### 3. How it works, step by step
Use cases:
1. **Multi-datacenter**: one leader per DC; clients write locally, DCs sync asynchronously. Benefits: lower write latency, tolerance of whole-DC outage and of inter-DC network problems.
2. **Offline clients** (calendar apps): every device is a leader with a local database; sync may lag hours or days (CouchDB is built for this).
3. **Collaborative editing** (shared documents): each user's edit applies locally and is merged with others'.

Write path and conflict:
1. User 1 renames wiki page A→B at leader 1; user 2 renames A→C at leader 2. Both commit.
2. Asynchronous replication delivers each change to the other leader.
3. Each leader now sees a conflicting update and must resolve it deterministically so all replicas converge.

### 4. Worked example
```text
Conflict strategies
1. Avoid:  route every user to their "home" leader (user_id -> region). No concurrent writers per record.
2. LWW:    keep the version with the largest timestamp.        (simple, loses writes)
3. Merge:  concatenate values  "B/C"  or union sets/counters.  (no loss, app-specific)
4. Record: keep all siblings, ask app/user to resolve later.   (Riak/CouchDB style)
```
Illustration: calendar event title edited offline on phone ("Lunch") and on laptop ("Team lunch"). LWW silently drops one; sibling retention shows both so the user picks. Prefer avoidance where possible: for per-user data, pin the user's writes to one region and the problem disappears except during failover.

### 5. Under the hood
- **Detection is asynchronous**: unlike single-leader, where the second writer blocks or aborts, here both succeed and the conflict is only found later, when asking the user may be too late. Making detection synchronous defeats the purpose (just use single-leader).
- **Resolution timing**: on write (a conflict handler runs when the conflict is detected in the replication stream) or on read (siblings stored, application resolves at read time, as in CouchDB).
- **Automatic resolution research**: CRDTs (counters, sets, maps, sequences) merge by design; mergeable persistent data structures track history like Git; operational transformation powers collaborative text editors.
- **Topologies**: circular and star topologies forward writes through intermediate nodes and need node IDs in each write to stop infinite loops; a single failed node can interrupt flow. All-to-all is more fault tolerant but messages can overtake each other, so causality can be violated (an update arriving before the insert it depends on); version vectors or logical timestamps are required.
- **Pitfalls**: auto-increment collisions across leaders, uniqueness constraints that cannot be checked locally, triggers firing in every DC, and poor testing coverage.

### 6. Performance and design implications
- Write latency is local; cross-DC bandwidth carries replication, not client requests.
- Throughput scales with leaders only if conflicts are rare.
- Conflict resolution logic becomes part of the application contract; test it with chaos tooling.
- Monitoring must track replication lag per leader pair and number of conflicts.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| MySQL group replication (multi-primary mode) / Tungsten | Multi-writer MySQL | Geo-distribution |
| PostgreSQL BDR / pglogical | Logical bidirectional replication | Multi-region active-active |
| CouchDB `_replicate`, conflict `_conflicts` | Sync with revision trees | Offline-first apps |
| Cassandra multi-DC `NetworkTopologyStrategy` | Leaderless across DCs | Alternative to multi-leader |
| CRDT libraries (Automerge, Yjs, Riak data types) | Automatic merge | Collaborative state |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Multi-region users need local write latency | Data needs strict constraints/uniqueness/transactions |
| Offline operation is a product requirement | Conflicts are frequent on shared records |
| Conflicts can be avoided by routing | Team cannot afford conflict-handling complexity |
| Writes are mergeable (counters, sets) | Overwrite-semantics data where LWW would lose money or orders |

### 9. Common mistakes
- Using LWW on important data → silent write loss → avoid, merge or keep siblings.
- Auto-increment IDs on each leader → duplicate keys → use UUIDs or per-leader ranges.
- Assuming all-to-all delivery preserves causality → out-of-order inserts/updates → version vectors.
- Enabling it "for HA" → conflict debt → single-leader plus failover is simpler.

### 10. 🎯 Interview kit
**30-second answer:** Multi-leader lets each datacenter or device accept writes locally and replicate asynchronously. It reduces latency and tolerates partitions but creates write conflicts, which must be avoided, or resolved via LWW, merging or siblings; hence it is used sparingly.

**Structure a longer answer:** motivation (latency, outage tolerance, offline) → architecture → conflict problem → avoidance and resolution techniques → topologies and causality → when you would not use it.

**Likely follow-ups:**
- *How to resolve conflicts without losing data?* → merge semantics, CRDTs, or siblings with app resolution.
- *Why not detect conflicts synchronously?* → you lose the independence that justifies multi-leader.
- *Which topology and why?* → all-to-all for fault tolerance, with version vectors for causality.

**Signals interviewers look for:** knowing conflicts are the central difficulty and listing multiple strategies with their data-loss properties.

### 11. Check yourself
- [ ] Describe the wiki-title conflict and two ways to resolve it.
- [ ] Explain why synchronous conflict detection defeats multi-leader.
- [ ] Compare circular, star and all-to-all topologies.
- [ ] Identify why auto-increment keys are risky.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Multi-Leader Replication Topologies
- *Designing Data-Intensive Applications* — Martin Kleppmann — Handling Write Conflicts
- *Designing Data-Intensive Applications* — Martin Kleppmann — Solutions for Replication Lag › Use Cases for Multi-Leader Replication
- *Designing Data-Intensive Applications* — Martin Kleppmann — Handling Write Conflicts › Custom conflict resolution logic
- *Designing Data-Intensive Applications* — Martin Kleppmann — Handling Write Conflicts › Converging toward a consistent state
- *Designing Data-Intensive Applications* — Martin Kleppmann — Limitations of Quorum Consistency › Multi-datacenter operation
- *Designing Data-Intensive Applications* — Martin Kleppmann — Databases and Streams › Keeping Systems in Sync
- *Designing Data-Intensive Applications* — Martin Kleppmann — Handling Write Conflicts › Synchronous versus asynchronous conflict detection
- *Designing Data-Intensive Applications* — Martin Kleppmann — Handling Write Conflicts › Conflict avoidance
- *Designing Data-Intensive Applications* — Martin Kleppmann — Solutions for Replication Lag › Clients with offline operation

