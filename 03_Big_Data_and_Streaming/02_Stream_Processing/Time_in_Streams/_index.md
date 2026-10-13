---
id: "03_Big_Data_and_Streaming/02_Stream_Processing/Time_in_Streams"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Reasoning About Time in Streams

**L4 Advanced** · [↑ Stream Processing](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Stream processors must window events by when they *happened* (event time), not when they were processed, and use watermarks to decide when a window is complete enough to emit while still handling stragglers.

### 1. The big picture

Many stream computations are about time: "requests per minute", "average over the last five minutes", "sessions per user". In a batch job the time of an event is a field you group by. In a stream there is a trap: the obvious clock to use is the processing machine's clock, but events can be delayed by queues, network, mobile devices going offline, or the processor itself restarting and catching up.

Using processing time is simple and gives low latency but produces wrong answers whenever there are delays or replays. Using **event time** gives deterministic and replay-safe results but forces you to answer "when is a window finished?" — you can never be sure that all events for 10:03 have arrived. The **watermark** is the practical answer.

### 2. Core concepts and vocabulary

- **Event time** — timestamp assigned where the event occurred (device, server).
- **Processing time** — time on the machine that handles the event.
- **Ingestion time** — time the event entered the pipeline (broker).
- **Out-of-order events** — events arrive in an order different from their event times.
- **Late (straggler) event** — arrives after the window for its time has already been emitted.
- **Watermark** — a moving threshold: "no events with event time earlier than W are expected anymore".
- **Allowed lateness** — extra time a window stays open to accept late events and update results.
- **Window** — finite time bucket for aggregation; types are covered in *Window Types*.
- **Trigger** — rule that decides when a window's result is emitted (on watermark, every N events, early/late firings).
- **Clock skew** — device clocks differ from server clocks.
- **Event-time timeout** — stateful logic that fires when event time (watermark) passes a deadline.

### 3. How it works, step by step

1. **Stamp events.** Each event carries its event time (e.g., `click_time`).
2. **Assign to windows.** The processor places each event into window(s) by its event time — not the arrival time.
3. **Track progress.** The engine observes the maximum event time seen and computes `watermark = max_event_time − allowed_delay`.
4. **Hold windows open.** A window `[10:00, 10:05)` stays open until the watermark passes 10:05.
5. **Emit.** When the watermark passes the window end, the aggregate is emitted (and optionally state is kept for `allowed lateness`).
6. **Handle late events.** An event older than the watermark either (a) is dropped, (b) triggers a *correction* to a previously emitted result, or (c) is sent to a side output for reconciliation.
7. **Purge state.** Windows beyond watermark + lateness are removed to bound memory.

```text
event time →   10:00        10:05        10:10
                [--- window ---)
arrivals:  e(10:02) e(10:04) e(10:01) ... e(10:03)<-- late, arrives at wall-clock 10:12
watermark: advances behind the max event time, passes 10:05 at ~10:06 → window emitted
```

**Whose clock?** A mobile app may buffer events while offline and upload hours later; the device clock may be wrong. Three timestamps help: when the event occurred (device clock), when it was sent (device clock), and when it was received (server clock). Subtracting the second from the third gives the clock offset, which can be applied to the first to estimate true event time.

### 4. Worked example

Count orders per 5-minute event-time window, tolerating 10 minutes of lateness (Spark Structured Streaming):

```python
from pyspark.sql.functions import window, col

counts = (orders
    .withWatermark("order_time", "10 minutes")
    .groupBy(window(col("order_time"), "5 minutes"), col("country"))
    .count())

(counts.writeStream
       .outputMode("update")          # re-emit windows when late data arrives
       .format("console").start())
```

What happens:

1. An order with `order_time=10:03` arrives at 10:04 → counted in window 10:00–10:05.
2. Max event time seen is 10:19 → watermark = 10:09. Windows ending ≤ 10:09 (like 10:00–10:05) are finalized and their state is dropped.
3. An order stamped 10:03 arriving now (watermark 10:09) is **too late** and is ignored.
4. An order stamped 10:12 arriving late relative to max time 10:19 still lands in the open window 10:10–10:15, since it is newer than the watermark.

Trade-off visible: a 10-minute delay means results are final 10 minutes late, but almost all stragglers are included.

### 5. Under the hood

- **You never know you are done.** Completeness in unbounded data is a heuristic, since a mobile device might reconnect after a day. The watermark expresses a *policy*, not a fact; some systems emit an early approximate result and then publish corrections.
- **Watermark computation.** Typically `max(event_time) − delay` per source; with multiple partitions/sources the global watermark is the *minimum* across inputs, so one idle or slow partition stalls everything (hence idleness detection).
- **Processing time artifacts.** If a job is down for an hour and replays its backlog, processing time sees a spike of "events per minute" that never occurred. Event time shows the true shape.
- **Replay safety.** Event-time results are a function of the data only, so reprocessing historical data gives the same answer — the key to unifying batch and stream code (the Kappa idea).
- **Stateful timeouts.** In arbitrary stateful processing (e.g., `mapGroupsWithState`) a timeout can be driven by event time: it fires when the watermark exceeds a deadline, e.g., "close this user session if no event for 30 minutes of event time" — so the logic is not affected by processing delays.
- **Combining streams with different delays.** The join of two streams uses the slower watermark; unequal delays enlarge state.
- **Clock quality.** NTP skew on servers is small (ms); device clocks can be minutes or years off. Decide whether to trust the device clock or the ingestion time.

### 6. Performance and design implications

- Larger allowed delay → more complete results but later output and more state.
- Smaller delay → faster, possibly less complete results; combine with corrections or a batch backfill.
- Watermarks on skewed or sparse partitions need idleness handling.
- State per open window multiplies by the number of keys; high-cardinality keys with long lateness can exhaust memory.
- Monitor: watermark lag (wall-clock minus watermark), late-event drop counts, state size.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `withWatermark(col, "10 minutes")` | Defines lateness bound (Spark) | Windowed aggregations/joins |
| `outputMode("append" / "update" / "complete")` | When rows are emitted | Choosing final vs updated results |
| Flink `WatermarkStrategy.forBoundedOutOfOrderness` | Bounded out-of-order watermarks | Flink jobs |
| `allowedLateness(...)` | Keep window state for stragglers | Correct-late-data use cases |
| `withIdleness(...)` | Ignore idle partitions in watermark | Sparse sources |
| Side output / dead-letter for late events | Capture dropped events | Audit and reconciliation |
| `timeoutConf = EventTimeTimeout` | Event-time timeouts in stateful ops | Session/inactivity logic |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| **Event time** for correctness, replay and backfill | You only need rough, immediate metrics |
| **Processing time** for simple monitoring with no replay | Delays, restarts or mobile clients exist |
| **Short watermark delay** for fast alerts | Data frequently arrives late — results incomplete |
| **Long delay + allowed lateness** for accurate reports | Low-latency requirement |
| **Emit-then-correct** for dashboards | Downstream cannot handle retractions/updates |

### 9. Common mistakes

- **Windowing on processing time** → replays and outages distort counts → use event time.
- **No watermark on stateful operations** → state grows forever → set a watermark.
- **Trusting device clocks blindly** → future/past timestamps → clamp or correct with server receive time.
- **Idle partition stalls the watermark** → no windows close → idleness timeout.
- **Dropping late data silently** → unexplained undercounts → side output and metrics.
- **Equating watermark with certainty** → treating results as final → know the policy.

### 10. 🎯 Interview kit

**30-second answer:** Streams carry events whose timestamps (event time) differ from arrival time (processing time) because of delays and out-of-order delivery. We window by event time for correctness and use a watermark — "no events older than W expected" — to decide when to close windows, with allowed lateness for stragglers.

**Structure a longer answer:** event vs processing time → problem examples (outage replay, mobile offline) → windows and watermarks → handling late data (drop, update, side output) → state/latency trade-off.

**Likely follow-ups:**
- *What is a watermark and who computes it?* → Estimate of completeness, derived from max event time minus delay, minimum across inputs.
- *What happens to late events?* → Dropped, accepted within allowed lateness with updates, or diverted.
- *How do you handle unreliable device clocks?* → Use receive time offset correction or ingestion time.
- *Why does event time help backfills?* → Same results regardless of when processed.

**Signals interviewers look for:** states the completeness-vs-latency trade-off, mentions state cleanup, knows idle-partition watermark issue.

### 11. Check yourself

- [ ] Give an example where processing-time windows give a wrong answer.
- [ ] Compute the watermark given max event time 10:19 and 10-minute delay, and say which windows close.
- [ ] Describe three ways to handle a late event.
- [ ] Explain why one idle partition can block window emission.
<!-- explainer:end -->

## In this section (study in order)

1. [Window Types](<Window_Types/_index.md>) (L5 Expert)

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Reasoning About Time
- *Designing Data-Intensive Applications* — Martin Kleppmann — Reasoning About Time › Event time versus processing time
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Arbitrary Stateful Computations › Event-time timeouts
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Aggregations with Event-Time Windows › Handling late data with watermarks
- *Designing Data-Intensive Applications* — Martin Kleppmann — Reasoning About Time › Knowing when you're ready
- *Designing Data-Intensive Applications* — Martin Kleppmann — Reasoning About Time › Whose clock are you using, anyway?
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Queries on Streaming Data › Watermarks

