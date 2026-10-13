---
id: "03_Big_Data_and_Streaming/02_Stream_Processing/Stream_Processing_Uses"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Uses of Stream Processing

**L4 Advanced** · [↑ Stream Processing](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Stream processing is used wherever results must follow events within seconds — monitoring and alerting, complex event pattern detection, continuous analytics, keeping derived systems fresh and materialized views — and it is often combined with batch in lambda-style architectures.

### 1. The big picture

Once events can be transported durably and in order (see *Event Streams and Messaging*, *Partitioned Logs*), you need something to *do* things with them: transform, aggregate, join, filter, and write results elsewhere. That is stream processing. Conceptually it is the streaming analogue of a batch job — read events, produce output — except it runs continuously and its output is itself a stream or an incrementally updated table.

The same operators serve different goals. This lesson organizes the common uses so you can recognize which pattern a requirement belongs to, and which tools fit.

### 2. Core concepts and vocabulary

- **Stream processor / operator** — a continuously running job (Flink, Spark Structured Streaming, Kafka Streams, Beam).
- **Stateless transformation** — per-event map/filter with no memory.
- **Stateful transformation** — aggregations, joins, deduplication that keep state.
- **Complex event processing (CEP)** — detecting patterns across events (a sequence, a count in a time frame).
- **Stream analytics** — aggregations and statistics over windows.
- **Approximate algorithms** — sketches (HyperLogLog, Bloom filters, Count-Min) trading accuracy for memory.
- **Materialized view maintenance** — keeping a cache/index/table updated from a change stream.
- **Lambda architecture** — a batch layer (accurate, slow) plus a speed layer (approximate, fast), merged at query time.
- **Kappa architecture** — a single streaming path; reprocess history by replaying the log.
- **Continuous query** — a query that stays registered and emits results as data arrives (the inverse of a stored query on changing data).

### 3. How it works, step by step

Common usage patterns, each with its pipeline:

**A. Monitoring and alerting (stream analytics)**
1. Metrics/logs flow into a topic.
2. A windowed aggregate (p99 latency per service per minute, error rate) is computed.
3. A threshold rule emits an alert event to a notification sink.

**B. Complex event processing**
1. Rules are stored as patterns, e.g. "login from country A then a purchase from country B within 10 minutes".
2. Incoming events are matched against pattern state machines (like a regular expression over events).
3. Matches produce a complex event (fraud alert, security incident).
CEP inverts a database query: queries are long-lived, events flow through them.

**C. Keeping derived systems in sync**
1. CDC events from the OLTP database enter a log.
2. Consumers update a search index, cache, or warehouse table.

**D. Materialized views and caches**
1. A processor maintains an aggregated table (e.g., counts per product) incrementally.
2. Applications query the view; it is updated as events arrive.

**E. Real-time data integration / ELT**
1. Ingest from sources, clean/validate/enrich in-flight.
2. Land in lake or warehouse in micro-batches or continuous streams.

**F. Reprocessing and unified batch/stream**
1. Keep the log long enough to replay.
2. Run new logic from the earliest offset to rebuild outputs (Kappa), or run batch for history and stream for recency (Lambda).

### 4. Worked example

Fraud rule + aggregation in one Flink SQL job:

```sql
-- Continuous aggregation: card spend per 10-minute window
CREATE VIEW spend AS
SELECT card_id, TUMBLE_END(txn_time, INTERVAL '10' MINUTE) AS w_end, SUM(amount) AS total
FROM transactions
GROUP BY card_id, TUMBLE(txn_time, INTERVAL '10' MINUTE);

-- Alert stream: more than 5,000 in a window
INSERT INTO alerts
SELECT card_id, w_end, total FROM spend WHERE total > 5000;

-- CEP: two purchases in different countries within 10 minutes
SELECT * FROM transactions
MATCH_RECOGNIZE (
  PARTITION BY card_id ORDER BY txn_time
  MEASURES A.country AS c1, B.country AS c2
  PATTERN (A B) WITHIN INTERVAL '10' MINUTE
  DEFINE B AS B.country <> A.country
);
```

Both queries run forever; every incoming transaction may trigger a result within milliseconds to seconds. The same business logic in batch would find fraud the next morning.

### 5. Under the hood

- **Approximate answers are acceptable.** Probabilistic structures such as HyperLogLog (cardinality), Bloom filters (membership) and percentile sketches keep tiny bounded state. They are chosen for memory efficiency, not because streaming is inherently approximate — exact stream processing exists.
- **Search on streams.** Full-text search can be inverted: queries are indexed and each document streams through, matching stored queries (media monitoring alerts).
- **Event-driven vs request-driven.** Stream processors resemble actors/microservices communicating by messages, but with durable logs, fault tolerance and ordering built in. Unlike RPC, processing is asynchronous and decoupled.
- **Lambda architecture** (batch + speed layer) was popular because early streaming engines were less accurate. Downsides: two codebases to keep equivalent, merging results at query time. Modern engines with exactly-once semantics and replayable logs let one codebase serve both (Kappa, or Beam/Flink's unified API).
- **Lifecycle perspective.** Streaming appears across the data engineering lifecycle: ingestion (from event buses), transformation (in-flight enrichment), serving (materialized views, reverse ETL). Treat batch as a special case of bounded streams.
- **Queries on streaming data.** Basic patterns include filter/projection, windowed aggregation, joins with reference tables, and top-N / deduplication. The key difference from batch is that the query engine is *triggered by the data* instead of by an external schedule.
- **Infrastructure.** Production streaming needs monitoring of lag, checkpoints and state size; pipelines should be deployable as code with versioned schemas.

### 6. Performance and design implications

- Choose the latency target first: seconds-level micro-batches are far cheaper to run than milliseconds-level per-event processing.
- Stateful operators require capacity planning of state (keys × window).
- Use approximations when exactness costs unbounded memory (distinct counts).
- Backfills must reuse the same logic or results diverge between historical and live paths.
- Alert on pipeline health (lag, checkpoint failures), not only business metrics.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| Flink SQL `MATCH_RECOGNIZE` | Pattern detection | CEP use cases |
| Spark `writeStream.trigger(...)` | Micro-batch cadence | Latency vs cost |
| Kafka Streams `KTable` / `aggregate` | Materialized state | Views from changelogs |
| `approx_count_distinct` / HLL | Approximate distinct counts | High-cardinality metrics |
| Kafka consumer `auto.offset.reset=earliest` | Start from the beginning | Reprocessing |
| Savepoints (Flink) | Versioned state snapshots | Upgrades and migrations |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| Value decays with delay (fraud, ops alerts, personalization) | Daily granularity is sufficient |
| Continuous derived views for apps | Logic needs full-history global computation each time |
| **Kappa** with a replayable log | Log retention is short or reprocessing is huge |
| **Lambda** when batch accuracy is a hard requirement | Two code paths can't be kept consistent |
| **Approximate** analytics for scale | Exact figures are legally or financially required |

### 9. Common mistakes

- **Using streaming for everything** → needless operational complexity → batch if hours of latency are fine.
- **Separate logic for batch and stream** → results diverge → shared code or replay.
- **Unbounded state in distinct counts** → memory blowup → sketches or windows.
- **Alert storms** from noisy thresholds → fatigue → hysteresis, dedupe, windows.
- **No backfill plan** → fixes require manual reprocessing → keep the log and idempotent sinks.

### 10. 🎯 Interview kit

**30-second answer:** Stream processing is used when results must reflect events within seconds: monitoring and alerting, fraud and pattern detection (CEP), continuous analytics, maintaining search indexes/caches/materialized views from change streams, and real-time ELT. It is often compared with batch and combined with it in Lambda or replaced with a single streaming path in Kappa.

**Structure a longer answer:** list the use-case families → one pipeline per family → state/time/guarantee implications → Lambda vs Kappa → trade-offs.

**Likely follow-ups:**
- *Lambda vs Kappa?* → Two paths vs a single replayable stream.
- *When use approximations?* → Large cardinalities or percentiles with bounded memory.
- *How is CEP different from a SQL query?* → Standing patterns over events vs ad-hoc queries over stored data.
- *How do you reprocess after a bug?* → Replay from the log with new code and compare.

**Signals interviewers look for:** matches use case to pattern, discusses batch vs stream honestly, notes operational burden.

### 11. Check yourself

- [ ] Name five uses of stream processing and the output of each.
- [ ] Explain how CEP "inverts" a query.
- [ ] Describe Lambda vs Kappa and a drawback of each.
- [ ] Choose a data structure for counting distinct users in a 1-hour window at massive scale.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Processing Streams › Uses of Stream Processing
- *Designing Data-Intensive Applications* — Martin Kleppmann — Chapter 11. Stream Processing
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Streaming
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Materialized Views, Federation, and Query Virtualization › Streaming Transformations and Processing
- *Designing Data-Intensive Applications* — Martin Kleppmann — Processing Streams › Stream analytics
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Serving Data for Analytics, Machine Learning, and Reverse ETL › Streaming Systems
- *Designing Data-Intensive Applications* — Martin Kleppmann — Processing Streams
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Queries on Streaming Data
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Lambda Architecture
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Future of Data Engineering › Streaming Pipelines and Real-Time Analytical Databases

