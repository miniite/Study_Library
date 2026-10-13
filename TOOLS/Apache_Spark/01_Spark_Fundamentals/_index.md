---
id: "TOOLS/Apache_Spark/01_Spark_Fundamentals"
level: 3
type: "curriculum"
curriculum: "data_engineering"
---

# Spark Fundamentals

**L3 Practitioner** · [↑ Apache Spark](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Spark is a unified, in-memory distributed engine: you write transformations in Python, Scala, Java, SQL or R, Spark turns them into a DAG of tasks, and a driver plus executors run those tasks in parallel across a cluster.

### 1. The big picture
Hadoop MapReduce proved that big data could be processed on commodity clusters, but every step wrote to disk, only offered map and reduce, and needed separate systems for SQL (Hive), streaming (Storm), machine learning and graphs. Spark was designed to fix this: keep intermediate data in memory, offer a rich set of operators, and put batch, SQL, streaming, ML and graph processing under one engine and one API. That unification is why the ACM recognised the original Spark paper ("a unified engine for big data processing").

Spark's own design philosophy has four traits: **speed**, **ease of use**, **modularity** and **extensibility** (it reads from and writes to many sources: HDFS, S3, Kafka, JDBC, Cassandra, Delta/Parquet…). This lesson covers the model; the tuning and streaming nodes build on it.

### 2. Core concepts and vocabulary
- **Application** — a user program built on Spark, consisting of a driver and executors.
- **Driver** — the process that runs `main()`, creates the `SparkSession`, builds the plan and schedules tasks.
- **Executor** — a JVM process on a worker node that runs tasks and caches data.
- **Cluster manager** — allocates resources: standalone, YARN, Kubernetes (Mesos in older versions).
- **SparkSession** — the unified entry point (replaces separate SparkContext/SQLContext/HiveContext).
- **RDD** — Resilient Distributed Dataset, the low-level immutable partitioned collection; fault-tolerant through *lineage*.
- **DataFrame / Dataset** — higher-level structured APIs with schema, optimised by the Catalyst optimiser.
- **Transformation** — lazy operation that defines a new dataset (`select`, `filter`, `groupBy`, `join`).
- **Action** — triggers execution (`count`, `collect`, `show`, `write`).
- **Job / stage / task** — an action creates a job; a job is split into stages at shuffle boundaries; a stage runs one task per partition.
- **Partition** — unit of parallelism; one task processes one partition.
- **Shuffle** — redistribution of data across the network between stages.

### 3. How it works, step by step
1. Your code calls transformations; Spark only records a **logical plan** (lazy evaluation).
2. An action arrives. The Catalyst optimiser rewrites the plan (predicate pushdown, column pruning, join reordering) and chooses a **physical plan**.
3. The **DAG scheduler** cuts the plan into **stages** at wide dependencies (shuffles).
4. The **task scheduler** sends tasks (one per partition) to executors, preferring those near the data.
5. Executors run tasks, possibly shuffling data to the next stage, and return results to the driver or write to storage.
6. If a task or executor fails, Spark re-runs the lost partitions from lineage.

| Dependency type | Example | Cost |
|---|---|---|
| Narrow | `map`, `filter`, `select` | No shuffle; pipelined in one stage |
| Wide | `groupBy`, `join`, `repartition`, `distinct` | Shuffle; new stage |

**Deployment modes**: *client* (driver on the submitting machine — good for interactive), *cluster* (driver runs inside the cluster — good for production jobs), and *local* (everything in one JVM for testing).

### 4. Worked example
```python
from pyspark.sql import SparkSession, functions as F

spark = SparkSession.builder.appName("max-temp").getOrCreate()

df = (spark.read.option("sep", "\t")
      .schema("year INT, temp INT, quality INT")
      .csv("hdfs:///data/ncdc/"))

result = (df.filter((F.col("temp") != 9999) & F.col("quality").isin(0,1,4,5,9))
            .groupBy("year")
            .agg(F.max("temp").alias("max_temp")))

result.explain()          # inspect the physical plan
result.show()             # action: runs the job
result.write.mode("overwrite").parquet("hdfs:///out/max_temp")
```
```bash
spark-submit --master yarn --deploy-mode cluster \
  --num-executors 10 --executor-cores 4 --executor-memory 8g app.py
```
Reading it: `filter` and `groupBy` are lazy; `show()` is the first action. `explain()` reveals a scan with pushed-down filters, a partial aggregate, an `Exchange` (shuffle) and a final aggregate — two stages. In the Spark UI (port 4040) the job appears with stages and task counts equal to the partition counts.

### 5. Under the hood
- **RDD lineage**: each RDD remembers the transformations that produced it; lost partitions are recomputed, so no data replication is needed for fault tolerance.
- **Catalyst** optimiser works on trees of expressions in phases: analysis → logical optimisation → physical planning → code generation.
- **Tungsten**: off-heap binary row format, cache-aware algorithms and **whole-stage code generation** that fuses operators into one tight loop; second-generation vectorised column layout for Parquet/ORC scans.
- **Why in-memory is faster than MapReduce**: stages pass data via memory/local shuffle files instead of writing every step to HDFS, and the DAG avoids unnecessary materialisation. It also lets iterative algorithms (ML) reuse cached data.
- Spark is not a storage system and not OLTP; it relies on external storage.

### 6. Performance and design implications
- Use DataFrames/SQL rather than raw RDDs so Catalyst and Tungsten can help.
- Count shuffles: each wide transformation is a stage boundary and a network/disk cost.
- Partition counts should match cluster parallelism (2–4 tasks per core) and avoid tiny or giant partitions.
- Use columnar formats (Parquet/ORC) and let filters push down.
- Avoid `collect()` on large data — it pulls everything to the driver.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `spark-submit` | Launch an application | Production jobs |
| `pyspark` / `spark-shell` / `spark-sql` | Interactive shells | Exploration |
| `df.explain(True)` | Show logical and physical plans | Understanding/tuning |
| `spark.sql.shuffle.partitions` | Partitions after a shuffle (default 200) | Tuning |
| `--executor-memory/-cores`, `--num-executors` | Executor sizing | Resource allocation |
| `df.cache()` / `persist()` | Keep data in memory/disk | Reused datasets |
| Spark UI (4040) / History Server | Jobs, stages, SQL, storage tabs | Debugging |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Large-scale ETL, analytics, ML, streaming in one stack | Tiny data (a single machine is faster) |
| Iterative or multi-step pipelines | Low-latency per-record serving |
| Need SQL + code in the same job | Work that is strictly single-pass and simple (plain SQL warehouse may do) |
| DataFrame API for performance and safety | Python UDFs on huge data without vectorisation |

### 9. Common mistakes
- Thinking transformations run immediately → confusion about timing → remember laziness; only actions run.
- `collect()` on big data → driver OOM → write out or aggregate first.
- Default 200 shuffle partitions for both tiny and huge data → overhead or spills → tune or enable AQE.
- Python UDFs row-by-row → slow → built-in functions or pandas UDFs.
- Forgetting that each action recomputes unless cached → repeated work → `cache()` reused data.

### 10. 🎯 Interview kit
**30-second answer:** Spark is a unified distributed engine where a driver builds a DAG from lazy transformations and executors run tasks per partition; stages split at shuffles, the Catalyst optimiser plans queries, Tungsten executes them efficiently, and lineage gives fault tolerance. It is faster than MapReduce because it keeps intermediate data in memory and optimises the whole DAG.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.

**Likely follow-ups:**
- *Transformation vs action?* → lazy vs triggers execution.
- *Narrow vs wide dependency?* → shuffle boundary / stage.
- *What happens when an executor dies?* → tasks rescheduled, partitions recomputed from lineage.
- *RDD vs DataFrame?* → low-level vs optimised structured API.

**Signals interviewers look for:** correct mental model of driver/executor, stages and shuffles.

### 11. Check yourself
- [ ] Draw the driver–cluster manager–executor architecture.
- [ ] Given a pipeline, count the stages and name the shuffles.
- [ ] Explain lazy evaluation and why it helps the optimiser.
- [ ] Choose client vs cluster mode for an interactive notebook and a nightly job.
<!-- explainer:end -->

## In this section (study in order)

1. [Spark Architecture and Execution](<Spark_Architecture/_index.md>) (L4 Advanced)
2. [DataFrame and Dataset APIs](<Structured_APIs/_index.md>) (L4 Advanced)
3. [Spark SQL and Data Sources](<Spark_SQL_and_Catalyst/_index.md>) (L4 Advanced)

## References

- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Introduction to Apache Spark A Unified Analytics Engine › Unified Analytics
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Apache Spark's Structured APIs › Chapter 3. Apache Spark's Structured APIs
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Introduction to Apache Spark A Unified Analytics Engine › What Is Apache Spark?
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Introduction to Apache Spark A Unified Analytics Engine › Chapter 1. Introduction to Apache Spark: A Unified Analytics Engine
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Apache Spark's Structured APIs › Spark SQL and the Underlying Engine
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Introduction to Apache Spark A Unified Analytics Engine › The Developer's Experience
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Machine Learning with MLlib › Why Spark for Machine Learning?
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Introduction to Apache Spark A Unified Analytics Engine › Apache Spark's Distributed Execution
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Apache Spark's Structured APIs › Structuring Spark
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Chapter 8. Structured Streaming

