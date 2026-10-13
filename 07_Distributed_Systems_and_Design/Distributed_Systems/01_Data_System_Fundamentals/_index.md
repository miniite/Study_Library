---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/01_Data_System_Fundamentals"
level: 3
type: "curriculum"
curriculum: "data_engineering"
---

# Reliability Scalability Maintainability

**L3 Practitioner** · [↑ Distributed Systems](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A data system is good when it keeps working correctly under faults (reliability), keeps performing as load grows (scalability), and stays easy for many people to change over years (maintainability).

### 1. The big picture
Most modern applications are *data-intensive* rather than *compute-intensive*: the hard part is not raw CPU power but the volume of data, its complexity, and how fast it changes. Such applications are assembled from standard building blocks: **databases** (store data so it can be found again), **caches** (remember expensive results), **search indexes** (keyword and filter queries), **stream processing** (handle messages asynchronously) and **batch processing** (periodically crunch accumulated data).

These blocks are such successful abstractions that we rarely think about them, but each has different characteristics, and combining several of them into one application is the real engineering job. Everything else in distributed systems (replication, partitioning, consensus, transactions) exists to deliver three properties: reliable, scalable, maintainable. This lesson is the vocabulary for the rest of the section.

### 2. Core concepts and vocabulary
- **Data-intensive application** — an application whose main challenge is data volume, complexity or rate of change, not CPU.
- **Fault** — one component deviating from its spec (a disk dies, a node crashes). **Failure** — the whole system stops serving the user. Reliable systems tolerate faults so they do not become failures.
- **Reliability** — continuing to work correctly, even when things go wrong (hardware faults, software bugs, human error).
- **Scalability** — the ability to cope with increased load. It is not a yes/no label; it means "if load grows in *this* way, what are our options?"
- **Load parameters** — a few numbers describing current load: requests/sec, read/write ratio, active users, cache hit rate.
- **Fan-out** — how many downstream requests one incoming request triggers (e.g., one tweet delivered to all followers).
- **Vertical scaling (scale up)** — a bigger machine. **Horizontal scaling (scale out)** — more machines, a *shared-nothing* architecture.
- **Elasticity** — a scalable system that adds and removes resources automatically with load (some even scale to zero when idle).
- **Availability** — the fraction of time a service is operable. **Maintainability** — how easily operations, new engineers and evolving requirements can be accommodated.

### 3. How it works, step by step
**Reasoning about scalability** follows a fixed routine:
1. **Describe the load** with the parameters that matter for *your* architecture.
2. **Describe the performance**: throughput for batch systems, response-time distribution for online ones (see the next lesson).
3. Ask the two growth questions: (a) if load grows and resources stay fixed, how does performance degrade? (b) how many more resources are needed to hold performance constant?
4. Choose a scaling approach and expect to re-architect every ~10× of growth.

**Worked case: timeline fan-out.** A social site has two operations: *post a tweet* (a few thousand/sec) and *read home timeline* (hundreds of thousands/sec). Two designs:

| Design | Write cost | Read cost |
|---|---|---|
| Insert into a global table; at read time join followers → tweets and merge | cheap | expensive (a heavy join per read) |
| Keep a precomputed timeline per user; at write time push the tweet into every follower's timeline | expensive (fan-out ∝ followers) | cheap (read one list) |

Because reads vastly outnumber writes, the second wins — except for celebrities with millions of followers, where fan-out on write is prohibitive. The production answer is a **hybrid**: fan out on write for normal users, merge celebrities' tweets in at read time. The key load parameter was not tweet volume but the *distribution of followers per user*.

### 4. Worked example
```text
Capacity thinking for an events pipeline (back-of-envelope)
- 50k events/sec average, 3x peak  -> design for 150k/sec
- 1 KB per event                   -> 150 MB/s peak ingest
- one broker node sustains ~50 MB/s (measured, not guessed)
  -> need >= 3 partitions/nodes at peak, 6 for headroom and one node loss
- retention 7 days x 50 MB/s avg   -> ~30 TB, x3 replication = ~90 TB
```
The point is the method: state the load parameters, measure per-node capacity, then derive node count, remembering to add headroom for failure and to multiply storage by the replication factor.

### 5. Under the hood
*Reliability* work splits by fault type:
- **Hardware faults** (disk MTTF ≈ 10–50 years means ~1 disk death per day in a 10,000-disk cluster) are handled by redundancy: RAID, replicas, multiple data centers. Cloud platforms increasingly prefer software fault tolerance (rolling upgrades, replacing nodes) to single-machine reliability.
- **Software faults** are correlated (the same bug crashes every node on the same bad input), so redundancy does not help; you need testing, isolation, process restarts, monitoring and chaos testing.
- **Human errors** are the leading cause of outages; mitigate with good abstractions and APIs, sandboxes, easy rollback, gradual rollout, and detailed monitoring.

*Scalability* in practice: stateless services scale out trivially behind a load balancer; stateful data systems are hard to distribute, so the common wisdom was "keep the database on one node until cost or availability forces you out". There is no generic "magic scaling sauce": an architecture is shaped by which of reads, writes, data volume, data complexity, response-time needs or access patterns dominates.

*Maintainability* rests on three principles: **operability** (make routine tasks easy), **simplicity** (remove accidental complexity with good abstractions), **evolvability** (make change easy as requirements shift).

### 6. Performance and design implications
- Scaling decisions follow measurements, not intuition: profile the bottleneck (CPU, I/O, network, lock contention) before adding nodes.
- Distributing a stateful system adds coordination cost, new failure modes (partial failure, network partitions) and operational burden; a bigger single machine is often cheaper and simpler.
- Redundancy raises availability but also cost and consistency complexity; choose how many nines you actually need.
- Design for failure: assume every component will fail eventually, and plan backup, failover and graceful degradation.

### 7. Commands and settings you should know
| Concept / tool | What it does | When to use |
|---|---|---|
| Load balancer | Spreads requests over stateless replicas | Scaling out the application tier |
| Autoscaling group / elastic compute | Adds/removes nodes with load | Unpredictable or spiky workloads |
| Replication | Copies data to more nodes | Availability and read scaling |
| Partitioning (sharding) | Splits data across nodes | Data or write volume beyond one node |
| SLO / SLA | Written targets on latency and availability | Making "reliable enough" measurable |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Scale up: workload fits one machine, you want simplicity | Single point of failure or hardware ceiling is near |
| Scale out: you need capacity, fault tolerance or geo-distribution | Team cannot operate distributed state yet |
| Elastic (auto) scaling: load is unpredictable | Load is steady; manual scaling has fewer surprises |
| Prioritize reliability: money, health, user trust at stake | Throwaway prototype where speed to market matters (cut corners consciously) |

### 9. Common mistakes
- Saying "this system is scalable" without naming the growth dimension → meaningless claim → state which load parameter grows and what you will do.
- Measuring average response time → hides tail pain → use percentiles.
- Treating redundancy as protection from software bugs → correlated failures take out all replicas → add isolation, rollout discipline and monitoring.
- Over-engineering for 100× scale on day one → complexity tax with no benefit → design for ~10× and re-evaluate.

### 10. 🎯 Interview kit
**30-second answer:** Reliability is continuing to work correctly despite faults; scalability is having a plan for coping with growth in specific load parameters; maintainability is keeping the system operable, simple and evolvable. Describe load and performance numerically before choosing an architecture.

**Structure a longer answer:** clarify what grows (reads, writes, data) → name load parameters → show how the design copes (scale up vs out, caching, fan-out strategy) → discuss failure modes and trade-offs → close with a real scenario such as the hybrid timeline.

**Likely follow-ups:**
- *Scale up or out?* → start simple; scale out when cost or availability demands, expect stateful systems to be harder.
- *How do you handle celebrities in a feed?* → hybrid fan-out: push for most, pull and merge for high-fan-out accounts.
- *Why are human errors important?* → leading cause of outages; design for easy rollback and gradual rollout.

**Signals interviewers look for:** quantitative thinking, naming load parameters, awareness that scalability is relative to a workload, and honest trade-off discussion.

### 11. Check yourself
- [ ] Define fault vs failure and give an example of each.
- [ ] List the load parameters for a system you know.
- [ ] Explain fan-out on write vs on read and when the hybrid is needed.
- [ ] Explain why redundancy does not protect against correlated software bugs.
- [ ] Name the three maintainability principles and give one practice for each.
<!-- explainer:end -->

## In this section (study in order)

1. [Describing Load and Performance](<Load_and_Performance_Metrics/_index.md>) (L4 Advanced)

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Distributed Systems, Scalability, and Designing for Failure
- *Designing Data-Intensive Applications* — Martin Kleppmann — Reliable, Scalable, and Maintainable Applications › Scalability
- *Designing Data-Intensive Applications* — Martin Kleppmann — Reliable, Scalable, and Maintainable Applications › Approaches for Coping with Load
- *Designing Data-Intensive Applications* — Martin Kleppmann — Reliable, Scalable, and Maintainable Applications › Chapter 1. Reliable, Scalable, and Maintainable Applications
- *Designing Data-Intensive Applications* — Martin Kleppmann — Reliable, Scalable, and Maintainable Applications › How Important Is Reliability?

