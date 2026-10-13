---
id: "03_Big_Data_and_Streaming/02_Stream_Processing"
level: 3
type: "curriculum"
curriculum: "data_engineering"
---

# Stream Processing

**L3 Practitioner** · [↑ Big Data and Streaming](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Stream processing runs computation continuously over unbounded sequences of events as they arrive, instead of waiting for a finished file — trading the simplicity of batch for low latency and the hard problems of time, ordering and failure.

### 1. The big picture

Batch jobs (see *Batch Processing*) assume the input is a finite file: read all of it, produce output, stop. But most real data is **unbounded** — users keep clicking, sensors keep reporting, orders keep arriving. Waiting for "the end of the day" to process it adds a day of delay. **Stream processing** removes the artificial boundary: every event is processed shortly after it happens, and the job never finishes.

Think of it as batch with the time slices made very small. A daily batch job and a stream job can compute the same thing; the stream job just updates its answer continuously.

This area covers, in order:

1. **Event streams and messaging** — how events travel from producers to consumers (brokers, queues, partitioned logs).
2. **Databases and streams** — treating a database's changes as a stream (CDC, event sourcing) so derived systems stay in sync.
3. **Time** — event time vs processing time, and the windows that group an unbounded stream into finite chunks.
4. **Joins** — combining streams with streams and tables.
5. **Fault tolerance** — how a never-ending job survives crashes without double counting.

Typical uses are monitoring and alerting, fraud detection, search-index and cache updates, real-time dashboards, and keeping a data warehouse or lakehouse fresh via change data capture.

### 2. Core concepts and vocabulary

- **Event** — a small, immutable record of something that happened at a point in time (a click, a sensor reading, a database write). The streaming analogue of a record in a batch file.
- **Producer / publisher** — the component that creates events. **Consumer / subscriber** — one that reads them.
- **Topic / stream** — a named group of related events, like a file name in the batch world.
- **Unbounded data** — data with no defined end; the opposite of a bounded file.
- **Message broker** — a server that sits between producers and consumers and buffers events.
- **Log-based broker** — a broker that stores events in an append-only, ordered, partitioned log (e.g., Kafka).
- **Change data capture (CDC)** — extracting each insert/update/delete from a database as an event stream.
- **Event time vs processing time** — when the event happened vs when the system handled it.
- **Window** — a finite slice of a stream (fixed, sliding, session) over which an aggregate is computed.
- **Backpressure** — slowing producers down when consumers cannot keep up.
- **Exactly-once** — a processing guarantee that each event affects the result exactly one time even across failures (usually achieved by atomic commit or idempotence).

### 3. How it works, step by step

A generic streaming pipeline:

1. **Capture** — an application, sensor or database log emits events (a click, a row change).
2. **Transport** — events are written to a broker topic. The broker stores them durably and orders them within a partition.
3. **Process** — a stream processor reads the topic, applies filters, transformations, aggregations over windows and joins, keeping **state** (counters, windows, lookup tables) between events.
4. **Output** — results are written to another topic, a database, a search index, a cache, a dashboard or an alerting system.
5. **Recover** — periodically the processor saves a checkpoint of its state and input position; after a crash it resumes from the last checkpoint instead of starting from scratch.

Compare with batch:

| | Batch | Stream |
|---|---|---|
| Input | Bounded file/table | Unbounded sequence of events |
| Latency | Minutes to hours | Milliseconds to seconds |
| Trigger | Schedule / manual run | Arrival of each event |
| Failure handling | Rerun the whole job | Checkpoints and replay |
| Time | Usually "data for yesterday" | Event time, windows, lateness |
| State | Output files | Continuously maintained state |

### 4. Worked example

A web shop wants a live "orders per minute by country" dashboard.

```sql
-- Streaming SQL-style definition (e.g., Flink SQL / ksqlDB-like syntax)
SELECT country,
       TUMBLE_START(order_time, INTERVAL '1' MINUTE) AS window_start,
       COUNT(*)                                      AS orders
FROM   orders_stream
GROUP  BY country, TUMBLE(order_time, INTERVAL '1' MINUTE);
```

What happens internally:

1. Each order is published to the `orders` topic, partitioned by `order_id` or `country`.
2. The processor keeps a counter per (country, minute) window in local state.
3. An event with `order_time = 10:03:42` increments the 10:03 counter for its country.
4. When the **watermark** (the system's estimate of "no earlier events still to come") passes 10:04:00, the 10:03 window is closed and its count is emitted.
5. A late order stamped 10:03:58 that arrives at 10:04:30 is either added (if a lateness allowance exists) or dropped/diverted.

The same result in batch would be a nightly query; the stream version is always within seconds of the truth.

### 5. Under the hood *(optional for L3)*

Two ideas explain most of stream processing's design:

- **The log as the central abstraction.** An append-only, ordered log can feed many independent consumers, be replayed from any position, and be used to rebuild state. Databases already use it (the write-ahead and replication logs); log-based brokers expose it as a public API.
- **State plus replay equals fault tolerance.** Operators keep local state; checkpoints and replayable input mean a crashed task can be restored and re-run deterministically.

A processor's local state is typically small enough to be checkpointed to durable storage or rebuilt by replaying a compacted changelog topic.

### 6. Performance and design implications

- **Throughput vs latency**: batching events in small micro-batches raises throughput; per-event processing lowers latency.
- **Partitioning** determines parallelism: one partition is read by one consumer in a group, so max parallelism = partition count.
- **State size** drives memory and checkpoint cost; unbounded state (e.g., "distinct users forever") must be bounded with windows or TTLs.
- **Backpressure** must propagate end to end or queues grow without bound.
- **Monitor consumer lag** (how far behind the head of the log) — the main health indicator.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| Consumer group id | Shares partitions among consumer instances | Scaling a consumer horizontally |
| Partition count | Upper bound on parallelism | Sizing a topic |
| Retention (time/size) | How long the broker keeps events | Replay and recovery windows |
| Checkpoint interval | How often state is saved | Trade recovery time vs overhead |
| Watermark delay / allowed lateness | How long to wait for late events | Balancing completeness and latency |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| Decisions or user experience depend on fresh data | A daily report is good enough — batch is simpler and cheaper |
| You need to react to events (alerts, fraud) | Results must be perfectly complete and final at one instant |
| You maintain derived views (search, cache) from a database | The pipeline has no owner able to operate a 24/7 system |
| Input volume is steady and you want smooth load | Logic needs repeated global scans of full history (use batch) |

### 9. Common mistakes

- **Using processing time instead of event time** → results change when the pipeline is delayed or replayed → window on event timestamps.
- **Unbounded state** → memory grows until the job dies → use windows, TTLs or compaction.
- **Assuming exactly-once end to end** → duplicates appear in an external sink → make writes idempotent or use transactional sinks.
- **Treating the stream as a database** → ad-hoc queries over history are slow → keep a table/serving store for lookups.
- **Ignoring skew** → one hot key overloads one partition → re-key or pre-aggregate.

### 10. 🎯 Interview kit

**30-second answer:** Stream processing handles unbounded data continuously: events are published to a broker, a processor maintains state and windows over them, and results are emitted with seconds of latency. The hard parts are time (event vs processing time, late data), state and fault tolerance (checkpoints, replay), and delivery guarantees.

**Structure a longer answer:** clarify the latency need → source/transport (log-based broker) → processing model (windows, state) → guarantees (at-least-once + idempotence vs exactly-once) → trade-offs vs batch → a real scenario.

**Likely follow-ups:**
- *Batch vs stream?* → Bounded vs unbounded input; stream gives low latency but needs state, time and failure handling.
- *What is a watermark?* → The system's estimate that no events older than T remain; used to close windows.
- *How do you avoid duplicates?* → Idempotent writes, transactional output, or dedupe keys.
- *Lambda vs Kappa?* → Lambda keeps separate batch and stream paths; Kappa replays the log through one streaming path.

**Signals interviewers look for:** you reason about time and ordering, name concrete guarantees, and don't present streaming as strictly better than batch.

### 11. Check yourself

- [ ] Explain, with an example, why an unbounded dataset needs windows.
- [ ] Draw a pipeline from a database to a search index using a broker and a stream processor.
- [ ] Describe what a processor does when it crashes mid-window.
- [ ] State two problems that appear in streaming but not in batch.
<!-- explainer:end -->

## In this section (study in order)

1. [Event Streams and Messaging Systems](<Event_Streams_and_Messaging/_index.md>) (L4 Advanced)
2. [Partitioned Logs](<Partitioned_Logs/_index.md>) (L4 Advanced)
3. [Databases and Streams](<Databases_and_Streams/_index.md>) (L4 Advanced)
4. [Uses of Stream Processing](<Stream_Processing_Uses/_index.md>) (L4 Advanced)
5. [Reasoning About Time in Streams](<Time_in_Streams/_index.md>) (L4 Advanced)
6. [Stream Joins](<Stream_Joins/_index.md>) (L4 Advanced)
7. [Fault Tolerance and Exactly-Once](<Stream_Fault_Tolerance/_index.md>) (L4 Advanced)

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 11. Stream Processing › Transmitting Event Streams
- *Designing Data-Intensive Applications* — Martin Kleppmann — Databases and Streams
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Queries on Streaming Data › Windows, triggers, emitted statistics, and late-arriving data
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Arbitrary Stateful Computations › Processing-time timeouts
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › Change Data Capture
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Change Data Capture
- *Database Internals* — Alex Petrov — Introduction and Overview › Processing
- *Designing Data-Intensive Applications* — Martin Kleppmann — Encoding and Evolution › Message brokers
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Security and Privacy › Logging, Monitoring, and Alerting
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Serving Data for Analytics, Machine Learning, and Reverse ETL › Embedded Analytics

