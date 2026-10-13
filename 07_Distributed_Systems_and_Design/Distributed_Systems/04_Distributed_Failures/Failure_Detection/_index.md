---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/04_Distributed_Failures/Failure_Detection"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Failure Detection

**L4 Advanced** · [↑ Faults and Failure in Distributed Systems](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A failure detector is a local component that guesses which peers are dead from the messages it does (or doesn't) receive — and because slow and dead look identical from outside, every detector trades false alarms against slow detection.

### 1. The big picture
In a distributed system you never *observe* that another node has crashed; you only observe that something you expected did not arrive. A crashed process, a process stuck in a long pause, a dropped packet and a congested link all produce the same symptom: silence. In a fully asynchronous system (no bound on message delay or process speed) it is provably impossible to tell these apart — this is the root of the FLP impossibility result for consensus.

Practical systems cope by adding timing assumptions and accepting that they will sometimes be wrong. Failure detection sits underneath leader election, replication, membership, and consensus: whenever a database "fails over", "removes a node from the ring", or "re-replicates a block", a failure detector pulled the trigger. This lesson covers the mechanisms (heartbeats, pings, outsourced heartbeats, phi-accrual, gossip, group-failure propagation). The sibling lessons cover *why* the underlying signals are noisy: network faults and timeouts, unreliable clocks, and how to stay safe even when the detector is wrong (fencing).

### 2. Core concepts and vocabulary
- **Crashed / dead / failed** — a process that has stopped executing steps for good.
- **Suspected** — a process that *looks* unresponsive (slow, faulty, unreachable) but may be perfectly alive.
- **Link failure vs process failure** — messages lost or delayed on the wire, versus the process itself dying or running slowly. Observationally the same.
- **Failure detector** — a local subsystem that classifies peers as alive/suspected/dead so the algorithm can exclude failed ones and keep making progress.
- **False positive** — marking a live process as dead (hurts availability, causes needless failover and duplicate work).
- **False negative** — leaving a dead process marked alive for too long (requests keep going to a node that cannot answer, hurting latency).
- **Liveness** — "something good must eventually happen" (a failure *will* be detected).
- **Safety** — "nothing bad happens" (a node marked dead really is dead). No timing-based detector can guarantee both perfectly.
- **Completeness** — every non-faulty member eventually notices a failed process.
- **Efficiency** — how fast, and with how many messages, the detector reaches that conclusion.
- **Accuracy** — how rarely it wrongly suspects live processes.
- **Heartbeat** — a periodic "I'm alive" message *pushed* by a process to its peers.
- **Ping** — a periodic probe *pulled* by the monitor, expecting an acknowledgment within a deadline.
- **Byzantine failure** — a process that lies; the detectors here assume it does not.

### 3. How it works, step by step
**Basic heartbeat/ping detector (timeout-based)**
1. Every process keeps a table: for each peer, the last time any message was received and a state (alive / suspected / dead).
2. Periodically each process sends a ping (or each peer sends a heartbeat).
3. On receiving a reply or heartbeat, the table entry's timestamp is refreshed and the peer is marked alive.
4. If no message arrives within the timeout, the peer is marked *suspected* (and, after a further grace period or policy, dead).
5. The "action" step fires: stop routing to it, start failover, re-replicate its data.

The weakness is visible in a delayed-reply scenario: P1 pings P2 and the acknowledgment arrives only *after* the next ping was already sent — P1 has already suspected a perfectly healthy process. Precision depends entirely on picking the ping interval and timeout well, and a single observer sees only its own link to the target.

![Pings for failure detection: responses delayed past the next probe](<../../../../_assets/Database_Internals_A_deep_dive_into_how_distributed_data_sys/dbin_0902.png>)

**Variants that reduce the single-observer problem**

| Approach | Core idea | Strength | Weakness |
|---|---|---|---|
| Deadline detector (e.g. Akka) | Fixed heartbeat + fixed timeout | Trivial, strong completeness | Brittle threshold; one observer's view |
| Timeout-free "Heartbeat" detector | Count heartbeats that propagate along paths; compare counters; no clocks | Works in an asynchronous model; correct despite faulty direct links | Needs full membership knowledge; picking a counter threshold is tricky |
| Outsourced heartbeats (SWIM) | On a missed ping, ask *k* random peers to probe the target and relay acks | Accounts for direct + indirect reachability; no broadcast; parallel | Extra messages on suspicion |
| Phi-accrual (Cassandra, Akka) | Output a continuous suspicion level φ from the statistical history of heartbeat arrivals | Adapts to network conditions; no fixed timeout | Assumes roughly normal arrival times; needs tuning of the φ threshold |
| Gossip-based | Each member increments its own counter and gossips its table to a random peer; merge by max | Reliable aggregate view; bandwidth capped and linear in group size | Slower convergence; more messages overall |
| FUSE (group failure) | Any detected failure makes the detector stop answering pings, so failure spreads as *group* failure | Every member learns; works under partitions | A single bad link can bring down the whole group |

**Outsourced heartbeats (SWIM) in detail**
1. P1 pings P2; no ack within the deadline.
2. P1 picks several random members (P3, P4) and asks them to ping P2 on its behalf.
3. If any of them gets an ack and forwards it, P2 is alive and the problem was the P1↔P2 link (or a blip).
4. If none do, P1 marks P2 as suspected and disseminates that.

![Outsourced heartbeats: P1 asks P3 and P4 to probe P2](<../../../../_assets/Database_Internals_A_deep_dive_into_how_distributed_data_sys/dbin_0903.png>)

**Phi-accrual in detail**
1. Keep a sliding window of the last N heartbeat inter-arrival times from a peer.
2. Estimate mean and variance (assuming a normal distribution).
3. When `t` time units have passed since the last heartbeat, compute the probability that a heartbeat is *still* going to arrive; φ is a logarithmic transform of that — higher φ means "more surely dead".
4. Compare φ to a threshold (Cassandra's `phi_convict_threshold`, default 8). Crossing it fires the callback.

Because the window slides, a noisy cross-datacenter link automatically gets a looser effective timeout than a quiet local one. Architecturally it separates *monitoring* (collecting samples), *interpretation* (turning them into φ and a decision) and *action* (the callback).

**Gossip detection in detail**
1. Each node keeps `{member → (heartbeat counter, local time the counter last increased)}`.
2. Every interval it bumps its own counter and sends its table to a random neighbor.
3. The receiver merges, keeping the larger counter per member and stamping the local time when a counter increases.
4. A member whose counter has not advanced for a configured period is declared failed.
5. A node whose direct link is cut is still seen alive because its counter arrives via other nodes.

### 4. Worked example
Cassandra uses gossip plus a phi-accrual detector. Inspect it on a running cluster:

```bash
# Cluster membership and the state each node believes about its peers
nodetool status
nodetool gossipinfo | head -40

# Relevant setting in cassandra.yaml
# phi_convict_threshold: 8   # raise to 10–12 on noisy cloud networks (e.g. EC2)
```

Scenario: node C is under a long GC pause. Heartbeats from C stop arriving at A. A's φ for C climbs: at 2× the usual interval φ is small (just jitter), at ~8× it crosses the threshold and A marks C `DOWN`; reads are routed to replicas and hints are stored. When the pause ends, C's heartbeats resume, gossip carries the fact back, and A flips C to `UP`. Lowering the threshold (say 5) detects true failures faster but flaps during GC pauses; raising it (12) tolerates noise but delays failover.

### 5. Under the hood
- **Why accuracy and completeness conflict.** Because delay is unbounded, a detector that never makes mistakes can never conclude anything (an unbounded wait), and a detector that always concludes must sometimes be wrong. Real detectors are "eventually" accurate under partial synchrony: after the system stabilizes, they stop making mistakes.
- **Heartbeat vs ping cost.** Pings need a request/response pair per probe (twice the messages, but the monitor controls the schedule and measures round trip directly). Heartbeats are one-way and cheaper, but the receiver must know the sender's period.
- **Epidemic dissemination.** Gossip spreads a fact to all *n* nodes in O(log n) rounds with constant per-node fan-out, which is why Dynamo-style systems, Consul/Serf and Cassandra use it for membership.
- **Suspicion state in SWIM.** Production SWIM variants add a *suspect* sub-protocol: a suspected node can refute by incrementing an incarnation number, which reduces false positives further.
- **Assumption of honesty.** All of these assume non-Byzantine processes; tolerating lying nodes needs a different class of protocol.

### 6. Performance and design implications
- Failover time ≈ detection time + election/reassignment time. Aggressive detection speeds recovery but multiplies false failovers.
- Declaring a slow node dead moves its load to others, which can be the final push into a **cascading failure** — all nodes starving each other of heartbeats. Prefer detectors that adapt (φ-accrual) or add jitter/backoff on takeover.
- Heartbeats should travel over a path as similar as possible to real traffic; a dedicated "health" thread that is alive while the data path is stuck gives misleading liveness.
- Gossip message volume per node is bounded and independent of cluster size per round, which keeps it cheap even at hundreds of nodes.
- Measure: track false-failover counts, flapping node state changes, and heartbeat-arrival jitter percentiles.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `nodetool status` / `gossipinfo` (Cassandra) | Show peers' UP/DOWN and gossip state | Debug membership flapping |
| `phi_convict_threshold` (Cassandra) | φ at which a node is convicted | Raise on high-latency/noisy networks |
| `zookeeper.tickTime`, `sessionTimeout` | Heartbeat tick and session expiry in ZooKeeper clients | Tune failover vs false expiry |
| `heartbeat.interval.ms` / `session.timeout.ms` (Kafka consumer) | Consumer liveness to the group coordinator | Prevent needless rebalances during GC/long processing |
| `dfs.heartbeat.interval`, `dfs.namenode.heartbeat.recheck-interval` (HDFS) | DataNode heartbeat and dead-node timeout (~10.5 min default) | Control how soon blocks get re-replicated |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Simple fixed-timeout heartbeats — small, stable LAN cluster | WAN or cloud links with variable latency |
| Phi-accrual — heterogeneous, variable-latency links | You need a hard, predictable upper bound on detection time |
| SWIM-style indirect probes — large clusters, flaky individual links | Tiny clusters where there are no useful third parties |
| Gossip — membership at scale, no central coordinator | You need fast, strongly consistent membership (use consensus instead) |
| Group-failure (FUSE) semantics — all-or-nothing group liveness | A single link flap must not tear down the group |

### 9. Common mistakes
- Treating "no reply" as proof of death → split brain / duplicate work when the node was only paused → combine detection with fencing or leases and idempotent actions.
- One global timeout copied across LAN and cross-region links → constant false positives → per-link adaptive thresholds.
- Declaring dead instantly under load → shifted load triggers more timeouts → add grace periods, backoff, and rate-limited re-replication.
- Monitoring only process liveness, not responsiveness → a hung-but-running node stays "alive" → probe a real code path.
- Forgetting that GC or VM pauses affect the *detector's own* thread too.

### 10. 🎯 Interview kit
**30-second answer:** A failure detector infers crashes from missing heartbeats or ping replies. You can't distinguish a dead node from a slow one, so it's a trade-off between false positives and detection latency. Better designs use indirect probes (SWIM), adaptive suspicion levels (phi-accrual), or gossip, so one observer's bad link doesn't decide the outcome.
**Structure a longer answer:** clarify the system model (async vs partially synchronous, crash vs Byzantine) → heartbeat/timeout baseline → its weaknesses → adaptive/collective detectors → how consumers stay safe despite mistakes (fencing, idempotence).
**Likely follow-ups:**
- *Why can't we detect failures perfectly?* → asynchronous model: delays unbounded; slow ≡ dead (FLP).
- *How does Cassandra decide a node is down?* → gossip carries heartbeat state; phi-accrual over arrival times vs a threshold.
- *What happens when the detector is wrong?* → needless failover; need fencing/leases so a revived node can't corrupt state.
- *Heartbeat vs ping?* → push vs pull; cost and who owns the schedule.
**Signals interviewers look for:** you name the slow-vs-dead ambiguity, mention both error types, know at least one adaptive or collective technique, and connect detection to safe takeover.

### 11. Check yourself
- [ ] Explain why a detector can't be both perfectly accurate and complete in an asynchronous system.
- [ ] Walk through a SWIM probe when P2 is alive but its link to P1 is down.
- [ ] Describe how φ is computed and what raising the threshold does.
- [ ] Explain how gossip detects a crashed node *and* tolerates a cut link.
- [ ] Give a concrete case where a false positive causes a cascading failure, and how to avoid it.
<!-- explainer:end -->

## References

- *Database Internals* — Alex Petrov — Failure Detection › Phi-Accural Failure Detector
- *Database Internals* — Alex Petrov — Failure Detection › Chapter 9. Failure Detection
- *Database Internals* — Alex Petrov — Failure Detection › Gossip and Failure Detection
- *Database Internals* — Alex Petrov — Failure Detection › Outsourced Heartbeats
- *Database Internals* — Alex Petrov — Failure Detection › Timeout-Free Failure Detector
- *Database Internals* — Alex Petrov — Failure Detection › Heartbeats and Pings
- *Database Internals* — Alex Petrov — Failure Detection › Reversing Failure Detection Problem Statement
- *Database Internals* — Alex Petrov — M › P

