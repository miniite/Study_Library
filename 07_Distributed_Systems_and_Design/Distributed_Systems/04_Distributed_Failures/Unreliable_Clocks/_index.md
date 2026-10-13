---
id: "07_Distributed_Systems_and_Design/Distributed_Systems/04_Distributed_Failures/Unreliable_Clocks"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Unreliable Clocks

**L4 Advanced** · [↑ Faults and Failure in Distributed Systems](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Every machine has its own imperfect clock, so wall-clock timestamps from different nodes can't be trusted for ordering or correctness — use monotonic clocks for durations, treat time-of-day as an interval with error, and never assume a process notices it was paused.

### 1. The big picture
Applications ask clocks two different kinds of question: *durations* (has this request timed out? what's p99 latency? how long did the user stay?) and *points in time* (when was this published? when does this cache entry expire? what timestamp goes in the log?). A distributed system muddies both: messages take a variable time to travel, and every node has its own quartz-crystal hardware clock that drifts. Clock synchronization (NTP) narrows the gap but never closes it, and a wrong clock usually causes silent data loss rather than a loud crash — which is why it's dangerous.

This lesson sits next to Network Faults and Timeouts (variable delay) and Fencing (surviving pauses), and is the basis for later topics on ordering and Spanner-style transactions.

### 2. Core concepts and vocabulary
- **Time-of-day (wall-clock) clock** — current date/time since the epoch (`CLOCK_REALTIME`, `System.currentTimeMillis()`); can jump when corrected.
- **Monotonic clock** — only moves forward; meaningful only as a *difference* on one machine (`CLOCK_MONOTONIC`, `System.nanoTime()`).
- **Epoch** — reference point, midnight UTC 1 Jan 1970 for POSIX time.
- **NTP** — Network Time Protocol: adjusts a machine's clock from a hierarchy of time servers (ultimately GPS/atomic).
- **Drift** — a clock running faster or slower than true time (temperature dependent).
- **Skew / offset** — the difference between two clocks at an instant.
- **Slewing** — NTP gradually speeding up/slowing a clock (up to ±0.05%) rather than jumping it.
- **Stepping** — NTP abruptly setting the clock, which may move time backward.
- **Leap second** — a minute of 59 or 61 seconds; breaks naive assumptions. **Smearing** spreads the adjustment over a day.
- **Confidence interval** — a clock reading is really a range `[earliest, latest]`.
- **TrueTime** — Google Spanner's API that returns that interval explicitly.
- **Process pause** — a thread stops (GC, VM suspend, I/O stall) for an unbounded time without noticing.
- **Lease** — a lock with an expiry time.

### 3. How it works, step by step
**Which clock for which job**
| Question | Clock | Why |
|---|---|---|
| Timeouts, elapsed time, latency measurement | Monotonic | Never jumps backward; no cross-node sync needed |
| Event timestamps for display / logs | Time-of-day | Human meaning; accuracy limited |
| Ordering events *across* machines | Neither, safely | Skew exceeds event gaps; use logical clocks / consensus |

**How good is synchronization? (typical sources of error)**
1. Quartz drift — Google assumes ~200 ppm: ≈6 ms per 30 s between syncs, ≈17 s if synced once a day.
2. Large offsets make NTP refuse to sync or force a step reset, so applications see time jump.
3. A node silently firewalled off from NTP just drifts unnoticed.
4. NTP precision is bounded by network delay: ~35 ms best case over the internet, spikes near a second.
5. Some NTP servers are wrong by hours; clients query several and discard outliers.
6. Leap seconds have crashed large systems; smearing is the pragmatic fix.
7. In VMs the hardware clock is virtualized and can stall or jump when the guest is paused.

**The lease-with-pause bug**
1. A leader holds a lease with an expiry time set by another machine.
2. Each loop it checks `expiry − now < 10 s` and renews if needed, then processes a request.
3. Flaw 1: comparing a remote timestamp to the local wall clock requires synchronized clocks.
4. Flaw 2: even with monotonic time, a 15-second pause *between the check and the work* makes the lease expire while the thread still believes it is valid; a new leader exists and both act.
5. Fix: the resource validates fencing tokens (see Fencing); don't rely on elapsed-time checks made by the paused actor.

**Reasons a thread can pause for a long time**: stop-the-world GC; VM suspend/migration; laptop lid close; OS context switch under heavy load; swapping/paging (or thrashing); synchronous disk I/O such as network-mounted storage; SIGSTOP (Ctrl-Z).

### 4. Worked example
Last-write-wins (LWW) with wall-clock timestamps loses data:

```text
Node A clock = 12:00:00.100 (ahead by 3 ms)
Node B clock = 12:00:00.097
t=real 0 ms : client 1 writes x=1 via A   -> ts 12:00:00.100
t=real 2 ms : client 2 reads x=1, writes x=2 via B -> ts 12:00:00.099
Replication merge (LWW): .100 > .099  => x=1 wins
```

Client 2's later write was silently discarded. No error is raised. The same pattern appears in Cassandra (timestamps from client or coordinator) and in time-based cache expiry or idempotency windows.

Safe duration measurement:

```java
long start = System.nanoTime();          // monotonic
doWork();
long elapsedNanos = System.nanoTime() - start;   // valid; never negative
```

Compare to the broken version using `System.currentTimeMillis()`: an NTP step during `doWork()` can make `elapsed` negative or huge.

### 5. Under the hood
- **A reading is an interval.** With a LAN NTP server you may get a few ms of uncertainty; with a public server tens of ms, spiking past 100 ms under congestion. Microsecond digits in a timestamp are mostly noise unless the source is a directly attached GPS/atomic reference whose error is documented.
- **Uncertainty is rarely exposed.** `clock_gettime()` doesn't return its error bound; TrueTime does, returning `[earliest, latest]` whose width depends on the time since last sync and drift assumptions. Spanner then *waits out* the uncertainty (commit wait) so that transaction timestamps are safely ordered: if two intervals don't overlap, the order is certain.
- **Ordering with clocks only works when intervals don't overlap.** Mainstream databases outside Google generally don't use synchronized clocks for global snapshots; they use logical timestamps/sequence numbers.
- **Monotonic clock caveats.** Per-CPU timers may differ; the OS compensates, but treat monotonicity as strong, not absolute. Values aren't comparable across machines.
- **Pauses and GC mitigation.** Treat GC like a planned outage (drain requests from a node about to collect), restart processes before full GCs accumulate, and use low-pause collectors. These reduce but cannot eliminate pauses; hard real-time guarantees require real-time OS support and are costly.

### 6. Performance and design implications
- Monitor clock offsets across all nodes; a node whose clock drifts beyond a threshold should be evicted, since silent skew is worse than a crash.
- Prefer logical clocks (Lamport, vector clocks, version vectors) or a single sequencer for ordering; reserve wall-clock for human-facing metadata.
- When aggregating events from many producers in data pipelines, normalize or carry *event time vs ingestion time* separately and allow lateness (watermarks) rather than trusting source timestamps.
- Schedule jobs and TTLs with slack for skew; don't encode correctness into "expires at exactly T".
- Run `chrony`/`ntpd` with several upstream sources and alert on offset and on loss of sync.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `chronyc tracking` / `chronyc sources` | Show offset, drift, source quality | Verifying sync health |
| `ntpq -p` | Peer offsets and jitter | Legacy ntpd checks |
| `timedatectl status` | Whether system clock is NTP-synchronized | Quick sanity check |
| `System.nanoTime()` / `time.monotonic()` / `CLOCK_MONOTONIC` | Monotonic elapsed time | Timeouts, latency |
| `SYSUTCDATETIME()` (SQL Server) | UTC time-of-day, not monotonic | Row timestamps — never for ordering across servers |
| `makestep`, leap smear options in `chrony.conf` | Control stepping and leap handling | Avoid backward jumps |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Monotonic clock for timeouts and durations | Needing a value comparable between machines |
| Wall-clock for logs, display, coarse TTLs | Using it to order concurrent writes or resolve conflicts |
| LWW with timestamps — acceptable loss, simple data (caches, metrics) | Important updates where silently dropped writes matter |
| TrueTime-style bounded clocks | You lack GPS/atomic infrastructure and tight bounds |
| Logical clocks / sequence numbers | You need human-meaningful real time |

### 9. Common mistakes
- Using `currentTimeMillis` to measure elapsed time → negative/huge durations after NTP step → monotonic clock.
- Ordering distributed events by local timestamps → lost updates under LWW → logical versions or consensus.
- Assuming a day has 86,400 s and every minute 60 s → leap-second failures → use UTC libraries and smearing.
- Trusting the lease check right before an action → pause invalidates it → fencing tokens.
- Never monitoring NTP → one VM drifts for months → add offset alerts.
- Treating sub-millisecond resolution as accuracy → false confidence → think in intervals.

### 10. 🎯 Interview kit
**30-second answer:** Each machine's clock drifts and NTP only bounds the error, so timestamps from different nodes can't reliably order events; wall-clocks can even jump backward. Use monotonic clocks for durations, logical clocks or consensus for ordering, and guard against process pauses with fencing rather than elapsed-time checks.
**Structure a longer answer:** two clock types and their uses → sources of error (drift, NTP limits, leap seconds, VMs) → concrete failure (LWW lost update, lease + pause) → mitigations (monitoring, logical clocks, TrueTime intervals, fencing).
**Likely follow-ups:**
- *Why not use timestamps for last-write-wins?* → skew can make a later write look older; silent loss.
- *How does Spanner get around it?* → TrueTime exposes an uncertainty interval and commit-wait ensures ordering.
- *Monotonic vs wall clock?* → duration vs date; jumps vs never backwards.
- *What about GC pauses?* → can't be prevented; fence stale holders.
**Signals interviewers look for:** you distinguish durations from points in time, quantify error (ms-to-s), and propose logical ordering rather than "just sync better".

### 11. Check yourself
- [ ] State when to use a monotonic clock versus a time-of-day clock.
- [ ] List five reasons a synchronized clock may be wrong.
- [ ] Produce a scenario where LWW with timestamps drops a newer write.
- [ ] Explain TrueTime's `[earliest, latest]` and how it enables ordering.
- [ ] Explain why a leader's lease check can still be unsafe after a pause.
<!-- explainer:end -->

## References

- *Designing Data-Intensive Applications* — Martin Kleppmann — Clock Synchronization and Accuracy
- *Designing Data-Intensive Applications* — Martin Kleppmann — Clock Synchronization and Accuracy › Clock readings have a confidence interval
- *Designing Data-Intensive Applications* — Martin Kleppmann — Synchronous Versus Asynchronous Networks › Unreliable Clocks
- *Designing Data-Intensive Applications* — Martin Kleppmann — Clock Synchronization and Accuracy › Synchronized clocks for global snapshots
- *Designing Data-Intensive Applications* — Martin Kleppmann — D
- *Designing Data-Intensive Applications* — Martin Kleppmann — Clock Synchronization and Accuracy › Relying on Synchronized Clocks
- *Designing Data-Intensive Applications* — Martin Kleppmann — Synchronous Versus Asynchronous Networks › Monotonic clocks
- *Designing Data-Intensive Applications* — Martin Kleppmann — Process Pauses
- *Database Internals* — Alex Petrov — Introduction and Overview › Clocks and Time
- *Designing Data-Intensive Applications* — Martin Kleppmann — Process Pauses › Limiting the impact of garbage collection

