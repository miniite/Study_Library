---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/03_Transactions_and_Consistency/Consensus/Leader_Election"
level: 5
type: "curriculum"
curriculum: "data_engineering"
---

# Leader Election

**L5 Expert** · [↑ Consensus Algorithms](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Leader election is how a group of equal processes picks exactly one coordinator to sequence work, and picks a new one when it fails — the hard part is guaranteeing "at most one leader" under partitions and slow nodes.

### 1. The big picture
Coordinating every step among all participants is expensive. A **leader** (coordinator) cuts the cost: it holds the global state, orders operations (total order broadcast), and drives reorganisation after failures. Leadership is a *temporary role* any process can assume; the leader holds it until it crashes or is suspected dead, and then an election picks a successor.

Two properties define a good election. **Liveness**: most of the time there is a leader and elections eventually finish. **Safety**: at most one leader at a time — no *split brain* where two nodes each believe they lead and each accepts writes. Many practical algorithms are strong on liveness and weak on safety when the network partitions; consensus-based election (Raft terms, ZooKeeper, etcd) closes that gap with quorums and epoch numbers.

This lesson sits inside the consensus topic; failure detection (to know *when* to elect) and fencing (to neutralise a deposed leader) are covered in the Distributed Failures topic.

### 2. Core concepts and vocabulary
- **Leader / coordinator** — the single process responsible for a decision stream; known to all peers.
- **Election** — protocol that selects one leader deterministically and informs everyone.
- **Rank / ID** — a comparable value (process ID, quality score) used to break ties.
- **Split brain** — two leaders active simultaneously, usually on opposite sides of a partition.
- **Term / epoch / generation** — a monotonically increasing number identifying a leadership period; stale leaders carry lower numbers.
- **Quorum-based election** — a candidate needs votes from a majority, making two simultaneous leaders in the same term impossible.
- **Lease** — a time-limited leadership grant that must be renewed.
- **Distributed lock** — exclusive ownership of a resource; unlike leadership, *who* holds it matters less than that it is eventually released.
- **Fencing token** — a monotonically increasing number the resource checks to reject writes from deposed leaders.
- **Bully algorithm** — highest-ranked live process wins.
- **Invitation algorithm** — groups with their own leaders merge by invitation.

### 3. How it works, step by step
**Bully algorithm**
1. A process notices the leader is gone (or none exists) and sends `Election` to all processes with *higher* IDs.
2. Any higher process that is alive replies `Alive` and takes over the election (starts its own round).
3. If nobody higher answers within a timeout, the initiator declares itself leader and sends `Elected` to all lower-ranked processes.
4. Others record the new leader.

**Next-in-line failover.** Each leader publishes an ordered list of alternates. When it fails, a detector contacts the highest-ranked alternate directly; if that one is alive it becomes leader at once, skipping a full election round.

**Candidate/ordinary optimisation.** Nodes are split into *candidates* (eligible leaders) and *ordinary* nodes. An ordinary node that detects failure contacts only candidates and picks the highest alive one. To avoid simultaneous elections, each node has a tiebreaker delay δ — larger than a round trip, smaller for higher-priority nodes — so one initiator goes first.

**Invitation algorithm.**
1. Every process starts as leader of a singleton group.
2. Group leaders invite peers outside their group. If the invitee is itself a leader, the groups merge; otherwise it reveals its leader so the two leaders can talk.
3. The leader of the larger group typically becomes the merged leader, so fewer nodes need notification.
This deliberately allows several leaders (one per group) and merges after partitions heal.

**Raft election (quorum-based)**
1. Follower's election timer (randomised, e.g. 150–300 ms) fires → becomes candidate, term+1, votes for self, sends `RequestVote`.
2. Each node grants at most one vote per term, and only to a candidate whose log is at least as up to date.
3. A majority makes the candidate leader; it sends heartbeats immediately to suppress other elections.
4. Split vote → timers fire again at different random moments; eventually one wins.
5. A leader that sees a higher term steps down.

### 4. Worked example
Bully algorithm with processes 1–6, leader 6 crashes, process 3 notices:
```
3 -> 4,5,6 : Election
4 -> 3     : Alive          (4 starts own election)
5 -> 3     : Alive          (5 starts own election)
(6 silent)
4 -> 5,6   : Election ; 5 -> 4 : Alive
5 -> 6     : Election ; no reply within timeout
5 -> 1,2,3,4 : Elected(leader=5)
```
A practical equivalent with a coordination service (leader election recipe):
```bash
# etcd: campaign for leadership and hold it while the session lives
etcdctl elect payments-leader node-b
# in another terminal
etcdctl elect payments-leader --listen    # prints the current leader
```
The key is attached to a *lease*; if `node-b` dies its lease expires and the next campaigner is promoted. The revision number acts as a fencing token — downstream writers should reject requests with an older revision.

### 5. Under the hood
- **Bully's weaknesses.** Under partition each side elects its own highest-ranked node → split brain. And an unstable high-ranked node can win, crash, win again, producing perpetual re-election; remedies include distributing node-quality metrics and electing on health, not just rank.
- **Safety needs quorum or external arbiter.** Majority voting means two partitions can't both win; the minority side simply cannot elect and becomes unavailable for writes.
- **A deposed leader doesn't know it.** It may pause (GC, VM freeze), wake after its lease expired, and continue acting. Defence: epoch numbers in every message and fencing tokens at the storage layer; leases must account for bounded clock drift.
- **Raft leader completeness.** Only candidates with all committed entries can win, so entries flow leader → follower only. Log comparison uses (last term, last index).
- **Election vs lock.** Leaders must be announced and known; locks only require eventual release. A lock scheme that prefers some processes starves others (violating liveness), whereas leadership by design favours one.
- **Timeouts as failure detection.** Elections are triggered by suspected failure, which can be a false positive (slow network). Short timeouts → fast failover but flapping; long → slow recovery.

### 6. Performance and design implications
- Election time = failure detection time + election round(s). Typical Raft: a few hundred ms to seconds; during that window writes stall.
- Frequent elections often signal GC pauses, disk stalls (fsync), or overloaded networks — fix the cause, not just timeouts.
- Prefer delegating election to a coordination service (etcd, ZooKeeper, Consul) rather than rolling your own; use ephemeral nodes/leases.
- Keep election messages and leader state small; the leader should not be a data-path bottleneck if avoidable (shard leaders per partition).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `etcdctl elect` | Campaign/observe election | Lightweight leader election |
| ZooKeeper ephemeral sequential znodes | Lowest sequence = leader | Classic recipe, e.g. Curator `LeaderLatch` |
| Kubernetes `Lease` objects (leader-election in controllers) | Lease-based election | Controller HA |
| `election-timeout` / `heartbeat-interval` | Detection vs flapping | Tune for network RTT |
| PostgreSQL Patroni `ttl`, `loop_wait` | Lease timing for DB failover | HA clusters |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Quorum-based (Raft/ZK) election: correctness matters | You only have two nodes — no majority |
| Bully/ranked: small trusted cluster, partitions unlikely, simplicity | Partitions possible and split brain is costly |
| Invitation/merge: tolerate multiple groups, merge later | The resource requires exclusive access |
| Lease + fencing token | Clocks are unsynchronised and storage can't check tokens |

### 9. Common mistakes
- Believing election alone prevents split brain → two active leaders after a pause → fence with epochs/tokens.
- Two-node "clusters" with automatic failover → can't distinguish crash from partition → add a witness/third vote.
- Leader renewing lease based on local time only → expired lease still acted upon → check time since last successful quorum contact.
- Making the leader do all reads/writes forever → bottleneck → partition leadership.
- Electing on rank only → flapping node keeps winning → include health.

### 10. 🎯 Interview kit
**30-second answer:** Leader election picks one coordinator among equal nodes, re-running when it fails. Simple algorithms like bully choose the highest rank; safe ones use a majority vote with term numbers (Raft) so two leaders can't coexist, plus fencing for stale leaders.

**Structure a longer answer:** why a leader → liveness vs safety → algorithm (bully/Raft) → split brain & fencing → real-world implementation (etcd lease/ZooKeeper).

**Likely follow-ups:**
- *How does Raft avoid split votes?* → Randomised timeouts; retry in a new term.
- *Election vs distributed lock?* → Leader must be known to all; lock holder only needs to release.
- *What if the old leader is paused?* → Terms and fencing tokens reject its writes.
- *Why not use wall-clock leases only?* → Clock drift can make two nodes believe they hold the lease.

**Signals interviewers look for:** unprompted mention of split brain, epochs, quorum, and fencing; practical knowledge of a coordination service.

### 11. Check yourself
- [ ] Run the bully algorithm by hand for a crashed leader and a lower-ranked detector.
- [ ] Explain how a partition causes split brain in bully but not in Raft.
- [ ] Describe what a fencing token is and where it must be checked.
- [ ] List two optimisations to shorten elections.
- [ ] Contrast leader election with distributed locking.
<!-- explainer:end -->

## References

- *Database Internals* — Alex Petrov — Leader Election › Bully Algorithm
- *Database Internals* — Alex Petrov — Leader Election › Candidate/Ordinary Optimization
- *Database Internals* — Alex Petrov — Leader Election › Chapter 10. Leader Election
- *Database Internals* — Alex Petrov — Leader Election › Next-In-Line Failover
- *Database Internals* — Alex Petrov — Raft › Leader Role in Raft
- *Database Internals* — Alex Petrov — Raft › Failure Scenarios
- *Database Internals* — Alex Petrov — Leader Election › Invitation Algorithm

