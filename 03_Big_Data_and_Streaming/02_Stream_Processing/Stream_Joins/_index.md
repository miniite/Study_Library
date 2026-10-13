---
id: "03_Big_Data_and_Streaming/02_Stream_Processing/Stream_Joins"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Stream Joins

**L4 Advanced** · [↑ Stream Processing](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A stream join combines events with other events (within a time window), with a table (enrichment via a local copy kept up to date by CDC), or table with table (materialized view maintenance) — and the result depends on the order of events, so time-dependence must be handled deliberately.

### 1. The big picture

In batch processing a join links records on a key across datasets, with everything available at once. In streaming, one or both inputs are unbounded and new events keep arriving, so the join must be **incremental**: when a new event arrives, find the matching data seen so far and emit a result — and maybe later when the other side shows up.

There are three types, distinguished by what the two inputs are:

| Type | Inputs | Typical use |
|---|---|---|
| Stream–stream (window join) | two event streams | link a search with the click that followed |
| Stream–table (enrichment) | event stream + changelog-backed table | add user profile data to each click |
| Table–table (materialized view) | two changelog streams | maintain a "timeline" or cache view |

### 2. Core concepts and vocabulary

- **Window join** — match events from two streams that share a key and fall within a time window of each other.
- **Join state** — events buffered by key until a match can occur or the window expires.
- **Stream enrichment** — adding fields from a reference table to each event.
- **Changelog** — stream of inserts/updates/deletes of a table (CDC), used to keep a local copy current.
- **Materialized view** — a precomputed query result updated incrementally.
- **Slowly changing dimension (SCD)** — a dimension whose attributes change over time; joins must pick the right version.
- **Watermark** — an event-time bound after which no more late events are expected; lets the engine clear join state.
- **Outer join (streaming)** — emits a null-padded row once the window closes without a match.
- **Broadcast / co-partitioned join** — local joins made possible by replicating a small table or partitioning both inputs by the same key.

### 3. How it works, step by step

**Stream–stream (window) join**

1. Both streams are partitioned by the join key so matching events meet in the same task.
2. The processor keeps a state store of recent events of each stream (e.g., last 1 hour).
3. When an event arrives from stream A, look up buffered B events with the same key within the time bound; emit one output per match.
4. Store the A event in state so future B events can match it.
5. As the watermark advances, state older than the window is dropped.
6. For an outer join, an unmatched event is emitted with nulls only after its window has closed (so "no match" is known).

Example: a click joined to a search within 1 hour — if a click has no search match in the window, treat it as a search-less click; for ad clicks vs impressions, a missing click means "the ad was not clicked".

**Stream–table join (enrichment)**

1. Subscribe to the table's changelog (CDC) and keep a local copy, indexed by key, in the processor (in memory or an embedded store).
2. For each stream event, look up the key in the local copy and emit the enriched event.
3. Updates to the table flow through the changelog and update the copy, so no remote database query per event is needed.

**Table–table join**

1. Both inputs are changelogs of tables.
2. The processor keeps both tables materialized and indexed.
3. When a row changes on either side, recompute the affected join result and emit the change to the output.
4. The output is itself a changelog of a materialized view (e.g., a timeline = join of tweets and follow relationships, with aggregation).

### 4. Worked example

Enrich page-view events with the user's plan, and count clicks per ad within 10 minutes of the impression, in Spark Structured Streaming.

```python
impressions = (spark.readStream.format("kafka")...
               .withWatermark("impression_time", "10 minutes"))
clicks      = (spark.readStream.format("kafka")...
               .withWatermark("click_time", "20 minutes"))

joined = impressions.join(
    clicks,
    expr("""
        impression_ad_id = click_ad_id AND
        click_time >= impression_time AND
        click_time <= impression_time + interval 1 hour
    """),
    "leftOuter")          # keep impressions with no click
```

Why each piece matters:

1. The **time-range condition** bounds how long an impression must be remembered; without it the state grows forever.
2. The **watermarks** tell the engine when no more clicks can arrive for old impressions, so state can be purged and `leftOuter` rows can be emitted.
3. The result for an unclicked impression appears only after the window plus watermark delay — joins trade latency for completeness.

For enrichment, a stream–table join reads `users_changelog`, keeps `{user_id → plan}` locally and annotates each `page_view` as it passes.

### 5. Under the hood

- **State is the cost.** A window join keeps all events of the window for both sides. Memory scales with rate × window length; this is why windows must be bounded and state stores are disk-backed.
- **Time-dependence.** If the table changes over time, the result depends on *when* the join is evaluated. Event order across streams (and across partitions) is not deterministic, so replaying can give a different answer. Example: a sale event joined to a tax rate that changed; the rate to use is the rate valid *at the time of the sale*. Joins whose output depends on interleaving of independent streams are nondeterministic and hard to reproduce.
- **Slowly changing dimensions.** The usual fix is to give each version of the joined record a unique identifier (a version id), so a join references a specific version and stays deterministic, at the cost of retaining all versions (no log compaction).
- **Local vs remote lookup.** Querying a remote database per event adds network latency and load and limits throughput; the CDC-fed local copy is much faster and keeps the processor and the database decoupled.
- **Co-partitioning.** Local joins require both streams to be partitioned by the join key with the same partitioner and partition count; otherwise a repartition (shuffle) is needed.
- **Late data.** Events later than the watermark are dropped or sent to a side output; for correctness-critical joins allow lateness or run a periodic batch correction.
- **Consistency.** The table copy lags the source; for a brief moment an event may be enriched with a stale value. Consider whether eventual consistency is acceptable.

### 6. Performance and design implications

- Bound state with join windows, watermarks and TTLs; monitor state size.
- Prefer broadcast of small tables and co-partitioning of large ones.
- Pre-aggregate before joining to reduce state.
- Choose window length from the real-world delay between related events (e.g., 95th percentile).
- Use compact, columnar or RocksDB-backed state for large tables.
- Test with replayed and out-of-order data to expose nondeterminism.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `df.withWatermark(col, delay)` (Spark) | Bounds lateness and state | Any stream–stream join/aggregation |
| `join(..., expr(time range))` | Window condition between streams | Stream–stream joins |
| `joinType = "leftOuter"` | Keep unmatched rows after watermark | Impression/click, orders without payment |
| Kafka Streams `KStream.join(KTable)` | Enrichment from a changelog table | Reference-data lookups |
| `GlobalKTable` / broadcast state | Full table on every task | Small dimension tables |
| State TTL (Flink) | Expire join state | Prevent unbounded growth |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| **Stream–stream window join** for events related within known delay | The relation spans days — use batch or a stateful store |
| **Local table via CDC** for fast enrichment | Table is huge and cannot fit in task state |
| **Remote lookup** for simple, low-rate enrichment | Throughput is high or the DB must be protected |
| **Outer joins** when "absence" matters | Latency requirement is lower than the window |
| **Versioned dimension keys** for deterministic history | Storage cost of keeping all versions is prohibitive |

### 9. Common mistakes

- **No time bound on a stream–stream join** → state grows without limit → add a time-range predicate and watermark.
- **Per-event database lookup** → latency and overload of the source DB → local changelog copy.
- **Joining on processing time** → results differ per run → use event time.
- **Mismatched partitioning** → events for the same key land on different tasks → same key and partition count, or repartition.
- **Ignoring dimension changes** → wrong historic attribution → version the dimension.
- **Forgetting late events** → silently dropped matches → monitor drop counters and allow lateness.

### 10. 🎯 Interview kit

**30-second answer:** There are three streaming joins: stream–stream (match events within a time window, buffering state), stream–table (enrich events using a local copy of a table kept current from a changelog) and table–table (maintain a materialized view). They are incremental, require partitioning by key, and are sensitive to time and ordering.

**Structure a longer answer:** state the three types → mechanism for each → state and watermark management → time-dependence/SCD → performance → scenario.

**Likely follow-ups:**
- *Why not query the database for each event?* → Latency, load, coupling; use CDC-fed local copy.
- *How do you keep join state bounded?* → Time-range predicates, watermarks, TTL.
- *What is the nondeterminism issue?* → Interleaving of independent streams and changing dimensions; fix with versioning.
- *How does an outer join work in streaming?* → Emit null-padded after the window closes.

**Signals interviewers look for:** knows state is the cost, uses event time, understands CDC-based enrichment and SCD handling.

### 11. Check yourself

- [ ] Sketch a window join between clicks and impressions with 1-hour bound.
- [ ] Explain how a table is kept up to date inside a stream processor.
- [ ] Describe a case where re-running the join gives a different answer, and how to fix it.
- [ ] Explain why a watermark is needed for an outer stream join.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Stream Joins › Time-dependence of joins
- *Designing Data-Intensive Applications* — Martin Kleppmann — Stream Joins › Stream-table join (stream enrichment)
- *Designing Data-Intensive Applications* — Martin Kleppmann — Stream Joins
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Streaming Joins › Stream--Stream Joins
- *Designing Data-Intensive Applications* — Martin Kleppmann — Stream Joins › Table-table join (materialized view maintenance)
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Queries on Streaming Data › Conventional table joins
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and DataFrames Interacting with External Data Sources › Windowing

