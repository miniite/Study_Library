---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/04_Distributed_Failures"
level: 3
type: "curriculum"
curriculum: "data_engineering"
---

# Faults and Failure in Distributed Systems

**L3 Practitioner** · [↑ Distributed Systems](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** In a distributed system parts fail independently and unpredictably, you can't tell slow from dead, and the only way to build something dependable is to state your failure and timing assumptions explicitly, design for partial failure, and let a quorum — not any single node — decide what is true.

### 1. The big picture
A single computer is mostly all-or-nothing: it works deterministically or it crashes outright, and that's a deliberate design choice (a crash is preferable to a silently wrong answer). Once several machines cooperate over a network, that idealized world is gone. Some parts are broken in unpredictable ways while others work fine — a **partial failure** — and because the outcome of anything involving multiple nodes is nondeterministic, you may not even know whether an operation happened.

The core job of a distributed-systems engineer is to build a reliable system from unreliable components. That is not paradoxical: TCP is reliable on top of lossy IP, and error-correcting codes give clean data over noisy channels. There are always limits (TCP can't remove delay), but a layered abstraction makes the remaining faults easier to reason about.

This block has four child lessons you should read in this order:
1. **Network Faults and Timeouts** — why the network gives no delay bound and why timeouts are guesses.
2. **Unreliable Clocks** — why per-machine clocks can't order events or measure leases safely.
3. **Failure Detection** — heartbeats, SWIM, phi-accrual and gossip: how nodes decide others are dead.
4. **Fencing and System Models** — how to stay correct when detection is wrong, and what assumptions algorithms may make.

It feeds directly into the next block on transactions, consensus and leader election, which is where "who is alive and who is the leader" becomes an agreed answer.

### 2. Core concepts and vocabulary
- **Fault** — one component deviating from spec. **Failure** — the whole system stops providing the service. Fault tolerance = preventing faults from becoming failures.
- **Partial failure** — some components broken, others working; nondeterministic.
- **Network partition (netsplit)** — groups of nodes cannot communicate and may each continue, producing conflicting results. Links can also fail *asymmetrically* (A hears B but B doesn't hear A).
- **Fallacies of distributed computing** — false assumptions: the network is reliable; latency is zero; bandwidth is infinite; the network is secure; topology doesn't change; there is one administrator; transport cost is zero; the network is homogeneous.
- **Shared-nothing** — machines share only the network.
- **Systematic / correlated fault** — a software bug or resource exhaustion that hits many nodes at once; far more damaging than independent hardware faults.
- **Cascading failure** — one component's fault triggers faults in others.
- **Failure model** — the formal description of how processes may fail: *crash* (stop for good), *crash-recovery* (stop, maybe return, durable state survives), *omission* (miss sending/receiving some messages), *arbitrary/Byzantine* (anything, including lying).
- **System (timing) model** — synchronous, partially synchronous, asynchronous.
- **Quorum** — a minimum number of votes (usually a majority) needed for a decision.
- **Availability, reliability, RTO, RPO** — percentage of time operable; probability of meeting spec over an interval; maximum tolerable outage time; maximum tolerable data loss.

### 3. How it works, step by step
**Why a request can silently go wrong.** After you send a request and hear nothing, one of these happened and you cannot tell which:
1. the request was lost on the way;
2. it is queued somewhere (network or remote OS);
3. the remote node crashed;
4. the remote node is paused (GC, VM, swap) and will resume;
5. it processed the request but the response was lost;
6. it processed the request but the response is delayed.

The only generic remedy is a timeout, followed by a decision: retry (needs idempotence), fail over, or give up.

**What feedback you may get, and why not to depend on it.** A closed port yields a TCP `RST`/`FIN`; a crashed process can have a supervisor script notify others (HBase does this); switch management APIs can report link failures; routers can send ICMP "destination unreachable". These accelerate detection but can't be relied upon. Even a TCP ack proves only that the OS got the data — if you need to know the work was done, you need an application-level reply.

**Deciding who is dead: truth by quorum.** Three stories show why no node can trust its own view:
1. *Asymmetric link:* node N receives everything but its replies are dropped. The others declare it dead while it insists it is healthy.
2. *Long GC pause:* N is frozen for a minute, declared dead, then wakes up and cheerfully carries on, unaware time passed.
3. *Process failure plus clock error:* N believes it holds a lease that has already expired.

Hence decisions (who is leader, is X dead) are made by a **majority quorum**: with 3 nodes you tolerate 1 failure, with 5 you tolerate 2. If the quorum declares a node dead, it *is* dead as far as the cluster is concerned, and the node must step down. Because two disjoint majorities can't exist, the quorum is safe against split brain even though it costs availability on the minority side. Dangerous leftovers (zombie leaders) are then neutralized with fencing tokens.

**Choosing assumptions.**
| Dimension | Models | Realistic default |
|---|---|---|
| Timing | synchronous / partially synchronous / asynchronous | partially synchronous |
| Node faults | crash-stop / crash-recovery / Byzantine | crash-recovery |
| Network | reliable-link / fair-loss / partitions possible | partitions possible |

An algorithm is proved correct *within a model*; the engineering question is how closely your real system matches it. Good designs keep **safety** unconditional (never wrong, even under extreme delays) and let **liveness** depend on timing (eventually progresses once things calm down).

### 4. Worked example
You operate a 3-replica database with automatic failover. A top-of-rack switch flaps for 90 seconds.

1. The leader is in the isolated rack; two followers and the clients' load balancer are on the other side.
2. Followers' failure detectors stop seeing leader heartbeats; φ/timeout trips; they hold an election and form a 2-of-3 majority → new leader, higher term/epoch.
3. The old leader still sees clients in its own rack. It cannot reach a majority so it cannot commit writes (quorum-based replication) and must stop accepting them or only serve stale reads.
4. The switch recovers; the old leader sees a higher term, steps down and resynchronizes.
5. Any write it tried to push to shared storage during the gap is rejected because its fencing token/epoch is stale.

Resulting checks you should be able to run on a real system:

```bash
# is the problem the network or the node?
mtr -rwc 50 <peer>           # loss and latency per hop
ss -ti dst <peer>            # TCP retransmits / RTT
# is time sane?
chronyc tracking             # offset from NTP
# is membership stable?
nodetool status              # (Cassandra) or kafka-metadata / zkCli stat
```

Then inject the failure on purpose in a test environment (`tc netem`, Toxiproxy) and verify the system keeps the intended guarantee.

### 5. Under the hood
- **Why deterministic code becomes nondeterministic:** the same operation can succeed, fail, or hang depending on queues, pauses and path; correctness cannot depend on "usually it responds in 10 ms".
- **Testing strategy:** it's nearly impossible to enumerate failure scenarios by reasoning; build harnesses that create partitions, add latency, skew clocks, vary relative speeds and corrupt data (Toxiproxy for network faults, Chaos Monkey for production instance kills, Jepsen-style tests for consistency).
- **Correlated failure:** hardware faults are roughly independent, but software bugs, config pushes and leap seconds are not; replicating a bug onto every node is not redundancy. Defenses: staged rollouts, process isolation, crash-and-restart supervision, self-checking invariants (e.g. in-count equals out-count for a queue), and diverse failure domains (rack, AZ, region).
- **Recovery objectives:** RTO and RPO turn "plan for failure" into numbers that drive replication mode (sync vs async), backup frequency and standby topology. In data engineering, "everything fails all the time" means pipelines must be idempotent, restartable and observable.
- **Weak forms of lying** (corrupt packets, bad input, wrong NTP server) should be guarded with checksums, validation and multi-source agreement even though full Byzantine tolerance is overkill in one datacenter.

### 6. Performance and design implications
- Every detection and takeover decision trades latency against stability; pick timeouts from measured percentiles, not guesses.
- Majority quorums add a round trip to the slowest member of the majority — place replicas knowing cross-region latency.
- Retries without backoff turn a blip into an outage; always pair them with jitter, budgets and circuit breakers.
- Make operations idempotent (idempotency keys, upserts, deterministic output paths) so "at-least-once" is safe.
- Prefer a small number of well-understood failure domains and test failover regularly; untested failover is a hypothesis, not a feature.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `tc qdisc ... netem` | Add delay/loss/partition on Linux | Fault injection in staging |
| Toxiproxy | Programmable TCP proxy with latency/timeouts/bandwidth toxics | App-level resilience tests |
| Chaos Monkey / Gremlin | Randomly kill instances / inject faults | Production-readiness drills |
| `chronyc tracking` | Clock offset and drift | Time sanity |
| Client timeouts, retry+backoff, circuit breaker | Bound waiting and amplification | Every remote call |
| Replication / ack settings (`min.insync.replicas`, `acks`, quorum `W`/`R`) | How many replicas must confirm | Balancing durability and availability |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Majority-quorum decisions (3 or 5 nodes) | Only two nodes available (no majority possible on split) |
| Crash-recovery model with durable logs | Treating memory as durable |
| Short failure-detection timeouts | Load is near saturation or networks are noisy |
| Synchronous replication — no loss on failover | Latency-sensitive paths or cross-region links |
| BFT consensus | Single trust domain; cost not justified |
| Fail-fast, restart quickly | Restart storms that thrash a shared dependency |

### 9. Common mistakes
- Assuming the network is reliable / latency is low → timeouts and duplicate effects in production → design for loss, delay and retries.
- Declaring dead on first missed heartbeat → flapping and cascading load → adaptive thresholds, quorum confirmation.
- Letting one node "decide" it's leader from its own clock → split brain → majority election + fencing.
- Treating redundancy as protection from a shared bug or config push → correlated outage → staged rollouts, diversity.
- Never rehearsing failures → surprises in an incident → scheduled game days.
- Confusing availability numbers with RPO → designing for uptime but losing data → set RTO and RPO separately.

### 10. 🎯 Interview kit
**30-second answer:** Distributed systems fail partially and nondeterministically. You can't distinguish a dead node from a slow link or a long pause, so you use timeouts and failure detectors that are sometimes wrong, make decisions by majority quorum, and protect resources with fencing so a wrongly-declared-dead node can't do harm. You also state your system model — typically partially synchronous with crash-recovery nodes — and test faults deliberately.
**Structure a longer answer:** partial failure and why it's hard → sources (network, clocks, pauses, correlated software faults) → detection and its error trade-off → quorum for truth → safety mechanisms (fencing, idempotence) → testing/chaos and recovery objectives.
**Likely follow-ups:**
- *What's a network partition and what does a system do?* → groups can't talk; quorum side continues, minority stops or serves stale reads.
- *Why majority quorums?* → any two majorities intersect, so there can't be two conflicting decisions.
- *Crash-stop vs crash-recovery vs Byzantine?* → gone forever / returns with durable state / lies.
- *How do you test this?* → fault injection, chaos engineering, Jepsen-style checkers.
**Signals interviewers look for:** you reason in terms of partial failure and ambiguity rather than "node up/down", you separate safety from liveness, and you mention both detection and containment.

### 11. Check yourself
- [ ] List six reasons a request might get no response and explain why they are indistinguishable.
- [ ] Explain why a node cannot trust its own view of whether it is the leader.
- [ ] Describe how majority quorums prevent split brain and what they cost.
- [ ] Contrast crash-stop, crash-recovery and Byzantine failure models and name a system for each assumption.
- [ ] Define RTO and RPO and show how each drives an architectural choice.
- [ ] Sketch a fault-injection test plan for a replicated service.
<!-- explainer:end -->

## In this section (study in order)

1. [Network Faults and Timeouts](<Network_Faults_and_Timeouts/_index.md>) (L4 Advanced)
2. [Unreliable Clocks](<Unreliable_Clocks/_index.md>) (L4 Advanced)
3. [Failure Detection](<Failure_Detection/_index.md>) (L4 Advanced)
4. [Fencing Tokens and System Models](<Fencing_and_System_Models/_index.md>) (L4 Advanced)

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Knowledge, Truth, and Lies › The Truth Is Defined by the Majority
- *Designing Data-Intensive Applications* — Martin Kleppmann — Knowledge, Truth, and Lies
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 8. The Trouble with Distributed Systems › Cloud Computing and Supercomputing
- *Database Internals* — Alex Petrov — Introduction and Overview › Failure Models
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 8. The Trouble with Distributed Systems › Faults and Partial Failures
- *Database Internals* — Alex Petrov — Introduction and Overview › Network Partitions and Partial Failures
- *Designing Data-Intensive Applications* — Martin Kleppmann — Unreliable Networks
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Principle 2: Plan for Failure
- *Database Internals* — Alex Petrov — Introduction and Overview › Fallacies of Distributed Computing
- *Designing Data-Intensive Applications* — Martin Kleppmann — Reliable, Scalable, and Maintainable Applications › Software Errors

