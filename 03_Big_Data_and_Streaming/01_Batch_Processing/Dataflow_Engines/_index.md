---
id: "03_Big_Data_and_Streaming/01_Batch_Processing/Dataflow_Engines"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Dataflow Engines beyond MapReduce

**L4 Advanced** · [↑ Batch Processing](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Dataflow engines (Spark, Tez, Flink) run a whole workflow as one job — a graph of operators connected by repartition, pipe or broadcast links — keeping intermediate data in memory or local disk and recovering lost data by recomputation instead of writing every stage to HDFS.

### 1. The big picture

MapReduce forces every workflow into alternating map and reduce stages, with full materialisation of each stage's output on a replicated distributed filesystem. That is robust but slow: sorting happens everywhere whether needed or not, mappers re-read what reducers just wrote, and each stage waits for the previous one to finish completely.

Dataflow engines fix this by modelling the *entire workflow* as a single job. Functions (now called **operators**) are assembled flexibly, the scheduler sees all data dependencies, and intermediate state flows through memory, local disk or network pipes. The same processing code — Pig, Hive or Cascading — can often be switched from MapReduce to Tez or Spark by a configuration change. In the broader field, the dataflow model also underlies the "batch as a special case of streaming" philosophy (Beam, Flink, Spark).

### 2. Core concepts and vocabulary

- **Operator** — a user-defined function in a dataflow graph; generalises map and reduce; processes one record at a time.
- **Dataflow graph / DAG** — directed acyclic graph of operators and the data exchanges between them.
- **Shuffle (repartition and sort)** — exchange that groups records by key with sorting.
- **Hash repartition (no sort)** — partition identically but skip the sort; enough for partitioned hash joins.
- **Broadcast** — send an operator's whole output to every partition of the consumer (broadcast hash join).
- **Materialisation** — writing intermediate results to durable files.
- **Pipelined execution** — consuming operator output incrementally, without waiting for completion.
- **RDD (resilient distributed dataset)** — Spark's immutable partitioned collection with recorded lineage (how it was derived).
- **Lineage** — the record of input partitions and operators used to compute a dataset, enabling recomputation.
- **Checkpointing** — saving operator state periodically (Flink) so execution can resume.
- **Deterministic operator** — same input always yields the same output.
- **Execution engine** — what runs a query plan; e.g., Hive on `mr`, `tez` or Spark.

### 3. How it works, step by step

1. **Compose** the workflow: a high-level API or SQL is compiled into a DAG of operators.
2. **Plan data movement** per edge: sorted shuffle, hash repartition, or broadcast, depending on the operation (sort-merge join, partitioned hash join, broadcast join).
3. **Schedule** tasks with global knowledge; place consumers near producers so data can pass via shared memory instead of network.
4. **Execute pipelined** where possible; operators like sort must consume all input before emitting.
5. **Keep intermediates** in memory or local disk (spilling to disk if needed), not replicated HDFS.
6. **Recover**: if a machine dies, recompute lost partitions from lineage (Spark), or restore operator state from a checkpoint (Flink).
7. **Write final output** to the distributed filesystem; input remains immutable and output is replaced wholesale, as in MapReduce.

| Aspect | MapReduce | Dataflow engine |
|---|---|---|
| Unit of work | One map→reduce job per stage | Entire workflow as one job |
| Sorting | Always between map and reduce | Only where required |
| Intermediate state | Replicated files on HDFS | Memory / local disk / network |
| Mapper tasks | Often redundant re-reads | Merged into preceding operator |
| Start-up | New JVM per task | JVM reuse |
| Fault tolerance | Re-read durable input | Recompute from lineage / checkpoint |
| Iteration | Re-read from disk each pass | Cache working set in memory |

### 4. Worked example

Switching engines in Hive without changing the query:

```sql
SET hive.execution.engine=tez;     -- default is mr (MapReduce); per-session or per-query
SELECT user_id, COUNT(*) FROM events GROUP BY user_id;
```

Creating and transforming RDDs in Spark and observing lineage:

```scala
val text   = sc.textFile("hdfs:///logs/2026-10-13")       // one partition per HDFS block
val counts = text.map(l => (l.split(" ")(6), 1)).reduceByKey(_ + _)
counts.toDebugString          // prints the lineage graph used for recomputation
counts.saveAsTextFile("hdfs:///derived/url_counts")
```

`reduceByKey` performs partial aggregation per partition (like a combiner) before shuffling, so the function must be commutative and associative. If an executor is lost, Spark recomputes only the missing partitions from `text` using the recorded lineage. Streaming query state in Spark Structured Streaming works similarly: state is cached in executor memory and every update is logged to a checkpoint location, co-versioned with the offsets processed, so a failed micro-batch is re-executed with the same input and same state.

### 5. Under the hood

**Connection options between operators.** The three exchange types let engines implement every MapReduce-era join: repartition-and-sort (sort-merge join, grouping), repartition without sort (partitioned hash join — order is irrelevant since building a hash table randomises it), and broadcast (broadcast hash join). Because all dependencies are declared, the scheduler can choose locality placements and skip unnecessary work.

**Lineage-based fault tolerance.** With durable materialisation, a failed task just reruns on the same input. Without it, the framework must know how each piece of data was computed. Spark tracks RDD ancestry; Flink checkpoints operator state so an operator can resume. Recomputation is not always the best choice: when intermediate data is much smaller than the source, or the computation is CPU-heavy, materialising to files is cheaper than recomputing.

**Determinism matters.** If a recomputed partition differs from the lost one and part of the lost data already went downstream, downstream results become contradictory; the usual remedy is to kill and rerun the downstream operators too (a cascading failure). Nondeterminism sneaks in via hash-table iteration order, random numbers without fixed seeds, the system clock, and external data sources. Remove it — e.g., seed pseudorandom generators — so recovery is cheap.

**Pipelining versus Unix pipes.** MapReduce resembles writing each command's output to a temp file; dataflow engines resemble pipes. Flink is built around pipelined execution. A sort cannot emit anything until it has seen all input, so such operators accumulate state, but many other stages can stream. Final outputs and primary inputs still live as materialised datasets on the filesystem.

**After MapReduce.** The model did not vanish: map, shuffle and reduce still exist, but constraints relax to allow in-memory caching. Spark and BigQuery treat data as distributed in-memory sets with *spill to disk* when memory overflows. Cloud economics help: memory is leased only for the duration of a job.

**High-level APIs.** Relational-style building blocks (join, group, filter, aggregate) in Spark DataFrames/SQL, Flink and Hive let the optimiser pick algorithms and make interactive, incremental development possible, echoing the Unix style.

### 6. Performance and design implications

- Cache working sets in memory for iterative algorithms (ML, graph processing) — the biggest gain over MapReduce.
- Look at the plan: unnecessary shuffles are the main cost; prefer broadcast or pre-partitioned joins where possible.
- Make operators deterministic and side-effect free so recomputation is safe.
- Watch memory: spill to disk is a performance cliff; size executors for the working set and shuffle buffers.
- Materialise (checkpoint/persist) deliberately where recomputation would be expensive or the lineage is long.
- Prefer columnar formats (Parquet) and partition pruning to reduce input.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `hive.execution.engine` (`mr`/`tez`/`spark`) | Selects Hive execution engine | Speeding up Hive without rewriting queries |
| `rdd.toDebugString` / `df.explain()` | Shows lineage / physical plan | Understanding shuffles and recomputation |
| `df.cache()` / `persist(level)` | Keeps data in memory/disk across actions | Iterative or reused datasets |
| `checkpoint()` / `checkpointLocation` | Saves lineage cut or streaming state | Long lineages, stateful streaming |
| `spark.default.parallelism`, `spark.sql.shuffle.partitions` | Parallelism of RDD / SQL shuffles | Tuning task counts |
| `spark-submit --executor-memory` | Executor memory sizing | Avoiding spill and OOM |
| Flink checkpoint interval | State snapshot frequency | Recovery time vs overhead |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| Multi-stage workflows, iterative algorithms, interactive SQL | One simple scan-and-aggregate job where MapReduce-era tools already suffice |
| Recompute from lineage when intermediates are large and cheap to recompute | Intermediates are small or computation is CPU-heavy: materialise instead |
| Spark for batch plus micro-batch streaming | You need true record-at-a-time low latency (consider Flink) |
| Tez/Spark under Hive/Pig for easy speed-up | Unsupported UDFs or cluster components |
| In-memory caching | Memory is scarce or shared; data larger than RAM spills |

### 9. Common mistakes

- Using nondeterministic functions (random, `now()`, unordered iteration) in transformations → inconsistent results after failure → seed RNGs, avoid clock use, sort before order-dependent logic.
- Caching everything → memory pressure and spills → cache only reused datasets.
- Assuming "in-memory" means no disk → surprising spill and shuffle files → monitor shuffle spill metrics.
- Ignoring skew → one straggler stalls the stage → see *Skew Handling*.
- Treating checkpointing as free → latency overhead → tune interval.

### 10. 🎯 Interview kit

**30-second answer:** Dataflow engines like Spark, Tez and Flink execute a full workflow as one DAG of operators instead of separate MapReduce jobs. They skip unneeded sorts, avoid materialising every stage to HDFS by keeping intermediates in memory or local disk, and recover by recomputing lost data from lineage (Spark) or restoring checkpointed state (Flink).

**Structure a longer answer:** MapReduce limitations → operator graph and exchange types → pipelining and locality → fault tolerance and determinism → when to materialise.

**Likely follow-ups:**
- *Why is Spark faster than MapReduce?* → No per-stage HDFS writes, JVM reuse, in-memory caching, optimisation across the whole DAG.
- *How does Spark recover from a lost executor?* → Recompute lost partitions from lineage.
- *Why does determinism matter?* → Recomputed data must match what downstream operators may already have consumed.
- *Is MapReduce fault tolerance stronger?* → Simpler (durable intermediates), at higher I/O cost.

**Signals interviewers look for:** you articulate materialisation vs recomputation trade-offs and relate them to determinism and performance.

### 11. Check yourself

- [ ] List three advantages dataflow engines have over MapReduce.
- [ ] Describe the three ways operators can be connected and the join each enables.
- [ ] Explain how Spark vs Flink achieve fault tolerance without HDFS-materialised intermediates.
- [ ] Give three sources of nondeterminism and fixes.
- [ ] Explain when materialising intermediate data beats recomputation.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Beyond MapReduce › Discussion of materialization
- *Designing Data-Intensive Applications* — Martin Kleppmann — Beyond MapReduce › Fault tolerance
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — E › F
- *Hadoop: The Definitive Guide* — Tom White — Hive › Execution engines
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Data Transformations › Distributed and fault-tolerant state management
- *Designing Data-Intensive Applications* — Martin Kleppmann — Beyond MapReduce › Dataflow engines
- *Hadoop: The Definitive Guide* — Tom White — Spark › Resilient Distributed Datasets
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — D
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Running code samples › Chapter 34
- *Designing Data-Intensive Applications* — Martin Kleppmann — E

