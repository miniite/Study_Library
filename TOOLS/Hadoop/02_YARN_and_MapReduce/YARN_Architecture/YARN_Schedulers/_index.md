---
id: "TOOLS/Hadoop/02_YARN_and_MapReduce/YARN_Architecture/YARN_Schedulers"
level: 5
type: "curriculum"
curriculum: "data_engineering"
---

# YARN Schedulers

**L5 Expert** · [↑ YARN Architecture](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** YARN offers three schedulers — FIFO, Capacity and Fair — which decide, queue by queue, who receives containers when demand exceeds supply, and add features such as elasticity, preemption, delay scheduling and dominant-resource fairness.

### 1. The big picture
Resource requests would be granted immediately in an ideal world; real clusters are oversubscribed. The scheduler inside the resource manager applies a policy to decide which waiting request gets the next free container. No policy is "best", so YARN offers a choice, selected by `yarn.resourcemanager.scheduler.class`. The Capacity Scheduler is the Apache default (the Fair Scheduler is the default in some distributions such as CDH).

This lesson builds on YARN architecture (requests, containers, locality) and prepares for cluster tuning: queue design is usually the biggest lever an administrator has over fairness and latency in a shared Hadoop cluster.

### 2. Core concepts and vocabulary
- **Queue (pool)** — a named share of the cluster; hierarchical (`root.dev.eng`).
- **FIFO scheduler** — one queue, strict submission order.
- **Capacity Scheduler** — queues with guaranteed fractions of capacity; FIFO within a queue.
- **Fair Scheduler** — dynamically balances resources among running apps/queues according to weights.
- **Queue elasticity** — a queue may use idle capacity beyond its share.
- **Maximum capacity** — hard ceiling on a queue's elastic use.
- **Preemption** — killing containers of over-share queues to serve under-share ones.
- **Delay scheduling** — waiting briefly for a node-local slot before relaxing locality.
- **Scheduling opportunity** — one node-manager heartbeat offering free resources.
- **Dominant resource fairness (DRF)** — fairness measured on each user's dominant resource (CPU or memory share).

### 3. How it works, step by step
**FIFO vs Capacity vs Fair, with a big job followed by a small one**
1. *FIFO*: the big job takes everything; the small job waits until it finishes.
2. *Capacity*: a dedicated queue is reserved for small jobs; they start immediately, but reserved capacity may sit idle and the big job finishes later than under FIFO.
3. *Fair*: the big job gets the whole cluster until the small job arrives; as containers free up, the small job gets half; when it ends, the big job reclaims everything. High utilization *and* timely small jobs, with some lag while containers drain.

**Fair share example.** Users A and B have queues. A alone gets 100%. When B submits, each converges to 50%. If B starts a second job, B's two jobs share B's half (25% each) while A keeps 50%.

**Placing an app in a queue (Fair).** Rules are tried in order: `specified` (queue named by the submitter, if it exists) → `primaryGroup` → `default`. Without a policy: `specified`, then `user` (a queue per user, created on demand).

### 4. Worked example
Capacity Scheduler, `capacity-scheduler.xml`:
```xml
<property><name>yarn.scheduler.capacity.root.queues</name><value>prod,dev</value></property>
<property><name>yarn.scheduler.capacity.root.dev.queues</name><value>eng,science</value></property>
<property><name>yarn.scheduler.capacity.root.prod.capacity</name><value>40</value></property>
<property><name>yarn.scheduler.capacity.root.dev.capacity</name><value>60</value></property>
<property><name>yarn.scheduler.capacity.root.dev.maximum-capacity</name><value>75</value></property>
```
`prod` is guaranteed 40%, `dev` 60% and may elastically grow to 75% when `prod` is idle, but not beyond; `dev` is split between `eng` and `science`.

Fair Scheduler, `fair-scheduler.xml`:
```xml
<allocations>
  <defaultQueueSchedulingPolicy>fair</defaultQueueSchedulingPolicy>
  <queue name="prod"><weight>40</weight><schedulingPolicy>fifo</schedulingPolicy></queue>
  <queue name="dev"><weight>60</weight><queue name="eng"/><queue name="science"/></queue>
  <queuePlacementPolicy>
    <rule name="specified" create="false"/>
    <rule name="primaryGroup" create="false"/>
    <rule name="default" queue="dev.eng"/>
  </queuePlacementPolicy>
</allocations>
```
Here `prod` runs its jobs serially (FIFO) so each finishes quickly, while `prod` vs `dev` is still divided 40:60 by weight (weights 2:3 would be the same). Unconfigured queues (e.g., dynamically created) have weight 1 — remember them when choosing weights.

### 5. Under the hood
- **Capacity Scheduler:** hierarchical queues each with `capacity` (guaranteed) and `maximum-capacity`. A job never exceeds its queue's share *unless* idle capacity is available, in which case another job in the same queue may borrow it (elasticity). By default it does **not** kill containers to reclaim: an under-capacity queue regains its share only as other queues' containers finish — mitigate with a maximum capacity (at the cost of elasticity). Within a queue, ordering is FIFO.
- **Fair Scheduler:** weights decide fair shares; queue policies may be `fair`, `fifo` or `drf`. *Minimum share* isn't a hard limit but orders allocation (the queue furthest below its minimum goes first) and drives preemption.
- **Preemption (Fair):** off by default; `yarn.scheduler.fair.preemption=true` plus timeouts. If a queue stays below its minimum share for `minSharePreemptionTimeout`, or below half of its fair share (threshold 0.5) for `fairSharePreemptionTimeout`, the scheduler kills containers of queues above their share. This makes start times predictable but wastes work since killed tasks re-execute.
- **Delay scheduling:** instead of relaxing locality at the first free slot, wait up to N scheduling opportunities (NM heartbeats, ~1/s) — a few seconds is enough to raise node-local hit rates dramatically. Capacity: `yarn.scheduler.capacity.node-locality-delay` (an integer). Fair: `yarn.scheduler.fair.locality.threshold.node/rack` expressed as a fraction of cluster nodes.
- **DRF:** with several resource types, compare users on the dominant share. Cluster 100 CPUs / 10 TB. A's container is (2 CPU, 300 GB) → (2%, 3%) → memory dominant 3%. B's is (6 CPU, 100 GB) → (6%, 1%) → CPU dominant 6%. B's dominant share per container is twice A's, so under fair sharing B gets half as many containers. By default only memory is considered; enable with `yarn.scheduler.capacity.resource-calculator=...DominantResourceCalculator` or `drf` policy in Fair.

### 6. Performance and design implications
- Queues should model *organisations or workload classes* (prod/ad hoc/ETL), not individual jobs.
- Capacity gives predictable guarantees and chargeback; Fair maximises utilisation with minimal reservation.
- Preemption is a latency vs waste trade-off; use timeouts that don't kill work needlessly.
- Reduce slow start (default 5% maps done) can let big jobs' reducers hold containers while maps are still running; raising it improves throughput.
- Without DRF, a CPU-heavy job can be starved of cores because only memory is counted.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `yarn.resourcemanager.scheduler.class` | Choose scheduler | Cluster setup |
| `yarn.scheduler.capacity.<q>.capacity` / `maximum-capacity` | Guaranteed / ceiling share | Capacity queues |
| `yarn.scheduler.fair.allocation.file` | Fair queue definitions | Fair setup |
| `yarn.scheduler.fair.preemption` | Enable preemption | Strict SLAs |
| `yarn.scheduler.capacity.node-locality-delay` | Delay scheduling (opportunities) | Improve locality |
| `yarn.scheduler.fair.user-as-default-queue` / `allow-undeclared-pools` | Queue-per-user control | Prevent queue sprawl |
| `yarn rmadmin -refreshQueues` | Apply queue changes without restart | Tuning |
| `mapreduce.job.reduce.slowstart.completedmaps` | Reduce start threshold | Big jobs |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| FIFO for a single-user, single-job cluster | Any shared cluster |
| Capacity for organisational guarantees and hierarchical quotas | Needing instant reclaim without preemption |
| Fair for mixed long/short workloads with minimal reservation | Needing hard capacity guarantees |
| DRF for CPU- and memory-diverse apps | Single-resource workloads (adds little) |
| Preemption for latency SLAs | Long non-restartable tasks |

### 9. Common mistakes
- Running the FIFO scheduler on a shared cluster → one big job blocks everyone → use Capacity or Fair.
- Setting no maximum capacity → one queue elastically swallows the cluster and others wait → set a ceiling.
- Enabling preemption without timeouts → nothing happens (timeouts unset by default) → set at least one.
- Per-user queues by default plus weights ignoring them → skewed shares → disable on-the-fly pools or account for them.
- Forgetting memory-only accounting → cores oversubscribed → enable DRF.

### 10. 🎯 Interview kit
**30-second answer:** FIFO runs jobs in order; Capacity reserves guaranteed fractions per queue with FIFO inside and elastic borrowing; Fair continuously rebalances resources among running jobs by weight, optionally preempting. Delay scheduling and DRF refine locality and multi-resource fairness.

**Structure a longer answer:** clarify workloads/SLAs → compare the three policies on one example → preemption/elasticity trade-offs → recommend a queue layout.

**Likely follow-ups:**
- *Capacity vs Fair?* → reservation vs dynamic sharing.
- *Why wait before relaxing locality?* → a short wait greatly increases node-local placements.
- *What is DRF?* → equalise dominant resource shares.
- *How do you make a high-priority queue start quickly?* → min share + preemption timeout or dedicated capacity.

**Signals interviewers look for:** concrete example of each policy and an awareness that preemption wastes work.

### 11. Check yourself
- [ ] Draw utilisation timelines for FIFO, Capacity and Fair with one big and one small job.
- [ ] Write a Capacity config with 40/60 queues and a maximum.
- [ ] Explain queue elasticity and why Capacity doesn't reclaim by killing.
- [ ] Compute DRF shares for the A/B example.
- [ ] State both Fair preemption timeouts and what triggers them.
<!-- explainer:end -->

## References

- *Hadoop: The Definitive Guide* — Tom White — YARN › Delay Scheduling
- *Hadoop: The Definitive Guide* — Tom White — YARN › Dominant Resource Fairness
- *Hadoop: The Definitive Guide* — Tom White — YARN › Enabling the Fair Scheduler
- *Hadoop: The Definitive Guide* — Tom White — YARN › Scheduling in YARN
- *Hadoop: The Definitive Guide* — Tom White — YARN › Note
- *Hadoop: The Definitive Guide* — Tom White — YARN › Capacity Scheduler Configuration
- *Hadoop: The Definitive Guide* — Tom White — YARN › Preemption
- *Hadoop: The Definitive Guide* — Tom White — C
- *Hadoop: The Definitive Guide* — Tom White — Setting Up a Hadoop Cluster › Job scheduler
- *Hadoop: The Definitive Guide* — Tom White — YARN › Queue placement

