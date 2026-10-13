---
id: "TOOLS/Apache_Spark/PySpark/Structured_Streaming_in_PySpark"
level: 4
type: "concept"
topic: "TOOLS/Apache_Spark/PySpark"
tools: ["Apache_Spark", "AWS"]
---

# Structured Streaming in PySpark

**L4 Advanced** · [↑ PySpark](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Structured Streaming lets you write a normal DataFrame query over an unbounded table; Spark runs it incrementally in micro-batches, using a checkpoint for exactly-once progress, and you tune it with triggers, watermarks and offset limits.

### 1. The big picture
Batch jobs read a finite dataset and finish. Structured Streaming treats a stream (Kafka topic, files landing in a folder, a Delta table) as a table that keeps growing: each trigger Spark processes only the *new rows* and updates the result. Because the API is the DataFrame/SQL API, the same transformation code runs on batch and streaming (`read` ↔ `readStream`, `write` ↔ `writeStream`), and Catalyst still optimises it.

The rest of the PySpark topics (lazy evaluation, UDF cost, retries, testing) apply unchanged; streaming adds three new concerns: **state** (aggregations, joins, dedup), **time** (event time, watermarks, lateness) and **progress** (offsets and checkpoints).

### 2. Core concepts and vocabulary
- **Source / sink** — where the stream is read (`kafka`, `cloudFiles`, `delta`, `rate`) and written (`delta`, `kafka`, `foreachBatch`, `console`).
- **Micro-batch** — a bounded chunk of new data processed as one small batch job; default engine.
- **Trigger** — when batches run: `processingTime='15 seconds'`, `availableNow=True` (process all available then stop), `once=True` (deprecated), continuous (experimental).
- **Output mode** — `append` (only new final rows), `update` (changed rows), `complete` (whole result table).
- **Checkpoint** — directory storing offsets, commits and operator state; enables restart with exactly-once semantics to idempotent sinks.
- **Event time vs. processing time** — when the event happened vs. when Spark saw it.
- **Watermark** — "I won't wait longer than X for late data"; lets Spark finalise windows and drop old state.
- **State store** — per-operator keyed state (running counts, dedup keys, join buffers).
- **`maxOffsetsPerTrigger` / `maxFilesPerTrigger`** — cap on the size of each micro-batch.
- **`foreachBatch`** — run arbitrary batch code (MERGE, JDBC, multi-sink) on each micro-batch DataFrame.

### 3. How it works, step by step
1. Driver plans the streaming query and starts a loop.
2. For each trigger: ask the source for the latest available offsets; compute the range since the last committed batch.
3. **Write the offset range to the checkpoint's `offsets/` log first** (write-ahead).
4. Run the incremental plan over that range as a normal Spark job (shuffles, state updates included).
5. Write results to the sink; on success write `commits/<batchId>`.
6. Update watermark from the max event time seen; evict state older than the watermark.
7. On failure/restart: reread the last offset entry; if it has no commit, re-run **the same range** — the sink must be idempotent (Delta is, keyed by batchId) for exactly-once results.

### 4. Worked example
```python
from pyspark.sql import functions as F

events = (spark.readStream.format("kafka")
    .option("kafka.bootstrap.servers", "broker:9092")
    .option("subscribe", "page_views")
    .option("startingOffsets", "earliest")
    .option("maxOffsetsPerTrigger", 500_000)       # backpressure
    .load()
    .select(F.from_json(F.col("value").cast("string"), schema).alias("v"))
    .select("v.*"))

agg = (events
    .withWatermark("event_time", "10 minutes")      # tolerate 10 min lateness
    .groupBy(F.window("event_time", "5 minutes"), "page")
    .count())

q = (agg.writeStream
     .format("delta").outputMode("append")           # emit a window once the watermark passes it
     .option("checkpointLocation", "s3://lake/_chk/page_counts")
     .trigger(processingTime="30 seconds")
     .toTable("gold.page_counts"))
```
Reading it: with `append` mode and a watermark, a 5-minute window is emitted only after the watermark moves past its end (event_time max − 10 min), so results are delayed by about lateness + window. `q.lastProgress` and `q.status` show `inputRowsPerSecond`, `processedRowsPerSecond`, `numInputRows` and state size; if input rate > processed rate the stream is falling behind.

### 5. Under the hood
- **Exactly-once** = replayable source (offsets) + checkpointed progress + idempotent/transactional sink. End-to-end across Kafka → Spark → Kafka is at-least-once unless the sink dedups.
- **State store**: RocksDB or HDFS-backed in-memory maps per partition, snapshotted to the checkpoint; unbounded state (no watermark) grows forever → OOM. Streaming `dropDuplicates` needs a watermark on the event-time column to bound state.
- **Checkpoint compatibility**: the checkpoint is tied to the query plan's stateful operators, sources and schemas. Changing aggregation keys, adding a stateful operator or changing the number of shuffle partitions of a stateful stage (state is partitioned by `spark.sql.shuffle.partitions` at first start) breaks restart; fixes are a new checkpoint + replay from a known offset, or keeping the same shuffle partition count.
- **Stream–stream joins** buffer both sides; need watermarks on both and a time-range condition to expire state.
- **Small files**: frequent triggers write many small files; use larger triggers, `optimize`/auto-compaction, or `availableNow` micro-batch jobs.

### 6. Performance and design implications
- Monitor: batch duration vs. trigger interval, input vs. processed rate, state rows/bytes, watermark delay, source lag (Kafka consumer lag).
- Set `maxOffsetsPerTrigger` to avoid giant first batches after a long outage; too small starves throughput.
- Keep transformations in built-in functions; Python UDFs inflate batch time.
- Prefer `availableNow` for cost-efficient incremental batch (scheduled by Airflow) when seconds-level latency isn't needed.
- Retry-safety in orchestrators: the checkpoint *is* the state, so reruns resume; clean up only non-checkpoint temp output, and keep sinks idempotent.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `readStream` / `writeStream` | Streaming reader/writer | Always |
| `.trigger(processingTime=..., availableNow=True)` | Batch cadence | Latency vs. cost |
| `.option("checkpointLocation", path)` | Progress + state | Mandatory in production |
| `.withWatermark(col, "10 minutes")` | Bound lateness and state | Windows, dedup, joins |
| `maxOffsetsPerTrigger`, `maxFilesPerTrigger` | Batch size cap | Backlog / backpressure |
| `foreachBatch(fn)` | Custom per-batch logic (MERGE) | Upserts, multi-sink |
| `query.lastProgress`, `query.recentProgress` | Metrics | Monitoring |
| `spark.sql.streaming.stateStore.providerClass` | RocksDB state store | Large state |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Micro-batch, 10s–minutes | You need sub-second latency (use Flink/continuous) |
| `availableNow` scheduled runs | You need an always-on low-latency view |
| `foreachBatch` MERGE | Simple append (overhead; use native sink) |
| Longer watermark | State size and output delay grow |
| `update` mode | Sink can't handle updates (append-only files) |

### 9. Common mistakes
- No watermark on aggregations/dedup → state grows forever → add watermark.
- Changing stateful logic and reusing the checkpoint → restart failure → new checkpoint and replay plan.
- Using `collect()`/`toPandas()` in streams → driver OOM → `foreachBatch` with distributed writes.
- Huge `maxOffsetsPerTrigger` after backlog → one giant batch OOM/spill → cap it.
- Overusing `foreachBatch` for plain writes; non-idempotent writes in it → duplicates on retry → key writes by `batchId` or MERGE.

### 10. 🎯 Interview kit
**30-second answer:** "Structured Streaming runs a DataFrame query incrementally over an unbounded table in micro-batches. Progress is tracked by offsets in a checkpoint, so with a replayable source and idempotent sink I get exactly-once; watermarks bound late data and state."

**Structure a longer answer:** model (unbounded table) → micro-batch loop → state & event time → exactly-once/checkpoint → ops (monitoring, backpressure) → when not to use it.

**Likely follow-ups:**
- *Job won't restart after a code change?* → checkpoint incompatible with changed stateful plan/schema/partition count; revert, or start a new checkpoint from a safe offset and reconcile.
- *Trigger interval vs. latency/throughput?* → shorter = lower latency, more small files and overhead.
- *Late data after watermark?* → dropped for aggregations; side-output via separate path if needed.
- *Retry-safe in Airflow?* → rely on checkpoint, idempotent sink, no external state cleanup beyond temp.

**Signals interviewers look for:** correct mental model of checkpoint/offset commit, state bounding, honest limits on latency.

### 11. Check yourself
- [ ] Draw the offset → batch → commit sequence and explain recovery after a crash mid-batch.
- [ ] Explain what a watermark does to state and output delay.
- [ ] Choose a trigger for hourly freshness at minimum cost and justify.
- [ ] List what makes a checkpoint incompatible after a code change.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Spark](<../_index.md>)
- [AWS](<../../AWS/_index.md>)

## Practice questions

- [Basic] [Explain lazy evaluation in Spark — what triggers actual computation and why is understanding the transformation/action split critical for debugging?](<../../../PRACTICE/TOOLS/Apache_Spark/PySpark/Q-36406_Explain_lazy_evaluation_in_Spark_what_triggers_actual_comput.md>)
- [Intermediate] [Why can a .collect() on a moderately sized DataFrame crash the driver and what pattern do you use when you truly need to pull results to the driver?](<../../../PRACTICE/TOOLS/Apache_Spark/PySpark/Q-36417_Why_can_a_collect_on_a_moderately_sized_DataFrame_crash_the_.md>)
- [Intermediate] [How would you migrate a legacy Hive-on-MapReduce ETL to PySpark and what compatibility issues (SerDe partition metadata) do you expect?](<../../../PRACTICE/TOOLS/Apache_Spark/PySpark/Q-36457_How_would_you_migrate_a_legacy_Hive-on-MapReduce_ETL_to_PySp.md>)
- [Intermediate] [Explain how you'd process nested/complex types (arrays structs maps) in PySpark — explode selectExpr and the cost of exploding a large array.](<../../../PRACTICE/TOOLS/Apache_Spark/PySpark/Q-36471_Explain_how_you_d_process_nested_complex_types_arrays_struct.md>)
- [Deep] [Why is a Python UDF often 10x slower than the equivalent built-in Spark SQL function and what is happening at the serialization boundary?](<../../../PRACTICE/TOOLS/Apache_Spark/PySpark/Q-36400_Why_is_a_Python_UDF_often_10x_slower_than_the_equivalent_bui.md>)
- [Deep] [Your streaming job keeps failing to restart from a checkpoint after a code change — what changed that broke checkpoint compatibility and how do you recover?](<../../../PRACTICE/TOOLS/Apache_Spark/PySpark/Q-36431_Your_streaming_job_keeps_failing_to_restart_from_a_checkpoin.md>)
- [Deep] [You see 'Container killed by YARN for exceeding memory limits' — how do you reason about executor memory overhead vs heap and what knobs do you turn?](<../../../PRACTICE/TOOLS/Apache_Spark/PySpark/Q-36455_You_see_Container_killed_by_YARN_for_exceeding_memory_limits.md>)
- [Deep] [How do you make a PySpark pipeline retry-safe when Airflow reruns a failed task — what state must be cleaned up on retry?](<../../../PRACTICE/TOOLS/Apache_Spark/PySpark/Q-36470_How_do_you_make_a_PySpark_pipeline_retry-safe_when_Airflow_r.md>)

## Related concepts

- [Pandas vs Polars vs PySpark](<../../Python/Python_Language/Pandas_vs_Polars_vs_PySpark.md>) *(Python Language)*
- [Spark Execution Caching and Debugging](<../Spark_Core/Spark_Execution_Caching_and_Debugging.md>) *(Spark Core)*
- [Spark Executor Sizing](<../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Spark_Executor_Sizing.md>) *(Big Data Processing)*

## References

- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Batch and Real-Time Processing in Databricks › Working with batch processing
- *Optimizing Databricks Workloads* — Anirudh Kala, Anshul Bhatnagar, Sarthak Sarbahi — Big Data Analytics › Understanding built-in Spark functions
- *Data Algorithms with Spark* — Mahmoud Parsian — **PART I** › PART I
- *Data Algorithms with Spark* — Mahmoud Parsian — **Spark Usage** › Chapter 2(#indexsplit008.htmlpage59)). A high-level view of PySpark's architecture is presented in Figure 1-7(#indexsplit006.htmlpage40).
- *Data Algorithms with Spark* — Mahmoud Parsian — **Serverless Spark** › CHAPTER 2
- *Data Algorithms with Spark* — Mahmoud Parsian — **CHAPTER 4** › CHAPTER 4
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Processing Incremental Data › Persisting streaming data
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Processing Incremental Data › Streaming Data Manipulations in Python
- *Data Engineering Design Patterns* — Bartosz Konieczny — Chapter 5. Data Value Design Patterns › Examples
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Flow Design Patterns › Heavy conditions
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Installing Delta Lake › PySpark shell
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Streaming In and Out of Your Delta Lake › Python
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Advanced Features › Python
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Architecting Your Lakehouse › Guarding the Bronze Layer with Permissive Mode in Spark
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Introduction to Apache Spark A Unified Analytics Engine › In Python
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Downloading Apache Spark and Getting Started › Step 2: Using the Scala or PySpark Shell
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Downloading Apache Spark and Getting Started › Using the Local Machine
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Downloading Apache Spark and Getting Started › Import the necessary libraries.

