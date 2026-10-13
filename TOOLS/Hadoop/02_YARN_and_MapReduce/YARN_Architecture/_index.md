---
id: "TOOLS/Hadoop/02_YARN_and_MapReduce/YARN_Architecture"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# YARN Architecture

**L4 Advanced** · [↑ YARN and MapReduce on Hadoop](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A YARN application is a client plus a per-application master that asks a cluster-wide resource manager for containers, which node managers then launch and watch.

### 1. The big picture
YARN (Yet Another Resource Negotiator) is Hadoop's cluster resource management system, introduced with Hadoop 2. Its job is deliberately narrow: *hand out slices of cluster capacity and keep track of them*. It knows nothing about maps, reduces, DAGs or RDDs. The framework-specific intelligence lives in the **application master** that each application brings.

This lesson explains the three kinds of processes (resource manager, node managers, application master), how a request for resources is expressed (including locality), how application lifetimes differ across frameworks, and what happens when a node manager fails. The next level down, schedulers, explains how the resource manager decides *who gets what*.

### 2. Core concepts and vocabulary
- **Resource manager (RM)** — one per cluster (can be HA); owns global resource view and scheduling.
- **Node manager (NM)** — per-node daemon; launches containers, enforces limits, reports usage and health by heartbeat.
- **Container** — constrained set of resources (memory, CPU) in which an application-specific process runs; a Unix process or Linux cgroup.
- **Application master (AM)** — first container of an application; negotiates more containers and coordinates the work.
- **Resource request** — memory/CPU per container, count, and a locality constraint.
- **Locality constraint** — specific node, specific rack, or anywhere (off-rack).
- **Application lifespan** — one app per job (MapReduce), one per session/workflow (Spark), or long-running shared (Impala proxy, Slider).
- **Blacklisting** — AM stops using a node that causes many task failures.

### 3. How it works, step by step
![How YARN runs an application](<../../../../_assets/Hadoop_The_Definitive_Guide_Tom_White/hddg_0402.png>)

1. A client contacts the RM and asks it to run an application master.
2. The RM finds a node manager with capacity and starts the AM in a container there.
3. The AM does application-specific work. It may simply compute inside its own container, or request more containers from the RM.
4. The RM grants containers (subject to the scheduler); the AM contacts the node managers to launch processes in them.
5. Tasks run and report to the AM using the application's *own* protocol — YARN does not provide client/AM/task communication.
6. When done, the AM unregisters; the RM reclaims all containers.

**Resource requests and locality.** A request says "N containers of X MB / Y vcores at location L". For a map task on an HDFS block, the AM asks for a container on one of the three nodes holding replicas; if that is not possible YARN can relax to a node on the same rack, then anywhere. Requests can be made at any time: Spark asks for a fixed number of executors up front; MapReduce asks for map containers first, reduce containers later, and additional containers when tasks fail and must be retried.

### 4. Worked example
```bash
yarn node -list                       # node managers and running containers
yarn application -list -appStates RUNNING
yarn application -status application_1700000000000_0042
yarn container -list appattempt_1700000000000_0042_000001
yarn logs -applicationId application_1700000000000_0042 -containerId container_..._000001
```
In a MapReduce job you will see `container_..._000001`: it is the application master; all following container numbers are tasks. If the node running a mapper dies you will see the AM request a replacement container and a new *attempt* number for that task.

### 5. Under the hood
**Application lifespans.**
1. *One application per job* — simple isolation; MapReduce.
2. *One per workflow or session* — containers reused and intermediate data cached across jobs; Spark.
3. *Long-running shared service* — an always-on AM (Slider, Impala's proxy) removes start-up latency for low-latency queries.

**Node manager failure.** A failed or hung NM stops heartbeating. After 10 minutes of silence (`yarn.resourcemanager.nm.liveness-monitor.expiry-interval-ms`) the RM drops it from the pool. Tasks and AMs on it are recovered by the usual retry mechanisms. In addition, the AM re-runs *completed* map tasks that had run on that node if their job is unfinished, because their intermediate output lives on that node's local disk and reducers may no longer reach it. The AM may also **blacklist** a node: for MapReduce, once more than three tasks of a job fail on one NM (`mapreduce.job.maxtaskfailures.per.tracker`) tasks are rescheduled elsewhere, even when the node isn't "dead".

**Design theme — frequent faults.** MapReduce was designed assuming tasks are killed often (preemption, failure). In environments where tasks are rarely terminated, its decisions to write all intermediate state to disk are a worse fit — that observation motivated engines like Spark and Tez.

**Separation of code and state.** YARN is a deployment environment for application code; durable state lives in HDFS or other stores — the same separation of logic from state that web architectures use.

### 6. Performance and design implications
- Always give locality hints; otherwise network bandwidth becomes the bottleneck.
- AM overhead matters: each application pays container start-up. For many short jobs prefer session-based frameworks or uber mode.
- The RM is a single scheduler thread-of-control: very large clusters (10k nodes / 100k tasks is the design target) rely on AM-side tracking so the RM stays light.
- Heartbeats (default 1 per second per NM) are the pacing signal for scheduling decisions.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `yarn node -list -all` | List NMs and state | Health check |
| `yarn application -kill` | Kill an application | Runaway job |
| `yarn rmadmin -refreshNodes` | Re-read include/exclude | Commission/decommission |
| `yarn.resourcemanager.nm.liveness-monitor.expiry-interval-ms` | Time to declare NM lost | Failure handling |
| `mapreduce.job.maxtaskfailures.per.tracker` | Blacklist threshold | Flaky nodes |
| `yarn.nodemanager.container-executor.class` | Container isolation (Linux/cgroups) | Enforce CPU |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| App-per-job for isolation and simplicity | Thousands of short jobs (AM start-up dominates) |
| Session-per-user app for interactive reuse | Strict per-job accounting |
| Long-running AM for low-latency services | Sharing without quotas (one tenant hogs) |
| Fine-grained container sizes | Guessing sizes – undersize fails, oversize wastes |
| Blacklisting flaky nodes | Cluster so small that blacklisting leaves no capacity |

### 9. Common mistakes
- Thinking YARN schedules tasks → it only grants containers; the AM places tasks → debug scheduling in AM logs too.
- Ignoring completed-task re-runs after a node loss → surprised by long recovery → expect map re-execution.
- Expecting YARN to carry data between client, AM, tasks → none provided → application RPC.
- Requesting only node-local containers with no relaxation → starvation → allow rack/off-rack fallback or delay scheduling.

### 10. 🎯 Interview kit
**30-second answer:** The resource manager allocates containers across node managers; each application has its own application master which requests containers with locality hints and runs tasks in them. YARN is generic — MapReduce, Spark and others are applications.

**Structure a longer answer:** clarify → components → run of one application → failure handling → scenario (node loss mid-job).

**Likely follow-ups:**
- *What happens if the AM dies?* → RM restarts it (limited attempts); MapReduce can recover completed tasks.
- *What if a node manager dies?* → 10 min expiry, task retries, completed maps re-run.
- *How does locality work?* → node → rack → off-rack relaxation.
- *Application per job vs per session?* → isolation vs reuse and caching.

**Signals interviewers look for:** you don't claim YARN knows about tasks; you can narrate the failure cases.

### 11. Check yourself
- [ ] List the six steps of a YARN application run.
- [ ] Explain what a resource request contains.
- [ ] Why are finished map tasks re-run after a node failure?
- [ ] Describe three different application lifespans with an example each.
<!-- explainer:end -->

## In this section (study in order)

1. [YARN Schedulers](<YARN_Schedulers/_index.md>) (L5 Expert)

## References

- *Hadoop: The Definitive Guide* — Tom White — YARN › Application Lifespan
- *Hadoop: The Definitive Guide* — Tom White — YARN › Anatomy of a YARN Application Run
- *Hadoop: The Definitive Guide* — Tom White — How MapReduce Works › Node Manager Failure
- *Hadoop: The Definitive Guide* — Tom White — YARN › Resource Requests
- *Designing Data-Intensive Applications* — Martin Kleppmann — Comparing Hadoop to Distributed Databases › Designing for frequent faults
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Choosing Technologies Across the Data Engineering Lifecycle › Our Advice
- *Designing Data-Intensive Applications* — Martin Kleppmann — Designing Applications Around Dataflow › Separation of application code and state

