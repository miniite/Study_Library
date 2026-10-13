---
id: "TOOLS/Hadoop/02_YARN_and_MapReduce"
level: 3
type: "curriculum"
curriculum: "data_engineering"
---

# YARN and MapReduce on Hadoop

**L3 Practitioner** · [↑ Hadoop](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** In Hadoop, YARN is the cluster's resource manager that hands out containers, and MapReduce is the original batch programming model that runs on top of it as just one YARN application among many.

### 1. The big picture
Hadoop has two layers that matter for compute: **storage** (HDFS) and **processing**. Processing itself is split into a *resource management* layer, **YARN**, and *frameworks* running on it — MapReduce first, but also Spark, Tez, Hive-on-Tez, HBase, and others. Higher-level tools like Hive, Pig and Crunch don't talk to YARN directly; they compile into MapReduce, Spark or Tez jobs.

![YARN applications](<../../../_assets/Hadoop_The_Definitive_Guide_Tom_White/hddg_0401.png>)

This split was introduced in Hadoop 2. In Hadoop 1 ("MapReduce 1") a single `jobtracker` did scheduling *and* monitoring and could not scale beyond roughly 4,000 nodes / 40,000 tasks. YARN separates the jobs: a cluster-wide **resource manager** only allocates resources, and a per-application **application master** drives its own job. The children of this topic cover YARN's architecture and schedulers, how MapReduce programs are developed, how they fail and recover, and how to administer the cluster.

### 2. Core concepts and vocabulary
- **MapReduce** — programming model: a *map* function turns input records into key-value pairs; the framework sorts and groups by key; a *reduce* function aggregates each group.
- **Job / task / task attempt** — a job has many map and reduce tasks; a task may be tried more than once.
- **Input split** — the chunk of input processed by one map task (usually one HDFS block).
- **YARN resource manager (RM)** — one per cluster; allocates containers.
- **Node manager (NM)** — one per node; launches and monitors containers.
- **Container** — a bundle of resources (memory, vcores) on one node in which a process runs.
- **Application master (AM)** — per-application process that negotiates containers and manages tasks; for MapReduce it is `MRAppMaster`.
- **Job history server / timeline server** — keeps information about completed applications.
- **Uber task** — a small job run entirely inside the AM's JVM.

### 3. How it works, step by step
**Programming model (the weather example).** Input lines contain a year and a temperature among many fields.
1. *Map*: input key = byte offset (ignored), value = line → emit `(year, temperature)`, dropping missing/suspect values.
2. *Shuffle/sort* (by the framework): group by key → `(1949, [111, 78])`, `(1950, [0, 22, -11])`.
3. *Reduce*: take the maximum of each list → `(1949, 111)`, `(1950, 22)`.

![MapReduce logical data flow](<../../../_assets/Hadoop_The_Definitive_Guide_Tom_White/hddg_0201.png>)

**Running it on YARN.**
1. The client submits the job; the RM allocates a container and starts the `MRAppMaster` there via a node manager.
2. The AM reads the input splits computed by the client, creates one map task per split and `mapreduce.job.reduces` reduce tasks.
3. If the job is small (default: <10 mappers, 1 reducer, input smaller than one block) and *uber mode* is enabled (`mapreduce.job.ubertask.enable`), it runs inside the AM's own JVM.
4. Otherwise the AM asks the RM for containers (with locality preferences), launches tasks through node managers, tracks progress and retries failures.
5. At the end the output committer commits output and the job history is written to HDFS.

### 4. Worked example
```bash
# submit the classic word count example and watch it
yarn jar hadoop-mapreduce-examples.jar wordcount /data/books /out/wc
yarn application -list                    # the application and its state
yarn logs -applicationId application_17..._0007 | head
hdfs dfs -cat /out/wc/part-r-00000 | sort -k2 -nr | head
```
Open the resource manager UI at `http://<rm-host>:8088/`: the *Cluster Metrics* section shows running applications and memory totals; the main table lists applications (the RM keeps up to 10,000 completed ones in memory, `yarn.resourcemanager.max-completed-applications`). Job history for a finished job is kept in HDFS (`mapreduce.jobhistory.done-dir`) for a week and is viewable in the history server or with `mapred job -history`.

### 5. Under the hood
- **Fine-grained resources.** In MapReduce 1, fixed *slots* were either too big (waste) or too small (failure). YARN containers request arbitrary memory and vcores. A node manager's pool is `yarn.nodemanager.resource.memory-mb` (default 8192, normally too low) and `yarn.nodemanager.resource.cpu-vcores`; set them to node capacity minus what datanode, node manager and OS need (each Hadoop daemon uses roughly 1 GB).
- **Two memory knobs per task:** the YARN *container* size (`mapreduce.map.memory.mb`, `mapreduce.reduce.memory.mb`) and the JVM *heap* inside it (`*.java.opts`); heap must be smaller than the container.
- **CPU is tracked, not enforced** by default; actual CPU limiting needs `LinuxContainerExecutor` with cgroups.
- **Component mapping:** jobtracker → resource manager + application master + timeline server; tasktracker → node manager; slot → container.
- **Multi-tenancy:** different MapReduce versions and entirely different frameworks run on one cluster. The shuffle handler and history server must still be upgraded cluster-wide.
- **Spark on YARN** has two modes: *client* (driver on the submitting machine; interactive shells) and *cluster* (driver in the AM; production, with retry if the AM fails).
- **Reduce slow start:** reducers are scheduled after 5% of maps finish by default (`mapreduce.job.reduce.slowstart.completedmaps`); raise it (e.g., 0.80) for big jobs so idle reducers don't hog containers.

### 6. Performance and design implications
- MapReduce is throughput-oriented: each job writes intermediate data to local disk and output to HDFS; chains of jobs pay that cost repeatedly (`ChainMapper/ChainReducer` can fuse map steps).
- Locality matters: tasks run where the block lives when possible.
- Task start-up in new containers is costly for tiny jobs — that's why uber mode exists.
- Spark's DAG engine and in-memory caching beat MapReduce for iterative and interactive work, often by an order of magnitude.
- Short-circuit local reads (`dfs.client.read.shortcircuit`) let a local client read blocks directly from disk.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `yarn application -list/-kill/-status` | Inspect or stop apps | Operations |
| `yarn logs -applicationId` | Aggregated container logs | Debugging |
| `mapred job -history` | Show a finished job's history | Post-mortem |
| `yarn.nodemanager.resource.memory-mb` | Memory pool per node | Cluster sizing |
| `yarn.nodemanager.resource.cpu-vcores` | Cores per node | Cluster sizing |
| `mapreduce.map.memory.mb` / `.java.opts` | Container vs heap | Task OOM / waste |
| `mapreduce.job.reduces` | Number of reducers | Tuning |
| `mapreduce.job.ubertask.enable` | Run tiny jobs in the AM | Many small jobs |
| `mapreduce.job.reduce.slowstart.completedmaps` | When reducers may start | Large jobs |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| MapReduce for huge, simple batch jobs with plenty of disk | Iterative/interactive workloads (use Spark/Tez) |
| YARN to share one cluster between frameworks | Single-purpose small clusters where simplicity is enough |
| Client mode for Spark development | Client mode for production (driver outside cluster) |
| Larger containers for memory-hungry tasks | Oversizing — fewer concurrent containers |
| Uber mode for tiny jobs | Enabling it for jobs with real parallelism |

### 9. Common mistakes
- Heap bigger than container → container killed by the node manager → keep `-Xmx` ≈ 75–80% of container size.
- Leaving `memory-mb` at 8 GB on big machines → idle hardware → set to real capacity.
- Assuming CPU limits are enforced → noisy neighbours → enable cgroups.
- Confusing old/new MapReduce API with MapReduce 1/2 implementations → wrong upgrade plans → API is client-side, implementation is runtime.
- Too many tiny map tasks → scheduler overhead → combine input or use bigger splits.

### 10. 🎯 Interview kit
**30-second answer:** YARN is Hadoop's resource manager: a resource manager allocates containers across node managers and each application brings its own application master. MapReduce is one such application — map, shuffle/sort, reduce — scheduled by YARN.

**Structure a longer answer:** clarify (batch or interactive? how many tenants?) → mechanism (RM / NM / AM / containers, map-shuffle-reduce) → trade-offs (disk-bound, locality, YARN vs MR1) → scenario (nightly aggregation on a shared cluster).

**Likely follow-ups:**
- *Why YARN over MR1?* → jobtracker bottleneck, rigid slots, single framework.
- *Who tracks task progress?* → the per-application AM.
- *What is a container?* → a resource slice on a node where a task process runs.
- *What is uber mode?* → run small jobs in the AM's JVM.
- *Where does job history live?* → HDFS, served by the history/timeline server.

**Signals interviewers look for:** you separate resource management from application logic and can name which component does what.

### 11. Check yourself
- [ ] Trace weather max-temperature through map, shuffle and reduce.
- [ ] Draw RM, NM, AM and containers for one job submission.
- [ ] Map every MR1 component to its YARN equivalent.
- [ ] Explain container memory vs JVM heap.
- [ ] State when a job becomes an uber task.
<!-- explainer:end -->

## In this section (study in order)

1. [YARN Architecture](<YARN_Architecture/_index.md>) (L4 Advanced)
2. [Developing MapReduce Applications](<MapReduce_Development/_index.md>) (L4 Advanced)
3. [MapReduce Failures and Speculative Execution](<MapReduce_Failures/_index.md>) (L5 Expert)
4. [Cluster Setup and Administration](<Cluster_Setup_and_Administration/_index.md>) (L4 Advanced)

## References

- *Hadoop: The Definitive Guide* — Tom White — YARN › Chapter 4. YARN
- *Hadoop: The Definitive Guide* — Tom White — YARN › YARN Compared to MapReduce 1
- *Hadoop: The Definitive Guide* — Tom White — Setting Up a Hadoop Cluster › CPU settings in YARN and MapReduce
- *Hadoop: The Definitive Guide* — Tom White — MapReduce › Chapter 2. MapReduce
- *Hadoop: The Definitive Guide* — Tom White — Setting Up a Hadoop Cluster › Memory settings in YARN and MapReduce
- *Hadoop: The Definitive Guide* — Tom White — How MapReduce Works › Job Initialization
- *Hadoop: The Definitive Guide* — Tom White — Spark › Chapter 19. Spark
- *Hadoop: The Definitive Guide* — Tom White — MapReduce Features › MapReduce Library Classes
- *Hadoop: The Definitive Guide* — Tom White — MapReduce › Analyzing the Data with Hadoop
- *Hadoop: The Definitive Guide* — Tom White — Developing a MapReduce Application › The MapReduce Web UI

