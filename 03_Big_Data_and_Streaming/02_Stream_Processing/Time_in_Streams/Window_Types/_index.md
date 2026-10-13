---
id: "03_Big_Data_and_Streaming/02_Stream_Processing/Time_in_Streams/Window_Types"
level: 5
type: "curriculum"
curriculum: "data_engineering"
---

# Window Types

**L5 Expert** · [↑ Reasoning About Time in Streams](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A window turns an endless stream into finite buckets you can aggregate; the four common shapes are tumbling, hopping, sliding and session windows, and the choice decides what question the number answers.

### 1. The big picture

You cannot compute a sum or average over an infinite stream — you need a boundary. Windows provide it: small batches of events, defined by time (or count), over which an aggregate is computed and emitted. In stream engines windows are generated dynamically as data arrives, and each is closed by a trigger (usually the watermark passing its end).

The choice of window type is a modelling decision: "orders per minute" is a different question from "orders in the last hour, updated every minute" or "orders per user visit".

### 2. Core concepts and vocabulary

- **Tumbling (fixed) window** — fixed-size, non-overlapping, gap-free; each event belongs to exactly one window (e.g., every minute).
- **Hopping window** — fixed size, but advances by a smaller *hop*, so windows overlap (5-minute window every 1 minute); each event belongs to size/hop windows.
- **Sliding window** — contains all events within some interval of each other (e.g., the last 5 minutes relative to each event); conceptually continuous.
- **Session window** — groups events of one key that occur close together, closed after an inactivity gap (e.g., 5 minutes); no fixed length.
- **Window size / hop (slide) / gap** — the parameters of the above.
- **Key** — windows are per key: each user gets their own sessions.
- **Trigger / watermark** — decides when a window result is emitted.
- **Count window** — window defined by number of events instead of time.

### 3. How it works, step by step

For each type, an event with timestamp *t* is assigned as follows:

1. **Tumbling (size 5 min):** window start = `floor(t / 5min) × 5min`; the event goes to `[start, start+5min)`. E.g., 10:03 → `[10:00,10:05)`.
2. **Hopping (size 5 min, hop 1 min):** the event is added to every window whose range covers *t*: 5 windows. 10:03 → `[09:59,10:04)`, `[10:00,10:05)`, … `[10:03,10:08)`.
3. **Sliding (5 min):** keep a buffer of events; on each new event, aggregate those with timestamps within 5 minutes; expire older ones.
4. **Session (gap 5 min):** start a session at the first event for a key; each new event extends the session end to `t + gap`; if `t` is beyond the current end, close the session and start a new one. Sessions may also *merge* when a late event bridges two sessions.

```text
time → 0  1  2  3  4  5  6  7  8  9  10
events:  x  x     x        x  x
tumbling(5): [0-5)     [5-10)  [10-15)
hopping(5,1): [0-5)[1-6)[2-7)[3-8)[4-9) ...
session(gap 3): [0 .. 3+3)     [7 .. 8+3)
```

### 4. Worked example

```sql
-- Flink SQL
-- Tumbling: orders per minute
SELECT window_start, COUNT(*) FROM TABLE(
  TUMBLE(TABLE orders, DESCRIPTOR(order_time), INTERVAL '1' MINUTE))
GROUP BY window_start, window_end;

-- Hopping: 5-min moving count refreshed every minute
SELECT window_start, COUNT(*) FROM TABLE(
  HOP(TABLE orders, DESCRIPTOR(order_time), INTERVAL '1' MINUTE, INTERVAL '5' MINUTE))
GROUP BY window_start, window_end;

-- Session: user visits, gap of 5 minutes
SELECT user_id, window_start, window_end, COUNT(*) AS page_views FROM TABLE(
  SESSION(TABLE clicks PARTITION BY user_id, DESCRIPTOR(click_time), INTERVAL '5' MINUTE))
GROUP BY user_id, window_start, window_end;
```

Reading the results:

- Tumbling gives one non-overlapping row per minute — good for dashboards and billing buckets.
- Hopping gives a smoothed series; 5× more output rows and state than tumbling.
- Session gives one row per visit; durations differ between users, and a late click may extend or merge sessions, so results can be revised.

### 5. Under the hood

- **State per window.** Tumbling keeps one accumulator per key per window; hopping multiplies by size/hop; sliding may keep every event or use incremental structures (subtract expired values, deque for min/max).
- **Pre-aggregation.** Windowed aggregates should be incremental (sum, count, min, max, or sketches), not buffer all events, to save memory.
- **Session merging.** Session windows are not known in advance; because windows are per key and can merge, the engine tracks tentative sessions and merges them when a bridging event arrives (possible only when lateness is allowed).
- **Window alignment.** Tumbling/hopping boundaries align to the epoch (or a configurable offset/time zone); daily windows need time-zone-aware offsets, and daylight saving shifts break fixed 24h windows.
- **Triggers.** Beyond "watermark passes end", engines can fire early (speculative partial results every N seconds) and late (on each straggler within allowed lateness), controlled via accumulation modes (discarding vs accumulating, retraction).
- **Batch-world analogue.** Session windows resemble "sessionizing" logs in batch: sort by user and time, split at gaps. Streams do it incrementally.
- **Count windows** ignore time and are simpler but not robust to bursts and idleness.

### 6. Performance and design implications

- Hop much smaller than size is expensive: with size 1 hour and hop 1 second each event updates 3,600 windows. Prefer a tumbling aggregation at the hop granularity followed by a combine over several buckets.
- Session state is per key and open for gap + lateness; many idle keys keep state until they expire.
- Window size drives latency (a result appears only when closed) and state; choose the smallest that answers the question.
- Skewed keys (one bot user with a never-ending session) create huge sessions — cap session length.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `window(col, "5 minutes")` (Spark) | Tumbling window | Periodic aggregates |
| `window(col, "10 minutes", "5 minutes")` | Hopping/sliding in Spark | Smoothed metrics |
| `session_window(col, "5 minutes")` (Spark 3.2+) | Session window | User visits |
| Flink `TUMBLE / HOP / SESSION` table functions | Windowing TVFs | Flink SQL |
| `allowedLateness` / `withWatermark` | Lateness bounds | Tuning completeness |
| `trigger(processingTime="10 seconds")` | How often micro-batches fire | Latency tuning |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| **Tumbling** for reports, billing, simplest cheapest state | You need smooth, continuously updated trends |
| **Hopping/sliding** for moving averages, alerting thresholds | Hop ≪ size (state blowup) |
| **Session** for user behaviour and visit analysis | Keys are unbounded/bot-like or events arrive very late |
| **Count windows** for fixed-size batches of events | Rate varies or sources can be idle |

### 9. Common mistakes

- **Using hopping where tumbling suffices** → multiplied state and duplicate counting of events → use tumbling.
- **Treating overlapping windows as disjoint** → summing hopping results double-counts → only sum tumbling windows.
- **Ignoring time zones** → "daily" windows split at UTC midnight → configure offsets.
- **Session gap too small/large** → sessions fragment or merge unrelated visits → pick from observed inter-event gaps.
- **No cap on session length** → one key holds state forever → add max duration.

### 10. 🎯 Interview kit

**30-second answer:** Windows slice a stream into finite buckets. Tumbling windows are fixed and non-overlapping, hopping windows overlap by advancing in smaller steps, sliding windows cover the interval around each event, and session windows are per-key and close after an inactivity gap.

**Structure a longer answer:** why windows are needed → the four types with an example question each → how events are assigned and when results are emitted → state/latency costs → pitfalls.

**Likely follow-ups:**
- *Tumbling vs hopping?* → Overlap and cost.
- *How do session windows work with late data?* → Sessions can extend or merge.
- *How would you compute "last 24h unique users updated every minute"?* → Hopping with sketch/pre-aggregated minute buckets.
- *How do windows interact with watermarks?* → The watermark closes windows and purges state.

**Signals interviewers look for:** picks the window type from the business question, aware of state cost and lateness.

### 11. Check yourself

- [ ] For an event at 10:03, list the windows it belongs to under tumbling (5 min) and hopping (5 min, hop 1 min).
- [ ] Explain how two session windows can merge.
- [ ] Why can't you sum hopping window counts to get a daily total?
- [ ] Choose a window type for "average speed over the last 30 seconds, every 5 seconds".
<!-- explainer:end -->

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Queries on Streaming Data › Fixed-time windows
- *Designing Data-Intensive Applications* — Martin Kleppmann — Reasoning About Time › Types of windows
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Queries on Streaming Data › Windows, triggers, emitted statistics, and late-arriving data
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and DataFrames Interacting with External Data Sources › Windowing

