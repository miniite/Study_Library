---
id: "02_Data_Engineering/01_Lifecycle_and_Ingestion/Message_and_Stream_Ingestion"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Message and Stream Ingestion

**L4 Advanced** · [↑ Data Engineering Lifecycle and Ingestion](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Stream ingestion consumes events from message queues or event-streaming platforms; because networks are unreliable it must be designed for out-of-order, duplicate and late events, evolving schemas, and bounded retention — and it should be adopted only when a real use case justifies the extra cost over batch.

### 1. The big picture

Almost all data is born as a stream: orders happen, sensors tick, users click. Batch ingestion is just a convenient way of cutting that stream into big chunks. Stream ingestion keeps the flow continuous, making data available to downstream systems seconds or less after it is produced ("near real time" — nothing is truly zero-latency).

The cost is complexity. Distributed messaging systems deliver messages out of order and sometimes more than once; events arrive late; payload schemas change without warning; and the platform itself needs sizing and operating. This lesson explains the core ideas — messages vs streams, ordering, delivery guarantees, replay, TTL, schema evolution, late data — and how to decide when streaming is worth it. It builds on the ingestion-considerations lesson and leads into the Kafka and stream-processing topics.

### 2. Core concepts and vocabulary

- **Message** — a single event handled individually and meant to be transient: consumed, acknowledged, removed.
- **Stream** — events appended to an ordered, persistent log that can be re-read, aggregated and combined with other streams.
- **Message queue** — broker that delivers messages to consumers and typically deletes them after acknowledgement (RabbitMQ-style).
- **Event-streaming platform** — distributed log with retention and replay (Kafka, Kinesis, Pulsar, Pub/Sub).
- **Producer / consumer (subscriber)** — writes events / reads them.
- **Ordering (FIFO)** — if A was ingested before B, A is delivered before B. Strict FIFO is expensive; many systems give best-effort ordering.
- **At-least-once vs exactly-once delivery** — messages may be delivered one or more times vs once only.
- **Idempotent processing** — processing a message many times yields the same result as once.
- **Replay** — reading a historical range of events again.
- **TTL (time to live)** — maximum time an event is retained before expiring.
- **Schema registry** — service that versions and validates event schemas.
- **Dead-letter queue (DLQ)** — holding area for events that couldn't be processed.
- **Event time vs processing/ingestion time** — when it happened vs when we saw it.
- **Micro-batch** — small batches (seconds to a minute) that approximate streaming.

### 3. How it works, step by step

1. **Producers publish** events to a topic/stream (partitioned for throughput).
2. **Broker persists** them (queue: until acknowledged; stream: until retention/TTL expires).
3. **Consumers read**, tracking position (offset/sequence). Each message carries a sequence number or offset.
4. **Receiver reorders and deduplicates.** Conceptually the receiver tracks the highest sequence it has seen *consecutively* and the highest it has *processed*; out-of-order messages wait in a buffer until the gap fills, and any message at or below the processed mark is discarded as a duplicate.
5. **Consumer processes** and acknowledges. If it crashes before acknowledging, the message is redelivered — which is why handlers must be idempotent.
6. **Bad events** go to a dead-letter queue instead of blocking the pipeline.
7. **Results** are written to storage (object store, warehouse, OLAP store) or republished to another topic.

| Property | Message queue | Event-streaming platform |
|---|---|---|
| Retention | Until consumed (transient) | Configurable; days to indefinite |
| Replay | Generally no | Yes — core capability |
| Multiple independent readers | Limited | Natural (each consumer group has its own position) |
| Typical use | Task distribution, decoupling services | Pipelines, analytics, event sourcing |

### 4. Worked example

Ingesting clickstream events from a stream into object storage in 5-minute files, with idempotency and a late-data cutoff.

```text
Config (conceptual)
  topic              clicks            (12 partitions, key = user_id)
  retention (TTL)    7 days
  consumer group     lake-writer
  delivery           at-least-once
  micro-batch        flush every 5 min or 128 MB
  late cutoff        events > 2 h behind max event_time -> late_events/ partition
  DLQ                clicks.dlq

Processing of one micro-batch
  1. read offsets 10_000..14_999 from each partition
  2. drop records whose (event_id) already appears in the last 24 h index   # dedup
  3. validate against schema v3 from registry; invalid -> clicks.dlq
  4. write parquet to lake/clicks/event_date=2026-10-12/hour=14/...         # keyed by EVENT time
  5. commit offsets only after the files are durable
```

Reading it: committing offsets *after* the write gives at-least-once; deduplication on `event_id` and partition-overwrite make the effect exactly-once from the consumer's view. Partitioning by event time (not ingestion time) keeps reports accurate when events arrive late; the cutoff bounds how long you keep re-opening old partitions.

### 5. Under the hood

**Perfect links from lossy networks.** Networks drop, duplicate and reorder messages. A "perfect link" abstraction — reliable delivery (a message sent once is eventually delivered), no duplication, no creation — is built from sequence numbers, acknowledgements, retransmission, a reordering buffer and a processed watermark. TCP is the familiar example, but its guarantee holds only within one session; when a connection resets, application-level sequence numbers and idempotency still matter.

**Ordering is expensive.** Reliable broadcast makes processes agree on *which* messages were delivered; atomic (total-order) broadcast additionally makes everyone agree on the *order*. That is essentially what consensus provides, and it costs coordination. This is why streaming platforms usually guarantee order only *within a partition* and why queue products offer a cheap best-effort mode and a stricter FIFO mode with extra overhead.

**Exactly-once is a property of the whole path.** A system may deliver a message once yet the consumer can process it fully and crash just before acknowledging — so the message is processed twice. End-to-end correctness therefore comes from idempotent or transactional sinks, not from the broker alone.

**Retention and tiering.** Messaging products differ widely: some retain up to about a week, some up to a year, and Kafka can retain indefinitely, bounded by disk or by tiering older segments to object storage. Replay is the standard read mechanism of a log; it enables both reprocessing after a bug and batch queries over a time range.

### 6. Performance and design implications

- **Throughput** depends on partition (shard) count and per-partition bandwidth; provision for peaks and use autoscaling where available.
- **TTL balance**: too short and unprocessed events vanish; too long and a growing backlog increases wait times and cost.
- **Backpressure and lag**: monitor consumer lag (latest offset − committed offset) and age of the oldest unprocessed event.
- **Small-file problem**: writing every few seconds to object storage creates tiny files; micro-batch or compact.
- **Schema safety**: version schemas centrally; make consumers tolerant of added fields.
- **Operational overhead**: self-managed streaming platforms are heavy; prefer managed services unless you have a reason.
- **Metrics**: end-to-end latency (event time → queryable), lag, DLQ rate, duplicate rate, late-event rate.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| Retention / TTL (`retention.ms`, stream retention hours) | How long events are kept | Balancing storage cost vs replay window |
| Consumer group / offset commit | Tracks a reader's position | Controlling at-least-once behaviour |
| Replay / seek to timestamp | Re-reads history from a point | Reprocessing after a bug or backfill |
| Schema registry compatibility mode | Enforces backward/forward compatibility | Preventing breaking schema changes |
| Dead-letter topic / queue | Parks failing events | Keeping the main flow unblocked |
| Partition count / key choice | Parallelism and ordering domain | Sizing throughput; per-key ordering |
| FIFO queue mode (e.g., SQS FIFO) | Stronger ordering at higher overhead | Order-sensitive workloads on queues |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| Streaming ingestion — a concrete action benefits from seconds-level data (alerts, personalisation, fraud) | Reports are daily/weekly and model training is batch anyway |
| Micro-batch — near-real-time with simpler semantics and fewer small files | Sub-second reaction is truly required |
| Event-streaming platform — replay, multiple consumers, long retention | Simple one-consumer task queue; a queue is enough |
| Managed service — small team, variable load | Cost at very high volume, or unsupported features |
| Self-managed Kafka/Flink — specialised needs and staff to run it | No one is on call to operate it |
| Strict ordering — state changes per entity | Throughput matters more and idempotent merges suffice |

Guideline from practice: adopt true real-time streaming only after a business use case justifies the extra time, money, maintenance and opportunity cost relative to batch.

### 9. Common mistakes

- **Assuming messages arrive in order** → wrong state after a late update overwrites a newer one → carry version/event-time and design for out-of-order.
- **Treating event time as ingestion time** → reports misattribute events → partition and window by event time; set a late cutoff.
- **Non-idempotent consumers** → duplicates after redelivery → dedup key or upsert/merge sink.
- **No schema governance** → a firmware or API change breaks the pipeline → registry, compatibility rules, DLQ, and talk to upstream owners.
- **Going streaming-first for everything** → cost and fragility without benefit → start from the use case.
- **TTL shorter than the worst outage** → permanent data loss → set retention above the longest expected consumer downtime.
- **Ignoring the impact on a live source** → production slowdown → prefer log-based CDC or event publication by the app.

### 10. 🎯 Interview kit

**30-second answer:** "Stream ingestion consumes events from a queue or log. Because delivery is at-least-once and unordered across partitions, I make consumers idempotent, handle late events with event-time and a cutoff, version schemas via a registry, send bad events to a dead-letter queue, and size TTL and partitions for peaks. I only choose streaming when a use case needs the latency; otherwise batch is cheaper."

**Structure a longer answer:** clarify (latency need, volume, consumers) → mechanism (log, partitions, offsets, acknowledgement, replay) → trade-offs (queue vs log, exactly-once cost, micro-batch) → scenario.

**Likely follow-ups:**
- *Message vs stream?* → Transient individual event vs persistent ordered log with replay.
- *Can you get exactly-once?* → Delivery rarely; effect yes, via idempotent/transactional sinks and offset management.
- *How to deal with late events?* → Event-time windows, watermark or cutoff, late-data partition, reprocess.
- *What if the schema changes?* → Registry with compatibility checks, tolerate unknown fields, DLQ, coordinate with producers.
- *What's replay good for?* → Fixing bugs, backfills, new consumers, batch queries over history.

**Signals interviewers look for:** realistic view of delivery semantics, event-time thinking, cost/benefit judgement about streaming, operational awareness (lag, DLQ).

### 11. Check yourself

- [ ] Contrast a message queue with an event-streaming platform on retention and replay.
- [ ] Explain how sequence numbers give FIFO order and deduplication on a lossy link.
- [ ] Describe why a consumer can process a message twice even with exactly-once delivery.
- [ ] Choose a TTL for a stream and justify both extremes' failure modes.
- [ ] Describe three defences against schema evolution breaking a pipeline.
- [ ] Argue for or against streaming-first for a weekly-reporting use case.
<!-- explainer:end -->

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Message and Stream Ingestion Considerations
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Batch versus streaming
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Message Queues and Event-Streaming Platforms
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Replay
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Key considerations for batch versus stream ingestion
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Streaming Storage
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Late-Arriving Data
- *Database Internals* — Alex Petrov — Chapter 14. Consensus › Atomic Broadcast
- *Database Internals* — Alex Petrov — Introduction and Overview › Message order
- *Designing Data-Intensive Applications* — Martin Kleppmann — Q

