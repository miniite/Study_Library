---
id: "TOOLS/Apache_Spark/02_Spark_Tuning"
level: 3
type: "curriculum"
curriculum: "data_engineering"
---

# Spark Tuning

**L3 Practitioner** · [↑ Apache Spark](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Spark tuning is a short checklist — reduce what you read, avoid or shrink shuffles, size partitions and executors sensibly, fix skew and spills, and use memory-efficient structured APIs — all guided by the Spark UI.

### 1. The big picture
Most slow Spark jobs are slow for the same handful of reasons: too much data read, too many or too large shuffles, skewed keys that make one task far slower than the rest, memory pressure causing spills and garbage collection, or an executor layout that wastes the cluster. Tuning is therefore not magic; it is **measure → find the dominant stage → apply the matching fix → re-measure**. The Spark UI (stages, tasks, SQL tab, storage tab) is the measurement tool; `explain()` shows the plan.

Detailed deep dives live in the Spark Core and Spark Optimization nodes (shuffle/broadcast joins, skew and spill). This lesson gives the map.

### 2. Core concepts and vocabulary
- **Shuffle** — all-to-all data exchange between stages; written to local disk, fetched over the network.
- **Spill** — when an operation (sort, aggregation, join) doesn't fit in execution memory and writes to disk.
- **Skew** — uneven key distribution so a few tasks process most of the data.
- **Broadcast join** — ship a small table to every executor, avoiding a shuffle of the big one.
- **AQE (Adaptive Query Execution)** — runtime re-optimisation: coalesce shuffle partitions, switch join strategy, split skewed partitions.
- **Unified memory** — executors share a region between *execution* (shuffles, joins, sorts) and *storage* (cached data); either can borrow from the other.
- **Tungsten / encoders** — off-heap binary row format and the mechanism converting JVM objects to it.
- **Predicate pushdown / column pruning** — reading only needed rows/columns from the source.
- **Partition count** — number of tasks in a stage.

### 3. How it works, step by step
A practical tuning loop:
1. **Reproduce and baseline** — note duration and the SQL/Stages tabs.
2. **Find the dominant stage** — longest duration, biggest shuffle read/write, most spill.
3. **Classify the symptom**:

| Symptom in UI | Likely cause | Typical fix |
|---|---|---|
| Few huge tasks, others tiny (max ≫ median) | Data skew | Salting, AQE skew join, filter bad keys |
| "Spill (memory/disk)" columns > 0 | Not enough execution memory or too-large partitions | More partitions, more memory per core |
| Thousands of tiny tasks | Too many partitions / small files | Coalesce, compact files, lower shuffle partitions |
| Long GC time | Heap pressure from objects | Use DataFrames, fewer cached objects, off-heap |
| Big shuffle in a join | Large-large or unneeded shuffle | Broadcast the small side, bucketing, pre-filter |
| Scan reads whole dataset | No pruning | Partition by filter columns, Parquet, push filters |

4. Apply **one** change, re-run, compare.

### 4. Worked example
```python
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.shuffle.partitions", "400")

from pyspark.sql.functions import broadcast

# 1. reduce input early
events = (spark.read.parquet("s3://lake/events/")
          .where("dt >= '2024-05-01'")           # partition pruning
          .select("user_id", "country", "amount")) # column pruning

# 2. avoid shuffling the big side
dim = spark.read.parquet("s3://lake/dim_country/")
joined = events.join(broadcast(dim), "country")

# 3. reuse an expensive intermediate
joined.cache()
joined.count()                                    # materialise

agg = joined.groupBy("country").sum("amount")
agg.explain("formatted")
```
How to check: in `explain` you should see `BroadcastHashJoin` rather than `SortMergeJoin`, `PartitionFilters` in the scan, and fewer `Exchange` nodes. In the UI the Storage tab shows the cached size; in the Stage tab compare *min/median/max* task durations to detect skew.

Executor sizing example: a 16-core, 64 GB node → ~3 executors of 5 cores and ~18–19 GB heap each (leave 1 core/1 GB for OS and YARN overhead). Very large executors (30+ cores) suffer GC and I/O contention; very small ones duplicate broadcast data.

### 5. Under the hood
- **Memory model**: executor heap is divided into reserved memory, user memory, and *unified* memory (`spark.memory.fraction`, default 0.6) split softly between execution and storage (`spark.memory.storageFraction` 0.5). Execution can evict cached blocks down to the storage floor; storage cannot evict running execution.
- **Memory history**: Spark 1.0 used Java objects on-heap (slow, GC-bound). Project Tungsten added a row-based binary format in off-heap memory via encoders. Second-generation Tungsten brought whole-stage code generation and a vectorised column layout. Structured APIs benefit automatically; RDDs of JVM objects do not.
- **Shuffle** writes sorted map outputs per task to local disk, then reducers fetch blocks; `spark.shuffle.service` or external shuffle storage keeps them if executors die.
- **AQE** uses runtime statistics at stage boundaries: merges small post-shuffle partitions toward `advisoryPartitionSizeInBytes` (64 MB default), converts sort-merge to broadcast if one side turns out small, splits skewed partitions.
- **Serialization**: Kryo is faster/more compact than Java serialization for RDD work; DataFrames use Tungsten encoders.

### 6. Performance and design implications
- Biggest wins in order: *read less* → *shuffle less* → *balance work* → *fit in memory* → *tune configs*.
- Target partition size ~100–200 MB after shuffle; tasks 100 ms–a few minutes.
- Prefer built-in functions; avoid row-wise Python UDFs (use pandas/Arrow UDFs).
- Don't cache everything: cache what is reused and fits; unpersist after use.
- Write output as a modest number of well-sized Parquet files (`coalesce`/`repartition` before write, or AQE).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `spark.sql.adaptive.enabled` (+ `.skewJoin.enabled`, `.coalescePartitions.enabled`) | Adaptive execution | Default on in Spark 3.2+ |
| `spark.sql.shuffle.partitions` | Post-shuffle partitions | Match data volume |
| `spark.sql.autoBroadcastJoinThreshold` | Max size to auto-broadcast (10 MB) | Join strategy |
| `spark.executor.memory`, `.cores`, `.memoryOverhead` | Executor sizing | Resource tuning |
| `spark.memory.fraction`, `storageFraction` | Unified memory split | Spill/cached-data balance |
| `df.repartition(n, col)` / `coalesce(n)` | Control partitioning | Skew, output files |
| `persist(StorageLevel.MEMORY_AND_DISK)` | Cache with fallback | Reused data |
| `spark.serializer=KryoSerializer` | Faster serialization | RDD-heavy jobs |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Broadcast join for a small dimension | The "small" table is hundreds of MB → executor OOM |
| More shuffle partitions for big data | Tiny data → scheduling overhead |
| Cache reused intermediate DataFrames | Single-use data → wasted memory |
| Salting hot keys | AQE skew handling already suffices |
| Fewer, bigger executors | Large heaps with long GC pauses |

### 9. Common mistakes
- Tuning configs blindly before reading the UI → no effect → locate the slow stage first.
- Caching without an action or with too little memory → no benefit or eviction thrash → check Storage tab.
- Leaving `shuffle.partitions=200` on terabytes → huge spills → increase or rely on AQE.
- Repartitioning by a skewed key → worsens skew → add salt or use range partitioning.
- Collecting large results → driver OOM → write to storage.

### 10. 🎯 Interview kit
**30-second answer:** I measure in the Spark UI, find the stage dominating runtime, and match the symptom to a fix: prune and push down to read less, broadcast or bucket to avoid shuffles, repartition/AQE for partition sizing, salting for skew, memory/executor sizing for spills and GC, and structured APIs so Tungsten and Catalyst can optimise.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.

**Likely follow-ups:**
- *Job stuck at 199/200 tasks?* → skew; look at max vs median task time.
- *How does unified memory work?* → execution and storage share and borrow.
- *Broadcast threshold risk?* → driver/executor OOM.
- *What does AQE change?* → partitions, join strategy and skew at runtime.

**Signals interviewers look for:** a measure-first method and fluency with the typical symptoms.

### 11. Check yourself
- [ ] Given stage metrics (min/median/max, spill), diagnose the problem.
- [ ] Explain how unified memory splits execution and storage.
- [ ] Choose executor cores/memory for a given node type.
- [ ] Name three things AQE does at runtime.
<!-- explainer:end -->

## In this section (study in order)

1. [Resources Memory and Parallelism](<Resources_and_Parallelism/_index.md>) (L4 Advanced)
2. [Caching and Persistence](<Caching_and_Persistence/_index.md>) (L4 Advanced)
3. [Spark Join Strategies](<Spark_Joins/_index.md>) (L4 Advanced)
4. [Reading the Spark UI](<Spark_UI_Diagnosis/_index.md>) (L4 Advanced)

## References

- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › Configuring Spark executors' memory and the shuffle service
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Optimizing and Tuning Spark Applications › Chapter 7. Optimizing and Tuning Spark Applications
- *Hadoop: The Definitive Guide* — Tom White — How MapReduce Works › Configuration Tuning
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Generalization with flatMapGroupsWithState() › Performance Tuning
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — E › G
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Machine Learning with MLlib › Optimizing Pipelines
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and Datasets › Memory Management for Datasets and DataFrames
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Managing, Deploying, and Scaling Machine Learning Pipelines with Apache Spark › In Python
- *Hadoop: The Definitive Guide* — Tom White — Spark › A Scala Standalone Application
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › Performance optimizations

