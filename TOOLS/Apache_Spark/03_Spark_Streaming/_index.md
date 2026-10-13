---
id: "TOOLS/Apache_Spark/03_Spark_Streaming"
level: 3
type: "curriculum"
curriculum: "data_engineering"
---

# Structured Streaming

**L3 Practitioner** · [↑ Apache Spark](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Structured Streaming lets you write a streaming job as if it were a batch query over an ever-growing table; Spark runs it incrementally in micro-batches (or continuously) and uses checkpoints and idempotent sinks for fault tolerance and exactly-once results.

### 1. The big picture
Classic streaming systems process one record at a time and expose low-level operator APIs, leaving developers to handle state, late data, failures and the interaction with batch jobs. Spark first answered with **DStreams**: chop the stream into small batches and run ordinary Spark jobs on each. Micro-batching gave fast recovery (reschedule small deterministic tasks) and exactly-once output, but the API was low-level and separate from batch code.

**Structured Streaming** was designed from scratch around one philosophy: *writing a streaming pipeline should be as easy as writing a batch pipeline.* The same DataFrame/SQL code works for both; the engine handles incrementalisation, state, fault tolerance and late data. It also broadens "streaming" to any application that processes data periodically (every few hours) up to continuously.

### 2. Core concepts and vocabulary
- **Unbounded table** — the conceptual model: each new record is a new row appended to an infinite input table.
- **Result table / output mode** — the table produced by the query; *append* (only new rows), *update* (changed rows), *complete* (whole table).
- **Source** — where data comes from: Kafka, files in a directory, sockets (testing), rate source.
- **Sink** — where results go: files (Parquet/Delta), Kafka, `foreachBatch`, console/memory (testing).
- **Trigger** — when to run: default ASAP, fixed interval (`processingTime`), `availableNow` (process everything then stop), or experimental continuous.
- **Micro-batch** — one incremental execution over the new data since the last offsets.
- **Checkpoint** — durable storage of offsets and state so a restarted query resumes exactly.
- **Event time / watermark** — event time is when the event happened; a watermark says how late data may arrive before state is dropped.
- **Stateful operation** — aggregation, deduplication, stream-stream join that keeps state between batches.

### 3. How it works, step by step
1. You define a streaming DataFrame with `readStream` and apply ordinary transformations.
2. You start the query with `writeStream`, giving a sink, output mode, trigger and **checkpoint location**.
3. For each trigger, the engine asks sources for the new offsets, logs them to the checkpoint's write-ahead log, and plans an incremental batch query (Catalyst treats it like a batch job).
4. Tasks process the new data, updating state stores for stateful operators.
5. Results are written to the sink; the batch is marked committed in the checkpoint.
6. On failure/restart, the engine reads the checkpoint, replays the last uncommitted batch with the **same offsets** and, because sources are replayable and sinks idempotent, produces the same output once.

```
Kafka ──► [micro-batch N: offsets 100–250] ──► state store ──► sink (idempotent)
                     ▲ checkpoint: offsets WAL + state snapshots
```

**Micro-batch vs true streaming.** Micro-batch (Spark) typically gives latencies of hundreds of milliseconds to seconds with high throughput; true record-at-a-time engines (Flink, Beam runners) reach lower latency at more overhead. Many "streaming" metrics are window-based anyway, so choose by required latency: sales dashboards every few minutes are fine with micro-batches; per-second attack detection may need a true streaming engine.

### 4. Worked example
```python
from pyspark.sql import functions as F

raw = (spark.readStream.format("kafka")
       .option("kafka.bootstrap.servers", "broker:9092")
       .option("subscribe", "clicks").load())

events = (raw.select(F.from_json(F.col("value").cast("string"),
                                  "user STRING, url STRING, ts TIMESTAMP").alias("e"))
             .select("e.*"))

counts = (events
          .withWatermark("ts", "10 minutes")
          .groupBy(F.window("ts", "5 minutes"), "url")
          .count())

query = (counts.writeStream
         .outputMode("update")
         .format("delta")                       # or parquet / kafka
         .option("checkpointLocation", "s3://lake/chk/clicks_5m")
         .trigger(processingTime="30 seconds")
         .start("s3://lake/clicks_5m"))
```
Reading it: the 5-minute tumbling window counts clicks per URL by **event time**. The 10-minute watermark lets late events within 10 minutes still update their window; older state is purged so memory stays bounded. `update` mode writes only windows that changed in each batch. `query.status` and `query.lastProgress` give input rate, processing rate and batch duration; the UI's Structured Streaming tab charts them.

### 5. Under the hood
- **Fault-tolerance contract** — three ingredients give end-to-end exactly-once: (1) replayable sources with offsets (Kafka, files), (2) deterministic processing with state checkpointed, (3) idempotent or transactional sinks (file sink with commit log, Delta, `foreachBatch` with batch-id dedupe).
- **State store** — per-partition key-value store (in-memory with changelog and periodic snapshots to the checkpoint directory; RocksDB provider for very large state).
- **Watermark semantics** — `watermark = max event time seen − threshold`; state for windows older than the watermark is dropped and later events for them are ignored. It is the only way to bound state in event-time aggregations and to emit finalised results in append mode.
- **Output-mode constraints** — append requires the engine to know a row will never change (watermarked aggregations, or no aggregation); complete mode needs all state and is rarely suitable for big keyspaces.
- **Continuous processing** trigger offers ~ms latency with at-least-once for a limited set of operations.
- Compare Structured Streaming with Kafka Streams and Flink in the stream-processing curriculum nodes.

### 6. Performance and design implications
- Throughput: size trigger interval so the batch finishes well before the next; if `processingRate < inputRate`, lag grows.
- Control batch size with `maxOffsetsPerTrigger` / `maxFilesPerTrigger`.
- Keep shuffle partitions modest (default 200 is often too many for streaming's small batches).
- State size: set watermarks, use `dropDuplicatesWithinWatermark`, consider RocksDB state store.
- Small output files: use longer triggers, Delta optimise/compaction, or `availableNow` for periodic jobs.
- Always set a durable `checkpointLocation`; changing the query's logic incompatibly may require a new checkpoint.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `spark.readStream` / `df.writeStream` | Define source / sink | Every streaming job |
| `.trigger(processingTime=…, availableNow=True)` | Schedule micro-batches | Latency/cost control |
| `.option("checkpointLocation", path)` | Durable progress/state | Always in production |
| `.withWatermark(col, delay)` | Bound state, handle lateness | Event-time aggregations |
| `foreachBatch(fn)` | Custom per-batch write (upserts, multi-sink) | MERGE into tables |
| `query.lastProgress`, `query.awaitTermination()` | Monitoring / block | Operations |
| `maxOffsetsPerTrigger` | Cap Kafka records per batch | Backfills, stability |
| `spark.sql.streaming.stateStore.providerClass` | RocksDB state store | Large state |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Seconds-level latency is fine; you already use Spark for batch | You need sub-100 ms per-event reaction (consider Flink) |
| One codebase for batch backfill and streaming | Complex custom event-time logic better expressed in Flink/Beam |
| Incremental ETL on a schedule (`availableNow`) | Always-on tiny workloads where cost of a cluster dominates |
| Delta/Kafka sinks with idempotent writes | Non-idempotent external sinks without dedupe |

### 9. Common mistakes
- No checkpoint → no recovery or duplicates → set a durable location.
- Aggregating on event time with no watermark → unbounded state → add a watermark.
- Using `complete` mode on high-cardinality aggregations → memory blow-up → use update/append with watermarks.
- Treating processing time as event time → wrong windows on replays/late data → use event time.
- Writing many tiny files every few seconds → file explosion → longer triggers, compaction.
- Changing group keys or schema and reusing the old checkpoint → failure/corrupted state → new checkpoint and backfill.

### 10. 🎯 Interview kit
**30-second answer:** Structured Streaming models a stream as an unbounded table and runs the same DataFrame/SQL query incrementally in micro-batches. Offsets and state are checkpointed; with replayable sources and idempotent sinks it gives end-to-end exactly-once, and watermarks handle late events and bound state.

**Structure a longer answer:** clarify → core mechanism → trade-offs → close with a real scenario.

**Likely follow-ups:**
- *How do you guarantee exactly-once?* → replayable source + checkpointed offsets/state + idempotent sink.
- *What is a watermark?* → threshold on lateness to finalise windows and drop state.
- *Output modes?* → append/update/complete and when each is valid.
- *Micro-batch vs continuous?* → latency vs guarantees and operator coverage.

**Signals interviewers look for:** you explain recovery mechanics, event time vs processing time, and state bounding.

### 11. Check yourself
- [ ] Describe what happens on restart after a mid-batch crash.
- [ ] Explain how a watermark of 10 minutes affects a 5-minute window.
- [ ] Choose an output mode for three queries and justify it.
- [ ] Decide between Structured Streaming and a true-streaming engine for a use case.
<!-- explainer:end -->

## In this section (study in order)

1. [Anatomy of a Streaming Query](<Streaming_Query_Anatomy/_index.md>) (L4 Advanced)
2. [Stateful Streaming Aggregations and Joins](<Stateful_Streaming/_index.md>) (L4 Advanced)
3. [Streaming Fault Tolerance and Monitoring](<Streaming_Reliability/_index.md>) (L4 Advanced)

## References

- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Chapter 8. Structured Streaming › The Advent of Micro-Batch Stream Processing
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Chapter 8. Structured Streaming › The Philosophy of Structured Streaming
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Streaming Data Sources and Sinks
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Managing, Deploying, and Scaling Machine Learning Pipelines with Apache Spark › Streaming
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Chapter 8. Structured Streaming › The Programming Model of Structured Streaming
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Accelerator-Aware Scheduler › Structured Streaming
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Materialized Views, Federation, and Query Virtualization › Micro-batch versus true streaming
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — The Fundamentals of a Structured Streaming Query
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Streaming Data Sources and Sinks › Custom Streaming Sources and Sinks
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — T

