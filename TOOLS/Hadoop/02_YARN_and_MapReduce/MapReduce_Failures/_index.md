---
id: "TOOLS/Hadoop/02_YARN_and_MapReduce/MapReduce_Failures"
level: 5
type: "curriculum"
curriculum: "data_engineering"
---

# MapReduce Failures and Speculative Execution

**L5 Expert** · [↑ YARN and MapReduce on Hadoop](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A MapReduce job survives failures by retrying at the granularity of a single task attempt, restarting the application master from its history, rerunning lost map output, and (with HA) failing over the resource manager — while speculative execution hedges against slow tasks.

### 1. The big picture
In the real world user code is buggy, processes crash and machines die. A major reason people used Hadoop is that a job still completes. To understand how, consider the four entities that can fail — the **task**, the **application master**, the **node manager** and the **resource manager** — and the recovery rule for each. Add to that one performance mechanism, **speculative execution**, which deals with tasks that are not failing but are *slow*.

MapReduce's design choices follow from its original environment: mixed-use datacentres where low-priority batch tasks can be pre-empted at any moment, so tasks *will* be killed regularly. That is why work is retried per task and intermediate data is written eagerly to disk. MPP databases make the opposite choice (abort and re-run the whole query, keep data in memory) because their queries run for seconds, not hours.

### 2. Core concepts and vocabulary
- **Task attempt** — one execution of a task; a task may have several attempts.
- **Task timeout** — time without progress after which an attempt is failed (`mapreduce.task.timeout`, default 10 minutes).
- **Max attempts** — `mapreduce.map.maxattempts`, `mapreduce.reduce.maxattempts` (default 4 each).
- **Job recovery** — a new AM uses the job history to avoid rerunning finished tasks.
- **Blacklisting** — AM avoids nodes with repeated failures (not shared across applications).
- **Resource-manager state store** — ZooKeeper- or HDFS-backed record of running applications used by a standby RM.
- **Speculative execution** — launching a duplicate of a straggling task; the first to finish wins.
- **Idempotent task** — safe to run more than once with the same result.
- **OutputCommitter** — promotes a successful attempt's temporary output to the final location.

### 3. How it works, step by step
**Task failure**
1. User code throws → the task JVM reports the error to the AM and exits; or the JVM dies → node manager tells the AM; or the task hangs → AM sees no progress for the timeout and kills it.
2. The AM marks the *attempt* failed and frees the container.
3. It reschedules the task, preferably on another node.
4. After 4 failed attempts the task — and by default the whole job — fails. For jobs that can tolerate some bad data, `mapreduce.map.failures.maxpercent` / `mapreduce.reduce.failures.maxpercent` let a percentage of tasks fail without failing the job. A task *killed* (e.g., by speculative execution or preemption) does not count toward the attempt limit.

**Application-master failure**
1. The AM heartbeats to the RM; the RM notices a failure.
2. The RM starts a new AM in another container, up to `mapreduce.am.max-attempts` (default 2), which is capped by the cluster limit `yarn.resourcemanager.am.max-attempts` (default 2).
3. The new AM reads the job history to recover completed tasks (`yarn.app.mapreduce.am.job.recovery.enable`, default true) so they need not be rerun.
4. The client, whose cached AM address now times out, asks the RM for the new address — transparent to the user.

**Node-manager failure:** see the YARN architecture lesson — 10-minute heartbeat expiry, retries, re-run of completed maps whose output was on that node, per-job blacklisting after >3 task failures on a node.

**Resource-manager failure**
1. Default: a single point of failure — all running jobs fail.
2. With HA: an active/standby pair; running-application state sits in a store (ZooKeeper or HDFS). Node manager info is not stored because it is rebuilt from heartbeats; task state isn't stored because the AMs own it, so state is small.
3. ZooKeeper leader election (an embedded failover controller) promotes the standby, which restarts all AMs. That restart does *not* count against AM max-attempts, because the application did not fail on its own.
4. Clients and node managers retry both RMs round-robin until one is active.

### 4. Worked example
```bash
# tolerate up to 5% failed map tasks, raise attempts, and keep speculation for maps only
hadoop jar analytics.jar Job \
  -D mapreduce.map.maxattempts=6 \
  -D mapreduce.map.failures.maxpercent=5 \
  -D mapreduce.task.timeout=1800000 \
  -D mapreduce.map.speculative=true \
  -D mapreduce.reduce.speculative=false  /in /out
```
In Java, long-running mappers should *report progress* (call `context.progress()`, set a status or increment a counter) rather than disabling the timeout; with `mapreduce.task.timeout=0` a truly hung task never releases its container and the cluster slowly clogs.

### 5. Under the hood
**Speculative execution.** The AM tracks task progress. When a task runs noticeably slower than its siblings it launches a duplicate attempt on another node. Whichever finishes first is kept; the other is killed. Properties: `mapreduce.map.speculative`, `mapreduce.reduce.speculative`. It reduces *job latency* at the cost of *cluster throughput* since duplicate work is performed.
- Disable cluster-wide on busy clusters and enable per job; older versions were overly aggressive.
- Often disabled for reducers: a duplicate reducer must fetch all the same map output, increasing network traffic.
- Disable (or make idempotent) when tasks have external side effects — e.g., writing to a database. Writing to HDFS is safe because attempts write to distinct temp directories and the `OutputCommitter` promotes only the winner.

**Why not hide partial failure?** The sender in a distributed system cannot tell whether a failed call executed; retries must therefore be safe. Idempotent tasks plus commit-on-success make MapReduce's retry semantics simple.

**Cost-benefit:** fine-grained recovery slows fault-free runs slightly (disk writes), but for jobs large enough to expect at least one failure, rerunning the whole job would be far costlier.

### 6. Performance and design implications
- Long-running tasks that stay silent are killed at the timeout; progress reporting is mandatory.
- A bad record that crashes the mapper fails four attempts and the job — use skipping modes, counters/validation, or a failure percentage for dirty data.
- Straggling tasks (bad disk, noisy neighbour) can dominate job time; speculation attacks that tail.
- Frequent AM restarts indicate cluster instability; the recovery makes them cheap for MapReduce but not for frameworks without checkpointing.
- Because blacklists are per application, a persistently bad node keeps hurting new jobs until the admin removes it.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `mapreduce.task.timeout` | Hang detection | Long tasks without progress |
| `mapreduce.{map,reduce}.maxattempts` | Attempt limit (4) | Flaky environments |
| `mapreduce.{map,reduce}.failures.maxpercent` | Tolerated failed tasks | Dirty data |
| `mapreduce.am.max-attempts` / `yarn.resourcemanager.am.max-attempts` | AM retries (2) | Raise both |
| `yarn.app.mapreduce.am.job.recovery.enable` | AM recovery | Keep true |
| `mapreduce.{map,reduce}.speculative` | Speculation switches | Latency vs throughput |
| `mapreduce.job.maxtaskfailures.per.tracker` | Blacklist threshold | Flaky nodes |
| `yarn rmadmin -getServiceState` | RM HA state | Operations |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Speculation on for heterogeneous or unreliable nodes | Busy clusters where throughput matters, reducers, side-effecting tasks |
| Higher max attempts for transient infra problems | Deterministic bugs (retries just waste time) |
| Failure percentage for dirty, noisy data | Exact accounting jobs |
| RM HA with state store for production | Dev clusters |
| Progress reporting | Setting timeout to 0 |

### 9. Common mistakes
- Disabling the task timeout → hung tasks hold containers forever → report progress instead.
- Raising `mapreduce.am.max-attempts` only → the YARN cap still applies → raise the cluster limit too.
- Non-idempotent tasks with speculation on (e.g., writing to an external DB) → duplicate side effects → disable speculation or use idempotent writes.
- Treating a data-dependent crash as an infra failure → four wasted attempts → fix or skip the record.
- Assuming blacklist protects later jobs → bad node reused → decommission the node.

### 10. 🎯 Interview kit
**30-second answer:** Failures are handled by retrying task attempts (4 by default), restarting the AM (2 attempts) with job-history recovery, re-running lost map output after node loss, and running an active/standby RM with a state store. Speculative execution runs duplicates of slow tasks to cut the tail.

**Structure a longer answer:** list the four failing entities → recovery rule each → speculation → trade-offs (idempotency, throughput) → scenario (node dies at 90% of a job).

**Likely follow-ups:**
- *What if a task hangs?* → no progress for 10 min → killed and retried.
- *Why rerun completed map tasks after a node failure?* → output on local disk is lost.
- *When disable speculation?* → reducers, busy clusters, non-idempotent side effects.
- *Is the RM a SPOF?* → by default yes; HA fixes it with ZooKeeper election and a state store.

**Signals interviewers look for:** correct defaults, awareness of idempotency, and clean separation of the recovery layers.

### 11. Check yourself
- [ ] Walk through what happens when a reducer throws an exception four times.
- [ ] Explain why an RM restart of AMs doesn't count as a failed attempt.
- [ ] Describe the cost/benefit of speculation and list when to turn it off.
- [ ] State which state a standby RM needs and which it does not.
- [ ] Contrast MapReduce's fault handling with an MPP database's.
<!-- explainer:end -->

## References

- *Hadoop: The Definitive Guide* — Tom White — How MapReduce Works › Application Master Failure
- *Hadoop: The Definitive Guide* — Tom White — How MapReduce Works › Failures
- *Designing Data-Intensive Applications* — Martin Kleppmann — Comparing Hadoop to Distributed Databases › Designing for frequent faults
- *Hadoop: The Definitive Guide* — Tom White — MapReduce Types and Formats › Warning
- *Hadoop: The Definitive Guide* — Tom White — ZooKeeper › Chapter 21. ZooKeeper
- *Hadoop: The Definitive Guide* — Tom White — How MapReduce Works › Streaming environment variables
- *Hadoop: The Definitive Guide* — Tom White — How MapReduce Works › Note
- *Hadoop: The Definitive Guide* — Tom White — Crunch › Debugging
- *Hadoop: The Definitive Guide* — Tom White — E › F
- *Designing Data-Intensive Applications* — Martin Kleppmann — B › C

