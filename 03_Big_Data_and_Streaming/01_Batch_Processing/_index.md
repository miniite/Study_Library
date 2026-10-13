---
id: "03_Big_Data_and_Streaming/01_Batch_Processing"
level: 3
type: "curriculum"
curriculum: "data_engineering"
---

# Batch Processing

**L3 Practitioner** · [↑ Big Data and Streaming](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Batch processing runs a job over a bounded, immutable input, produces a new output and has no other side effects — the model behind Unix pipelines, MapReduce and Spark, and the simplest reliable way to process very large datasets on many machines.

### 1. The big picture

A *batch job* takes a fixed, finite dataset as input, reads all (or most) of it, and writes derived output. Nothing is served to a user while it runs; you care about **throughput** (how much per hour), not response time. This is an old idea — punched-card tabulating machines in the 1890 US census and IBM card sorters in the 1940s–50s worked this way — and MapReduce looks strikingly like those sorters.

This area walks from the single-machine story (Unix tools) to the distributed one (MapReduce on HDFS), then to the techniques every distributed batch engine relies on (sort-merge and broadcast joins, grouping, skew handling), the use of batch output (search indexes, key-value stores, derived data), and finally why dataflow engines such as Tez, Spark and Flink replaced raw MapReduce.

Batch sits between transaction processing (OLTP: look up a few records by key, low latency) and analytics (OLAP: scan many records, build reports). It resembles analytics because it scans a large share of the input, but its output is often not a report — it is another dataset: a model, an index, a denormalised table, a cache to load into a serving store.

### 2. Core concepts and vocabulary

- **Batch job** — computation over a bounded input that produces output and finishes; typically scheduled (hourly, daily).
- **Bounded vs unbounded data** — batch works on finite input; stream processing works on never-ending input.
- **Immutable input / no side effects** — a job does not modify its input; rerunning gives the same result, so failures are safe to retry.
- **MapReduce** — programming model with a *map* function (extract key/value from each record) and *reduce* function (process all values of one key), with a sort/shuffle between.
- **HDFS** — Hadoop Distributed File System; shared-nothing, a daemon on every machine plus a central NameNode tracking which blocks live where (open-source reimplementation of GFS).
- **Shared-nothing** — machines with their own disks connected by an ordinary network, in contrast to shared-disk NAS/SAN appliances.
- **Data locality** — scheduling computation on a machine that stores a replica of the input ("put the computation near the data").
- **Shuffle** — redistributing intermediate key-value pairs so that all records with the same key land on one reducer.
- **Dataflow engine** — a successor to MapReduce (Tez, Spark, Flink) that models a job as a graph of operators instead of fixed map/reduce stages.
- **Derived data** — datasets computed from other datasets (indexes, aggregates, ML features).

### 3. How it works, step by step

1. **Read** input files from a distributed filesystem or object store; split them into records.
2. **Map**: call user code per record, emitting zero or more key-value pairs.
3. **Shuffle and sort**: partition pairs by hash of key and sort them, so equal keys are adjacent at one reducer.
4. **Reduce**: call user code once per key with an iterator over its values; write output files.
5. **Chain** jobs when needed: the output directory of one becomes the input of the next (e.g., count, then sort by count).
6. **Publish**: load results into a serving system or leave them as files for downstream jobs.

| Aspect | OLTP | Analytics / warehouse | Batch workflow |
|---|---|---|---|
| Access pattern | Few records by key (index) | Large scans, aggregates | Full scans of large input |
| Output | Records to user | Report, chart | Derived dataset |
| Latency goal | ms | seconds–minutes | minutes–hours |
| Failure handling | Transactions, retries | Query retry | Rerun task or job |

### 4. Worked example

The same question — "what are the five most popular pages?" — at three scales.

```bash
# One machine: Unix pipeline over an access log
awk '{print $7}' access.log | sort | uniq -c | sort -rn | head -n 5
```

```text
# MapReduce view of the same pipeline
map:    record (log line) -> (url, 1)            # awk '{print $7}'
shuffle+sort: group equal urls together          # sort
reduce: (url, [1,1,1,...]) -> (url, count)       # uniq -c
job 2:  sort by count descending, take top 5     # sort -rn | head
```

```sql
-- What a distributed SQL engine runs underneath (Hive / Spark SQL / BigQuery)
SELECT url, COUNT(*) AS hits
FROM access_log
GROUP BY url
ORDER BY hits DESC
LIMIT 5;
```

A SQL `GROUP BY user_id` on a petabyte table becomes one map task per block (each counting its own block — "embarrassingly parallel"), a hash-based shuffle that sends each user ID to exactly one node, and a reduce step that sums the partial counts. Scan throughput grows roughly linearly with the number of nodes.

### 5. Under the hood

**Why MapReduce scales linearly.** Data is partitioned and map/reduce are pure functional primitives that run on separate partitions. Double the data and a job takes twice as long; double the cluster too and it takes the same time. Many SQL queries do not have that property.

**Fault tolerance by re-execution.** Because mapper and reducer are side-effect-free functions of immutable input, the framework simply reruns any failed or killed task; the application logic never sees partial failures. Output is written once, sequentially, and only becomes visible when the task completes. This makes MapReduce robust on unreliable multi-tenant clusters with frequent task preemption.

**Where it hurts.** Every MapReduce stage writes its full output to the distributed filesystem (with replication) and the next job starts a fresh JVM per task. Intermediate state is *materialised*, which is slow for chains of jobs and for iterative algorithms (machine learning), and there is no shared notion of indexes — a job always scans whole inputs. Some workloads run orders of magnitude faster on other engines.

**Beyond MapReduce.** Tez and Spark treat the whole workflow as a DAG of operators (a generalisation of map and reduce), pipeline operators in memory, reuse JVMs, avoid unnecessary sorts and keep intermediate state in memory or local disk. Pig, Hive and Cascading workflows can move from MapReduce to Tez or Spark by changing configuration. YARN, introduced in Hadoop 2, separated cluster resource management from MapReduce, enabling interactive SQL (Impala, Hive on Tez), iterative processing (Spark), streaming (Storm, Samza, Spark Streaming) and search (Solr) on the same cluster.

### 6. Performance and design implications

- Treat inputs as immutable and write output to new locations (e.g., date-partitioned directories); this gives free reruns and easy debugging.
- Measure bottlenecks: input read rate, shuffle volume, number of materialisation steps. Reduce shuffle with combiners, map-side joins and column pruning.
- Prefer a few large input files/blocks to many tiny ones; each file costs task start-up and metadata work.
- Use high-level languages (Hive, Spark SQL, Pig) instead of raw MapReduce; they let the optimiser choose join algorithms and let you change engines later.
- Plan for retries: tasks must be idempotent and must not write directly into live serving systems.
- Batch latency is bounded by the slowest task (stragglers and skewed keys) — see *Skew Handling*.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `hadoop jar job.jar Class in out` | Submits a MapReduce job | Running packaged jobs |
| `hdfs dfs -ls / -put / -get / -cat` | Inspect and move HDFS files | Staging inputs, checking output |
| `yarn application -list / -kill` | Manage jobs on the cluster | Monitoring, stopping runaway jobs |
| `spark-submit` | Runs a Spark batch application | Dataflow-engine batch jobs |
| `EXPLAIN` / `df.explain()` | Shows the planned operators and join strategy | Verifying broadcast vs shuffle |
| Number of reducers (`mapreduce.job.reduces`) | Sets output parallelism | Tuning shuffle fan-out |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| Large scans, backfills, model training, building indexes/aggregates | You need answers in seconds for a user (use an interactive engine or serving store) |
| You need rerunnable, deterministic transformations | Data arrives continuously and results must be fresh within seconds (streaming) |
| Single-machine Unix tools for gigabytes | Data exceeds disk/memory of one machine or must be processed in parallel |
| Raw MapReduce only to learn or maintain legacy | New development — use Spark, Hive on Tez or a warehouse |
| Dataflow engine for chains and iterations | Cluster is tiny; framework overhead dominates |

### 9. Common mistakes

- Writing job output in place over live data → partial results visible and no rollback → write to a new path and swap.
- Using raw MapReduce for complex logic → thousands of lines and hand-built joins → use Hive/Spark SQL.
- Running a full-table scan job to look up a few keys → hours for a millisecond task → use an index or serving store.
- Ignoring data layout (tiny files, wrong partitioning) → slow jobs → compact and partition on query keys.
- Assuming batch output equals a report → missing the "derived dataset" use cases (search indexes, key-value stores).

### 10. 🎯 Interview kit

**30-second answer:** Batch processing runs deterministic jobs over bounded, immutable input and writes new output. MapReduce splits that into map (extract key/value), shuffle/sort (group by key) and reduce (aggregate), running on a distributed filesystem with data locality and automatic task retry. Dataflow engines like Spark generalise it by chaining operators and keeping intermediate data in memory.

**Structure a longer answer:** clarify bounded vs unbounded → mechanics of map/shuffle/reduce → fault tolerance by re-execution → limits of MapReduce → modern engines and when to use streaming instead.

**Likely follow-ups:**
- *Why is MapReduce fault tolerant?* → Pure functions over immutable input; failed tasks are simply re-run.
- *Why did Spark/Tez replace it?* → Materialisation of intermediate state, JVM start-up per task, and no in-memory iteration.
- *What is data locality?* → Scheduling map tasks on machines that already hold the block.
- *How does batch differ from analytics queries?* → Output is often a dataset, not a report, and jobs are workflows.

**Signals interviewers look for:** you reason from immutability and idempotence; you can map SQL to map/shuffle/reduce; you know the cost of shuffles and materialisation.

### 11. Check yourself

- [ ] Explain why a failed MapReduce task can be retried without corrupting output.
- [ ] Sketch the MapReduce stages for `SELECT user_id, COUNT(*) FROM events GROUP BY user_id`.
- [ ] Name three weaknesses of MapReduce that dataflow engines address.
- [ ] Explain "put the computation near the data" and when it can't be achieved (e.g., object storage).
- [ ] Give two examples of batch output that is not a report.
<!-- explainer:end -->

## In this section (study in order)

1. [Unix Philosophy and Batch Processing](<Unix_Philosophy_and_Batch/_index.md>) (L4 Advanced)
2. [The MapReduce Programming Model](<MapReduce_Model/_index.md>) (L4 Advanced)
3. [Joins and Grouping in Batch Jobs](<Batch_Joins_and_Grouping/_index.md>) (L4 Advanced)
4. [Dataflow Engines beyond MapReduce](<Dataflow_Engines/_index.md>) (L4 Advanced)
5. [Batch Output and Derived Data](<Batch_Output_and_Derived_Data/_index.md>) (L4 Advanced)

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 10. Batch Processing
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Modeling Streaming Data › MapReduce
- *Designing Data-Intensive Applications* — Martin Kleppmann — MapReduce Job Execution
- *Designing Data-Intensive Applications* — Martin Kleppmann — MapReduce Job Execution › Distributed execution of MapReduce
- *Designing Data-Intensive Applications* — Martin Kleppmann — Beyond MapReduce
- *Designing Data-Intensive Applications* — Martin Kleppmann — The Unix Philosophy › MapReduce and Distributed Filesystems
- *Hadoop: The Definitive Guide* — Tom White — Meet Hadoop › Beyond Batch
- *Designing Data-Intensive Applications* — Martin Kleppmann — Map-Side Joins › The Output of Batch Workflows
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Modeling Streaming Data › Batch Transformations
- *Designing Data-Intensive Applications* — Martin Kleppmann — Beyond MapReduce › Dataflow engines

