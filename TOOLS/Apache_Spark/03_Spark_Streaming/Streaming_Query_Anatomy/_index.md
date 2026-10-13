---
id: "TOOLS/Apache_Spark/03_Spark_Streaming/Streaming_Query_Anatomy"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Anatomy of a Streaming Query

**L4 Advanced** · [↑ Structured Streaming](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A Structured Streaming query is a DataFrame query over an unbounded table: you define a source, transformations, a sink, an output mode, a trigger and a checkpoint location, and Spark runs it as a loop of incremental micro-batches.

### 1. The big picture
Structured Streaming's programming model treats a live stream as a table that keeps growing: each new record is a new row appended to an **input table**. You write the same DataFrame/SQL query you would for a static table; Spark computes the **result table** incrementally and writes changes to a sink. This is why batch and streaming code look nearly identical and why the Catalyst optimiser, joins and aggregations all carry over.

Compared with the older DStream API (RDD-based micro-batches), Structured Streaming is declarative, event-time aware, and supports end-to-end exactly-once semantics when source, computation and sink cooperate. The reliability and state lessons build on the anatomy described here.

### 2. Core concepts and vocabulary
- **Source** — where data comes from: files (directory), Apache Kafka, a socket (testing), or the built-in `rate` generator.
- **Sink** — where results go: files (Parquet/JSON…), Kafka, `foreach`/`foreachBatch`, console/memory (testing), Delta tables.
- **Micro-batch** — one execution of the incremental plan on the new data range.
- **Trigger** — when to run the next batch: default as soon as possible, fixed interval (`processingTime`), once / `availableNow`, or experimental continuous.
- **Output mode** — what part of the result table is written each trigger: *append* (only new final rows), *update* (changed rows), *complete* (whole table).
- **Checkpoint location** — directory holding offsets, commit log and state; identifies the query and enables recovery.
- **`StreamingQuery`** — handle returned by `start()`: `status`, `lastProgress`, `awaitTermination`, `stop`.
- **Offset** — position in a source (file set, Kafka offset ranges) recorded per batch.
- **Event time vs processing time** — timestamp in the data vs the time Spark sees it.

### 3. How it works, step by step
The five steps to define a query:
1. **Define the input source** with `spark.readStream.format(...)`; streaming file and Kafka sources need a schema or provide a fixed one.
2. **Transform** with ordinary DataFrame operations (`select`, `filter`, `groupBy`, `join` with static data, windows).
3. **Define sink and output mode** with `df.writeStream.format(...).outputMode(...)`.
4. **Specify processing details** — trigger, checkpoint location, query name.
5. **Start** with `.start()`; it returns immediately (non-blocking), so call `awaitTermination()` if the main thread should wait.

What the engine does once started:
1. The logical plan is analysed and made incremental.
2. A background thread loops: check sources for new data per the trigger.
3. If data exists, **plan the batch**: record the offset range in the checkpoint's write-ahead *offset log*.
4. **Execute** a normal Spark job over only that range (reading source, updating state, computing results).
5. **Write** to the sink and then record completion in the *commit log*.
6. Repeat until failure, `stop()`, or (for `once`/`availableNow`) no more data.

### 4. Worked example
```python
from pyspark.sql import functions as F

lines = (spark.readStream.format("socket")
         .option("host", "localhost").option("port", 9999).load())

counts = (lines
          .select(F.explode(F.split("value", " ")).alias("word"))
          .groupBy("word").count())

query = (counts.writeStream
         .format("console")
         .outputMode("update")                       # or "complete"
         .trigger(processingTime="10 seconds")
         .option("checkpointLocation", "/chk/wordcount")
         .queryName("wordcount")
         .start())

print(query.status)        # {'message': 'Processing new data', 'isDataAvailable': True, ...}
print(query.lastProgress)  # numInputRows, inputRowsPerSecond, processedRowsPerSecond, durationMs, stateOperators
query.awaitTermination()
```
Output modes, applied to the word count: *complete* reprints the full table every trigger (works because aggregation state is kept anyway); *update* prints only words whose count changed; *append* is invalid here because the aggregated rows keep changing (it would be valid for an event-time window aggregation once a watermark has finalised a window). A file sink:
```python
(df.writeStream.format("parquet")
   .option("path", "/lake/events").option("checkpointLocation", "/chk/events")
   .trigger(availableNow=True).start())
```
`availableNow` processes all available data in several batches and stops — a cheap way to run "streaming" ETL on a schedule.

### 5. Under the hood
- **Checkpoint contents:** `offsets/` (planned range per batch id), `commits/` (batches finished), `metadata` (query id) and `state/` (versioned state store files per operator and partition). Deleting or changing the location makes it a new query.
- **Exactly-once recipe:** replayable source (can re-read the same range) + deterministic transformations + idempotent sink (recognise a re-executed batch id). File sink and Kafka source meet this; socket source and console sink do not.
- **Micro-batch latency:** hundreds of milliseconds to seconds, dominated by job scheduling and offset-log writes. Continuous processing mode (~ms) exists but supports only a restricted set of operations and at-least-once.
- **Incrementalisation:** the optimiser rewrites aggregations to maintain state; joins with static tables are plain batch joins per micro-batch; sorting is only allowed after aggregation in complete mode; some operations (`limit`, `distinct`, non-windowed `orderBy`) are unsupported or restricted.
- **File-source specifics:** discovers new files by listing and tracks processed file names in the checkpoint (`maxFilesPerTrigger`, `latestFirst`, `cleanSource`); it assumes files are atomically placed and not modified.
- **Kafka source:** each batch reads an offset range per partition; offsets are managed by Spark in the checkpoint, not by consumer groups.
- **Metrics:** `StreamingQueryProgress` JSON per batch includes `numInputRows`, `inputRowsPerSecond`, `processedRowsPerSecond`, `durationMs` (`addBatch`, `getBatch`, `walCommit`…), `stateOperators` (rows, memory) and source/sink offsets; can be published through Dropwizard metrics or a `StreamingQueryListener`.

### 6. Performance and design implications
- Keep `processedRowsPerSecond` ≥ `inputRowsPerSecond`; otherwise lag grows. Look at batch duration breakdown in the UI's Structured Streaming tab.
- Choose trigger interval to balance latency and efficiency; too short creates tiny batches and many small output files.
- Bound the work per batch with `maxFilesPerTrigger` / `maxOffsetsPerTrigger` to smooth bursts and recover after downtime.
- Set `spark.sql.shuffle.partitions` explicitly for streaming (the number is baked into state/checkpoint and cannot change on restart without migration).
- Prefer `foreachBatch` to reuse batch writers (JDBC, multi-sink writes) with an idempotent batch-id key.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `spark.readStream.format(...)` / `df.writeStream` | Streaming read/write | Define any query |
| `.outputMode("append"/"update"/"complete")` | Choose emitted rows | By query type |
| `.trigger(processingTime=…, once=True, availableNow=True)` | Batch cadence | Latency/cost control |
| `.option("checkpointLocation", path)` | Recovery metadata | Always in production |
| `.foreachBatch(fn)` | Custom per-batch logic | JDBC, merge/upsert, multi-sink |
| `query.status`, `lastProgress`, `recentProgress` | Inspect | Monitoring |
| `query.awaitTermination()`, `stop()` | Lifecycle | Applications |
| `maxFilesPerTrigger`, `maxOffsetsPerTrigger` | Rate limits | Backlog, burst control |
| `spark.streams.addListener(...)` | Event hooks | Alerts, metrics export |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Append mode for event logs and finalised windows | Aggregations without watermark (not allowed) |
| Update mode for dashboards fed by upsert-capable sinks | Sinks that can only append (files) |
| Complete mode for small result tables | Growing keyspaces: output and state become huge |
| `availableNow` for incremental batch | Low-latency requirements |
| Continuous trigger for ms latency | Needs tolerance for at-least-once and limited operators |

### 9. Common mistakes
- Omitting the checkpoint location (or using a temp path) → no recovery, replay from scratch → durable shared storage per query.
- Reusing one checkpoint for different queries → corrupted/confused state → one directory per query.
- Calling `start()` and letting the driver exit → query stops immediately → `awaitTermination()`.
- Expecting `show()`/`collect()` on a streaming DataFrame → AnalysisException → use a sink or `memory` for testing.
- Using schema inference on file streams in production → surprises → explicit schema.
- Treating `count` of a window as final before the watermark passes.

### 10. 🎯 Interview kit
**30-second answer:** Structured Streaming models a stream as an unbounded table and runs your DataFrame query incrementally in micro-batches. You define source, transformations, sink, output mode, trigger and checkpoint; Spark records offsets and state in the checkpoint, so it can recover and deliver end-to-end exactly-once with replayable sources and idempotent sinks.
**Structure a longer answer:** clarify latency/semantics → model (input table, result table) → five definition steps → engine loop with offset/commit logs → output modes and triggers → failure and monitoring.
**Likely follow-ups:**
- *Append vs update vs complete?* → new final rows vs changed rows vs whole table.
- *What does the checkpoint hold?* → offsets, commits, state, query id.
- *How is exactly-once achieved?* → replayable source, deterministic plan, idempotent sink.
- *Streaming vs batch trigger?* → `processingTime` vs `availableNow`.
- *Latency?* → micro-batch ~100 ms–seconds; continuous ~ms but limited.
**Signals interviewers look for:** clear mental model of micro-batch loop, correct output-mode reasoning, and attention to checkpoint/monitoring.

### 11. Check yourself
- [ ] List the five steps to define a streaming query and write one for a Kafka source.
- [ ] Explain why append mode is not allowed for an un-watermarked aggregation.
- [ ] Describe what happens on restart after a crash mid-batch.
- [ ] Read `lastProgress` and decide whether the query is keeping up.
- [ ] Say which conditions the socket source/console sink violate for exactly-once.
<!-- explainer:end -->

## References

- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — The Fundamentals of a Structured Streaming Query › Step 3: Define output sink and output mode
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Under the Hood of an Active Streaming Query
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — The Fundamentals of a Structured Streaming Query › Step 5: Start the query
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — P › Q
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Streaming Data Sources and Sinks › Writing to files
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Queries on Streaming Data › The Kappa architecture
- *Designing Data-Intensive Applications* — Martin Kleppmann — Observing Derived State › End-to-end event streams
- *Designing Data-Intensive Applications* — Martin Kleppmann — Aiming for Correctness
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Push Versus Pull Versus Poll Patterns
- *Designing Data-Intensive Applications* — Martin Kleppmann — Trust, but Verify › The end-to-end argument again

