---
type: "practice"
difficulty: "Deep"
---

# Explain the difference between token bucket and sliding window for rate limiting.

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Idempotent Consumer Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Idempotent_Consumer_Design/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Compare two rate-limiting algorithms — token bucket and sliding window — and explain how they differ in what they permit, how much state they cost, and when to choose each. Note "sliding window" is itself two variants (sliding-window log and sliding-window counter); a strong answer disambiguates them. This is a "deep understanding" LLD question, so I should explain the mechanics, the burst behavior, memory/accuracy trade-offs, and give concrete numeric examples where they diverge.

What the interviewer is probing for: Do I understand that token bucket controls an average rate while permitting bursts up to a capacity, whereas sliding window enforces "at most N requests in any trailing T seconds" — a genuine rolling limit? Can I reason about the fixed-window boundary-burst problem that sliding window fixes, and the memory/accuracy trade-off between the log and counter variants? And can I pick the right one for a scenario?

Clarifying questions: Do we need to allow bursts (token bucket) or cap them strictly (sliding window)? How much per-client memory can we afford? Is approximate accuracy acceptable? Single node or distributed?

Plan: Define each algorithm precisely, then contrast on five axes — burst handling, what "the limit" actually means, memory, accuracy, and smoothness — with a worked numeric scenario that shows them behaving differently, then give a decision guide.

Token bucket — mechanics

A bucket holds up to capacity tokens and is refilled at refillRate tokens/second (up to the cap). Each request removes one token; if the bucket is empty, the request is rejected. Two parameters — capacity (the max burst) and refillRate (the sustained average rate) — are decoupled. State per client is O(1): just (tokens, lastRefillTimestamp), with refill computed lazily on access.

Key property: it allows a burst of up to capacity requests instantly, then throttles to refillRate. So "10 requests/second, capacity 50" lets a client fire 50 at once, then 10/s thereafter. It regulates the long-run average while tolerating short spikes — which matches real API traffic (clients often batch).

Sliding window — mechanics (two variants)

The limit is phrased as "at most N requests in any trailing window of T seconds," and the window slides continuously with the current time (unlike a fixed window that resets on clock boundaries).

Sliding window log: store the timestamp of every request (e.g. in a sorted set). To decide, drop timestamps older than now - T and count the rest; allow if count < N. Exact and boundary-safe, but memory is O(N) per client — you keep up to N timestamps. Expensive at scale.
Sliding window counter: keep just two counters — the current fixed window's count and the previous window's count — and estimate the rolling count by weighting the previous window by how much of it still overlaps the sliding window: estimate = currentCount + previousCount * overlapFraction. O(1) memory, smooths the fixed-window boundary spike, but is a slight approximation (it assumes requests were uniformly spread in the previous window).
The core difference — bursts vs. a true rolling cap

This is the heart of the answer. Token bucket regulates a rate and permits bursts up to its capacity — it does not care how requests were distributed in the last T seconds, only whether tokens are available now. Sliding window enforces a hard rolling count — "no more than N in any T-second window" — so it does not let a client suddenly do N-then-N; it always looks back over the real trailing window.

Concretely, with token bucket you choose to allow bursts by setting capacity > refillRate. With sliding window there is no separate burst knob — the burst you allow is exactly N spread anywhere, but never 2N across a boundary (which is what a fixed window wrongly permits). So sliding window's headline value is fixing the fixed-window boundary problem while keeping the semantics "N per T."

The fixed-window boundary problem (what sliding window fixes)

A naive fixed-window limiter of "5 per minute" resets the counter at each minute boundary. A client can send 5 requests at 12:00:59 and 5 more at 12:01:00 → 10 requests in ~1 second, double the intended rate, because two windows abut. Sliding window (log or counter) eliminates this because the window is anchored to now, not the clock; the 12:00:59 requests are still counted at 12:01:00. Token bucket also avoids this — but by a different mechanism (tokens simply run out).

Memory and accuracy trade-offs
Token bucket: O(1) memory, exact for its own semantics, minimal state — the cheapest robust option.
Sliding window log: O(N) memory (N timestamps/client), perfectly accurate. Use when precision is contractual (billing, strict quotas) and N is small.
Sliding window counter: O(1) memory, approximate (uniform-distribution assumption in the previous window causes small over/under-count near boundaries), but far cheaper — the usual production default when you want the "N per T" semantics without O(N) memory.
Smoothness and downstream protection

Token bucket lets bursts through, so a downstream service still sees spikes (up to capacity at once). If the goal is to smooth load to a strict constant rate, leaky bucket (a cousin) is better than either. Sliding window caps the count but, like token bucket, doesn't force even spacing within the window. So neither guarantees perfectly smooth output — that's the leaky-bucket job.

Decision guide
Choose token bucket when you want to allow legitimate bursts (batch clients, retries, API SDKs), need O(1) memory, and think in "average rate + max burst." It's the industry default (AWS, Stripe).
Choose sliding window log when accuracy must be exact and N is small (strict per-second quotas, security throttles).
Choose sliding window counter when you want "N per T" semantics with O(1) memory and can tolerate a tiny approximation — a great gateway default that fixes the fixed-window spike cheaply.
Summary — what a strong answer emphasizes

Token bucket regulates an average rate and deliberately permits bursts up to a capacity with O(1) state; sliding window enforces a true rolling "N per T" cap and exists to fix the fixed-window boundary 2× spike. Sliding-window log is exact but O(N) memory; sliding-window counter is O(1) but slightly approximate. Pick token bucket for burst-tolerant APIs and cheap state, sliding-window log for exactness at small N, sliding-window counter for cheap "N per T."

#### Common mistakes
conflating "sliding window" with "fixed window" (missing the boundary point entirely); claiming token bucket forbids bursts (it's designed to allow them); forgetting that sliding-window log is O(N) memory; presenting sliding-window counter as exact (it's approximate).

#### Follow-up
probes: "Distributed version?" (token bucket → atomic Redis Lua on (tokens, ts); sliding-window counter → two Redis counters; log → Redis sorted set with ZREMRANGEBYSCORE). "Which is cheaper per request?" (token bucket / SW-counter: O(1); log: O(log N) to trim). "How to allow bursts and be boundary-safe?" (token bucket already is both).

**Comparison:**

```
Dimension	Token bucket	Sliding window log	Sliding window counter
What it limits	Avg rate + burst cap	Exact N in trailing T	~N in trailing T
Allows bursts	Yes, up to capacity	No (hard rolling cap)	No (hard rolling cap)
Fixed-window boundary spike	Avoided (tokens deplete)	Avoided (exact)	Avoided (smoothed)
Memory / client	O(1) (tokens,ts)	O(N) timestamps	O(1) two counters
Accuracy	Exact for its semantics	Perfectly exact	Approximate (uniform assumption)
Cost / request	O(1)	O(log N)/O(1) trim	O(1)
Best for	Burst-tolerant APIs, cheap state	Strict/small-N quotas, billing	"N per T" cheap default
Question	Token bucket	Sliding window
Can a client send 2N in ~1s across a boundary?	No	No
Can a client send a big legit burst?	Yes (by design)	No
Separate burst vs rate knob?	Yes (capacity vs refillRate)	No (single N/T)
```

**Code:**

```java
// --- Token bucket: average rate + burst, O(1) state ---
class TokenBucket {
    double tokens; long lastNanos;
    final long capacity; final double ratePerSec;   // burst=capacity, avg=ratePerSec
    TokenBucket(long cap, double rate){ capacity=cap; ratePerSec=rate; tokens=cap; lastNanos=System.nanoTime(); }
    synchronized boolean allow() {
        long now = System.nanoTime();
        tokens = Math.min(capacity, tokens + (now-lastNanos)/1e9 * ratePerSec); // lazy refill
        lastNanos = now;
        if (tokens >= 1) { tokens -= 1; return true; }
        return false;
    }
}

// --- Sliding window LOG: exact, O(N) memory ---
class SlidingWindowLog {
    final Deque<Long> stamps = new ArrayDeque<>();
    final int limit; final long windowMs;
    SlidingWindowLog(int limit, long windowMs){ this.limit=limit; this.windowMs=windowMs; }
    synchronized boolean allow() {
        long now = System.currentTimeMillis();
        while (!stamps.isEmpty() && stamps.peekFirst() <= now - windowMs) stamps.pollFirst(); // trim old
        if (stamps.size() < limit) { stamps.addLast(now); return true; }
        return false;
    }
}

// --- Sliding window COUNTER: approximate, O(1) memory ---
class SlidingWindowCounter {
    long curWindowStart; long curCount, prevCount;
    final int limit; final long windowMs;
    SlidingWindowCounter(int limit, long windowMs){ this.limit=limit; this.windowMs=windowMs; }
    synchronized boolean allow() {
        long now = System.currentTimeMillis();
        long start = now - (now % windowMs);
        if (start != curWindowStart) {                       // rolled into a new fixed window
            if (start - windowMs == curWindowStart) prevCount = curCount; else prevCount = 0;
            curWindowStart = start; curCount = 0;
        }
        double overlap = (windowMs - (now - curWindowStart)) / (double) windowMs; // 1->0 across window
        double estimate = prevCount * overlap + curCount;    // weighted rolling estimate
        if (estimate < limit) { curCount++; return true; }
        return false;
    }
}
```

**Example input / output:**

```text
Scenario A — bursts. Config: "10 req/sec". TokenBucket cap=50,rate=10 vs SlidingWindow N=10,T=1s.

Client fires 40 requests instantly at t=0:
  Token bucket : first 40 -> ALLOW (bucket had 50 tokens) -> burst permitted
  Sliding window(N=10,T=1s): first 10 -> ALLOW, next 30 -> REJECT (only 10 per rolling second)
  => same "10/sec" intent, opposite burst behavior.

Scenario B — fixed-window boundary spike (why sliding window exists). Limit 5 per minute:
  Fixed window : 12:00:59 send 5 ALLOW ; 12:01:00 send 5 ALLOW  => 10 in ~1s (BUG, 2x)
  Sliding window log: at 12:01:00 the 5 stamps from 12:00:59 are within trailing 60s
                      -> count=5 already -> next 5 REJECTED  (no 2x)
  Token bucket(cap5,rate 5/60s): 5 tokens spent at 12:00:59 -> empty ->
                      12:01:00 refilled ~0.08 tokens -> next requests REJECT (no 2x)

Scenario C — memory:
  Sliding window log, N=1000 -> stores up to 1000 timestamps/client  (O(N))
  Token bucket / SW-counter    -> stores (tokens,ts) / (two counters) (O(1))
```

**Flow:**

1. yes
2. no, strict rolling N/T
3. exact, small N
4. cheap, approx ok
5. Need to allow bursts?
6. Token bucket: capacity=burst,refillRate=avg, O(1)
7. Accuracy vs memory?
8. Sliding window LOG: O(N)timestamps
9. Sliding window COUNTER: twocounters, O(1)

