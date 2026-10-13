---
id: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Batch_vs_Streaming_Features"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration"
tools: ["Apache_Spark", "Apache_Kafka", "Apache_Airflow"]
---

# Batch vs Streaming Features

**L4 Advanced** · [↑ Pipeline Orchestration](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Batch computes features over bounded data on a schedule; streaming computes them continuously from events — choose streaming only when the freshness the use case needs justifies its state, ordering, and operational complexity, and keep one definition of each feature so training and serving agree.

### 1. The big picture
"Features" are the derived values that models, alerts and dashboards consume: a 5-minute error rate, a customer's 30-day spend, an alert-correlation score. They can be computed two ways:

- **Batch:** read a bounded dataset (yesterday's events), compute, write a table. Simple, cheap, easy to re-run, but results are as old as the last run.
- **Streaming:** process each event as it arrives, maintain running state (counts, windows), emit updated values continuously. Fresh, but you now run an always-on system with state, time semantics and failure recovery.

Most organizations use both. A common pattern computes expensive history features in batch and a few time-critical ones in streaming, then serves both to the consumer. The decision is rarely "which is better" and almost always "how fresh must this value be, and what does that freshness cost to operate?"

### 2. Core concepts and vocabulary
- **Bounded vs unbounded data** — a batch has an end; a stream does not.
- **Micro-batch** — a streaming style that processes small time slices as batches (Spark Structured Streaming's default).
- **True/continuous streaming** — per-event processing (Flink, Kafka Streams).
- **Event time vs processing time** — when something happened vs when the system saw it.
- **Window** — a grouping of events by time (tumbling, sliding, session).
- **Watermark** — the engine's estimate of how far event time has progressed; decides when a window is complete and when late data is dropped and state can expire.
- **State** — data the operator remembers between events (running counts, dedup keys).
- **Trigger interval** — how often a micro-batch fires.
- **Training/serving skew** — the offline and online computations of a feature disagree.
- **Lambda / Kappa** — architectures that run batch + stream in parallel vs a single streaming path with replay.

### 3. How it works, step by step
**Deciding**
1. State the freshness requirement in business terms ("score within 2 minutes of the alert" vs "refresh nightly").
2. If tolerance is minutes, test whether frequent micro-batch scheduling (e.g. an orchestrated job every 5 minutes) is enough; it is far simpler operationally.
3. If seconds-level or event-driven, use a stream processor.
4. Check team capability: streaming needs expertise in state, checkpoints, backpressure and reprocessing.

**Streaming feature computation (e.g. Spark Structured Streaming)**
1. Read events from a log (Kafka) with an offset checkpoint.
2. Assign event time; define a watermark for lateness.
3. Aggregate in windows or by key, holding state.
4. Write results to a sink idempotently (Delta table, key-value store).
5. State is cleaned up when the watermark passes a window; without it state grows forever.

**Keeping both paths consistent**
1. Define the feature logic once (shared library/SQL) where possible.
2. Backfill the streaming feature from the batch history so it starts warm.
3. Compare online and offline values periodically.

### 4. Worked example
Feature: number of distinct alerts per host in the last 10 minutes.

Batch version (runs every hour):
```sql
SELECT host, window_start, COUNT(DISTINCT alert_id) AS alerts_10m
FROM alerts
GROUP BY host, window(event_time, '10 minutes');
```
Streaming version (Spark Structured Streaming):
```python
from pyspark.sql import functions as F

features = (alerts_stream
    .withWatermark("event_time", "5 minutes")
    .groupBy("host", F.window("event_time", "10 minutes"))
    .agg(F.approx_count_distinct("alert_id").alias("alerts_10m")))

(features.writeStream
    .outputMode("update")
    .option("checkpointLocation", "s3://chk/alerts_10m")
    .trigger(processingTime="30 seconds")
    .toTable("features.alerts_10m"))
```
What to look at: the watermark bounds state — events more than 5 minutes late are dropped; the checkpoint lets the query resume; the 30-second trigger sets latency. If the query falls behind, check three things: input rate (events/s) vs processed rate, batch duration vs trigger interval, and state store size.

### 5. Under the hood   *(optional at L4)*
- **Micro-batch execution:** each trigger plans a batch from new offsets, runs it with the same engine as batch jobs, commits offsets and state. This reuses batch optimizations and gives exactly-once with idempotent sinks, at the cost of latency floor equal to batch overhead.
- **State stores:** keyed state is held in memory with changelog/checkpoint on durable storage; large state needs RocksDB-style stores and careful TTLs.
- **Event-time correctness:** processing-time results are easy but depend on arrival jitter and reruns; event-time is reproducible but needs watermarks and late-data policy. Edge cases (backfill, out-of-order sources) often force event time.
- **Cost model:** batch pays only while running; streaming pays continuously (though you can autoscale or use triggers like "available now" to run stream code as batch).

### 6. Performance and design implications
- Streaming latency is dominated by trigger interval, shuffle, state size and sink commit time.
- Unbounded state is the classic failure; always combine watermarks with state expiry, and bound keys that never "finish" (e.g. alerts that never resolve) with a TTL.
- Batch is easiest to test, replay and backfill; streaming needs replayable logs and idempotent sinks.
- Operational burden counts: on-call, upgrades with state compatibility, schema evolution, monitoring lag.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `withWatermark(col, delay)` | Defines lateness and state expiry | Any windowed streaming aggregate |
| `trigger(processingTime=…)` / `availableNow` | Sets micro-batch cadence or runs stream as batch | Tune latency/cost; backfill with streaming code |
| `checkpointLocation` | Persist offsets/state | Required for recovery |
| Output mode `append/update/complete` | What is emitted per trigger | Match sink semantics |
| Orchestrated micro-batch (Airflow every N min) | Near-real-time without a stream engine | Minutes-level freshness |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Batch — daily/hourly freshness is enough, heavy joins over history | Decisions depend on events within seconds |
| Orchestrated micro-batch — minutes, small team | Intervals shorter than job startup overhead |
| Streaming — low-latency actions (alerts, fraud, personalization) | Team lacks streaming expertise and the value of seconds is unproven |
| Event time — correctness under replay | Only rough monitoring needed; processing time is fine |
| One shared feature definition | Online and offline implementations written separately without tests |

### 9. Common mistakes
- Choosing streaming because "real-time" was said → needless complexity → confirm actual latency tolerance.
- No watermark → state grows until OOM → set watermark and TTL.
- Duplicate events from at-least-once sources → inflated features → dedupe by event id within the watermark.
- Different logic online and offline → training/serving skew → share code and monitor differences.
- Treating lag as a mystery → wasted time → compare input rate, batch time, state size.

### 10. 🎯 Interview kit
**30-second answer:** Batch computes features over bounded data on a schedule; streaming computes them continuously with state. I start from the required freshness; if minutes are fine I use frequent micro-batches, and only use true streaming when seconds matter and the team can operate it, using event time, watermarks and idempotent sinks, and sharing logic with batch to avoid skew.

**Structure a longer answer:** clarify freshness and volume → batch option → micro-batch option → streaming option → state/time/failure concerns → decision and cost.

**Likely follow-ups:**
- *How do you tell why a streaming job lags?* → trigger interval vs batch duration, input rate, state size.
- *Event time vs processing time default?* → event time for correctness, processing time for simple monitoring; exceptions for late/unordered sources.
- *How to bound state?* → watermarks, TTL, key cardinality limits.
- *When re-architect batch to streaming?* → when freshness has measurable business value exceeding operational cost.

**Signals interviewers look for:** skepticism about "real-time", awareness of state and time semantics, and operational thinking.

### 11. Check yourself
- [ ] Explain micro-batch vs continuous streaming.
- [ ] Describe what a watermark does and what happens to late data.
- [ ] List three reasons a streaming job falls behind and how you'd check each.
- [ ] Say how you would prevent training/serving skew.
<!-- explainer:end -->

## Go deeper

- [Choosing Processing Engines](<Choosing_Processing_Engines.md>)

## Implementations (tools)

- [Apache Spark](<../../../../TOOLS/Apache_Spark/_index.md>)
- [Apache Kafka](<../../../../TOOLS/Apache_Kafka/_index.md>)
- [Apache Airflow](<../../../../TOOLS/Apache_Airflow/_index.md>)

## Practice questions

- [Basic] [What is the difference between batch processing and stream processing when handling operational telemetry?](<../../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13377_What_is_the_difference_between_batch_processing_and_stream_p.md>)
- [Deep] [When would you compute alert-correlation features in a batch Spark job versus in a streaming job and what breaks if you pick wrong for a paging-critical use case?](<../../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13449_When_would_you_compute_alert-correlation_features_in_a_batch.md>)
- [Deep] [How would you compute a rolling 7-day alert-rate-per-service feature efficiently in Spark without reprocessing the full history every night?](<../../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13471_How_would_you_compute_a_rolling_7-day_alert-rate-per-service.md>)
- [Deep] [How do you decide when the batch alert-correlation pipeline should be re-architected into streaming and what does that cost operationally for an on-call team?](<../../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13510_How_do_you_decide_when_the_batch_alert-correlation_pipeline_.md>)
- [Deep] [How do you implement watermarking in Spark Structured Streaming to bound state for a windowed aggregation and what happens to events that arrive past the watermark?](<../../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13681_How_do_you_implement_watermarking_in_Spark_Structured_Stream.md>)
- [Deep] [How do you handle small-file problems produced by a streaming-to-batch feature pipeline that hurt downstream Spark read performance?](<../../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13162_How_do_you_handle_small-file_problems_produced_by_a_streamin.md>)
- [Deep] [How do you decide whether to invest in a real-time streaming feature platform now versus keeping batch Spark and revisiting later; what business signals drive that?](<../../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13282_How_do_you_decide_whether_to_invest_in_a_real-time_streaming.md>)
- [Expert] [When would you choose Flink over Spark Structured Streaming for a stateful streaming pipeline and what operational tax does Flink add?](<../../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13650_When_would_you_choose_Flink_over_Spark_Structured_Streaming_.md>)

## Related concepts

- [Streaming-first Data Architecture](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Streaming-first_Data_Architecture.md>) *(Data Architecture)*
- [Structured Streaming Micro-batches](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Structured_Streaming_Micro-batches.md>) *(Big Data Processing)*
- [Batch vs Streaming Cost Trade-offs](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_vs_Streaming_Cost_Trade-offs.md>) *(Batch vs Stream)*

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Streaming
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › Message Queues and Event-Streaming Platforms
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Data Generation in Source Systems › Scalability
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Storage › Streaming Storage
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Chapter 8. Structured Streaming
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Chapter 8. Structured Streaming › The Advent of Micro-Batch Stream Processing
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Chapter 8. Structured Streaming › Lessons Learned from Spark Streaming (DStreams)
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Chapter 8. Structured Streaming › The Philosophy of Structured Streaming
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Data Engineering Fundamentals › 2. Data Engineering Fundamentals
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Data Ingestion Patterns in Data Engineering › 4. Data Ingestion Patterns in Data Engineering
- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Stream Processing Patterns › 7. Stream Processing Patterns
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Streaming In and Out of Your Delta Lake › Streaming Versus Batch Processing
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Streaming In and Out of Your Delta Lake › Streaming terminology
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Architecting Your Lakehouse › For Delta Lake and Spark Structured Streaming
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Successful Design Patterns › Delta and Flink in Harmony
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Data Engineering != Spark › Batch and Real-Time Systems
- *97 Things Every Data Engineer Should Know* — Tobias Macey (ed.) — Streaming Is Different from Batch › Chapter 66. Streaming Is Different from Batch
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Prerequisite Knowledge for Aspiring Data Engineers › Flink
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Prerequisite Knowledge for Aspiring Data Engineers › Real-Time Stream Processing
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Overview of AWS Analytics and Auxiliary Services › AWS Analytics Services
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Amazon EventBridge › Practice Questions

