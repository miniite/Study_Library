---
id: "TOOLS/Apache_Spark/03_Spark_Streaming/Stateful_Streaming"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Stateful Streaming Aggregations and Joins

**L4 Advanced** · [↑ Structured Streaming](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Stateful streaming keeps summaries of past events (counts, windows, join buffers, user sessions) in a fault-tolerant state store, and watermarks tell Spark how late data may arrive so old state can be dropped.

### 1. The big picture
Stateless operations (`select`, `filter`, `map`) treat every record independently. Anything that needs history — running counts, windowed aggregates, joining two streams, deduplication, sessionisation — must remember something from earlier micro-batches. That memory is **state**: it lives in executor memory, is versioned and checkpointed to storage per batch, and is recovered after failure.

State creates the central design problem of streaming: without a rule for when state is "old", it grows forever. Structured Streaming solves this with **event time** and **watermarks**: you declare how late data can be, and Spark finalises windows and evicts state accordingly.

### 2. Core concepts and vocabulary
- **State store** — per-partition, per-operator key→value store (default: HDFS-backed in-memory maps; RocksDB option) with versions per batch.
- **Event time** — timestamp embedded in the record; correctness is defined in event time.
- **Window** — time bucket for aggregation: *tumbling* (fixed, non-overlapping) or *sliding* (overlapping via slide interval).
- **Watermark** — `max(event time seen) − delay threshold`; data older than the watermark is considered too late and state for windows ending before it is dropped.
- **Managed stateful operations** — Spark cleans state: streaming aggregations, stream-stream joins, deduplication.
- **Unmanaged stateful operations** — you define state and cleanup: `mapGroupsWithState`, `flatMapGroupsWithState`.
- **State timeout** — processing-time or event-time timeout that lets arbitrary-state logic expire a key.
- **Stream–stream join** — joining two unbounded inputs, buffering rows from each side.
- **Stream–static join** — joining a stream with a fixed table; stateless.

### 3. How it works, step by step
**Windowed aggregation with watermark**
1. Each record's event time is mapped to its window(s) (`window("eventTime", "5 minutes")`).
2. The aggregate for (key, window) is updated in the state store.
3. At the end of each batch the watermark is advanced to `max(eventTime) − threshold`.
4. Windows whose end is before the watermark are considered final: in *append* mode their result is emitted once; their state is deleted.
5. Records arriving later than the watermark for a finalised window are dropped.

**Stream–stream join**
1. Buffer rows from both sides as state, keyed by the join key.
2. For each new row, probe the other side's buffer and emit matches.
3. Watermarks plus an event-time range condition (e.g. click within 1 hour after impression) define when buffered rows can never match again, enabling cleanup; *outer* joins emit null-padded rows only once that is known.

**Arbitrary state**
1. Group by key; for each batch Spark calls your function with `(key, new values, state)`.
2. You update or remove the state, optionally set a timeout, and emit zero/one/many rows.
3. When the timeout fires without new data, the function is called with an empty iterator so you can finalise (e.g. close a session).

### 4. Worked example
```python
from pyspark.sql import functions as F

readings = (spark.readStream.format("kafka")
            .option("kafka.bootstrap.servers", "k:9092").option("subscribe", "sensors").load()
            .select(F.from_json(F.col("value").cast("string"), "sensorId STRING, eventTime TIMESTAMP, v DOUBLE").alias("r"))
            .select("r.*"))

# 1) Tumbling 5-minute windows, tolerate 10 minutes of lateness
agg = (readings
       .withWatermark("eventTime", "10 minutes")
       .groupBy("sensorId", F.window("eventTime", "5 minutes"))
       .agg(F.count("*").alias("n"), F.avg("v").alias("avg_v")))

(agg.writeStream.outputMode("append")        # window emitted once, after watermark passes
    .format("parquet").option("path", "/lake/sensor_5m")
    .option("checkpointLocation", "/chk/sensor_5m").start())

# 2) Stream-stream outer join: impressions with optional clicks within 1 hour
imp = impressions.withWatermark("impressionTime", "2 hours")
clk = clicks.withWatermark("clickTime", "3 hours")
joined = imp.join(clk,
    F.expr("""clickAdId = impressionAdId AND
              clickTime BETWEEN impressionTime AND impressionTime + interval 1 hour"""),
    "leftOuter")

# 3) Deduplicate events
dedup = readings.withWatermark("eventTime", "10 minutes").dropDuplicates(["sensorId", "eventTime"])
```
Reading it: with *update* mode the same aggregation would emit running results per window before finalisation; with *append* nothing appears for a window until the watermark passes its end (plus delay). In the left outer join, an impression's null-padded row appears only after the watermark guarantees no click can still arrive. In the Spark UI's Structured Streaming tab and in `lastProgress.stateOperators` watch `numRowsTotal`, `memoryUsedBytes` and `numRowsDroppedByWatermark`; steady state should plateau, not climb.

### 5. Under the hood
- **State store mechanics:** each shuffle partition has its own store instance; per batch Spark loads the last version, applies updates, and writes a delta (or snapshot every N versions) to the checkpoint. Recovery replays the last committed version. The default provider keeps all state in executor JVM heap; the RocksDB provider (Spark 3.2+) stores it off-heap on local disk, supporting far larger state with lower GC pressure.
- **Watermark semantics.** The watermark is computed at the end of a batch and applied to the *next*, is monotonic, and with several inputs the global watermark is the **minimum** across streams by default (`spark.sql.streaming.multipleWatermarkPolicy`). The delay guarantees that data up to that late is processed; later data *may* be dropped (best-effort, not guaranteed dropped).
- **Output mode interplay:** append requires a watermark for aggregations; complete keeps all state forever (no watermark cleanup); update emits changed keys and also cleans state with a watermark.
- **Joins:** inner joins work without watermarks (state grows unbounded); outer joins require both the watermark and time-range constraint; unsupported combinations (e.g. full outer join between streams in older versions, certain chains after joins) raise analysis errors. Matches are emitted as data arrives, not in order.
- **Arbitrary state functions** run per key per batch on executors; state is serialised with encoders you define (case class), so schema changes between versions require care.
- **State and restarts.** Changing the aggregation keys, number of shuffle partitions, or the state schema is not allowed with an existing checkpoint; plan state versioning.

### 6. Performance and design implications
- State size = keys × (window count or buffer duration) × per-key bytes. Bound it with watermarks, narrow windows, and key cardinality control.
- Use RocksDB for large state; watch GC time and `memoryUsedBytes`.
- Pre-aggregate or filter early; avoid keys with unbounded cardinality (e.g. raw user ID × tiny window).
- Choose watermark delay from real lateness distribution: too small drops valid data, too large inflates state and delays append output.
- Time-range join conditions should be as tight as business logic allows.
- Monitor `numRowsDroppedByWatermark`, state rows, batch duration and checkpoint write time.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `.withWatermark("eventTime", "10 minutes")` | Define lateness | Before windowed agg, dedup, outer joins |
| `window(col, "5 minutes", "1 minute")` | Tumbling/sliding windows | Event-time aggregates |
| `dropDuplicates([...])` with watermark | Streaming dedup | At-least-once sources |
| `groupByKey(...).mapGroupsWithState(...)` / `flatMapGroupsWithState` | Custom state | Sessionisation, state machines |
| `GroupStateTimeout.ProcessingTimeTimeout` / `EventTimeTimeout` | Expire idle state | Cleanup in custom state |
| `spark.sql.streaming.stateStore.providerClass` | State store backend (RocksDB) | Large state |
| `spark.sql.streaming.multipleWatermarkPolicy` | min/max of watermarks | Multi-input queries |
| `lastProgress["stateOperators"]` | State metrics | Monitoring growth |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Watermark + append for final, immutable window results | You need early, speculative results → update mode |
| Update mode for dashboards/upserts | File sinks that cannot update rows |
| Built-in aggregations/joins | Logic that doesn't fit SQL → custom state (only Scala/Java as of 3.0) |
| RocksDB state store | Small state: default provider is simpler and faster |
| Generous watermark for unreliable devices | Longer delay → more state and later output |

### 9. Common mistakes
- Aggregating by event time without a watermark → state grows forever → add `withWatermark` before the aggregation (on the same column used by the window).
- Applying the watermark after the aggregation → no effect → place it on the input stream.
- Outer join without watermarks/range condition → analysis error or unbounded state.
- Using complete mode for high-cardinality windows → memory blow-up.
- Changing shuffle partitions or grouping keys then restarting from the old checkpoint → failure or corrupted semantics → new checkpoint or state migration.
- Assuming late rows are always dropped → they are dropped only if older than the watermark at processing time.

### 10. 🎯 Interview kit
**30-second answer:** Stateful streaming operations keep summaries of past data in a checkpointed state store; watermarks bound how late data can arrive so Spark can finalise windows, emit results in append mode and delete old state. Built-in managed operations (windows, stream-stream joins, dedup) clean themselves; arbitrary-state operators let you define state and timeouts.
**Structure a longer answer:** clarify event time vs processing time → state and why it grows → watermark definition and effect → windows/joins/dedup → arbitrary state and timeouts → operations (size, RocksDB, restarts).
**Likely follow-ups:**
- *What is a watermark?* → max event time minus allowed delay.
- *Why must outer stream joins have watermarks?* → to know when a null result is final.
- *Append vs update with windows?* → final once vs revised.
- *How do you handle huge state?* → RocksDB, narrower windows, TTL, key design.
- *What cannot change between restarts?* → state schema, keys, shuffle partitions.
**Signals interviewers look for:** correct watermark mechanics, explicit trade-off between lateness and state/latency, and operational awareness.

### 11. Check yourself
- [ ] Compute the watermark after a batch with max event time 12:07 and delay 10 minutes, and say which 5-minute windows are final.
- [ ] Write an outer join between impressions and clicks with the necessary constraints.
- [ ] Explain why the same query in complete mode never frees state.
- [ ] Choose between `mapGroupsWithState` and a built-in window for sessionisation.
- [ ] List the state metrics you would alert on.
<!-- explainer:end -->

## References

- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Data Transformations › Stateful Streaming Aggregations
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Streaming Joins
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Aggregations with Event-Time Windows
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Queries on Streaming Data › Stream-to-stream joining
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Arbitrary Stateful Computations
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Streaming Joins › Inner joins with optional watermarking
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — S
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Streaming Joins › Outer joins with watermarking
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Data Transformations › Types of stateful operations
- *Designing Data-Intensive Applications* — Martin Kleppmann — Observing Derived State › Reads are events too

