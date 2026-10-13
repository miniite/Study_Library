---
type: "practice"
difficulty: "Deep"
---

# You added a cache but the app still feels slow — how would you investigate?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Tail Latency After Expansion](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Tail_Latency_After_Expansion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: "You did the 'obvious' optimization — added a cache — and it didn't help (or barely helped). How do you figure out why?" The interviewer is probing whether I treat "add a cache" as a magic latency button or as a system whose behavior must be measured. They want to see that I can (a) verify the cache is even being used, (b) reason about hit rate, key design, and where the real bottleneck actually is, and (c) recognize that caching can hide or even worsen problems if applied carelessly.

Clarifying questions I'd ask first:

"Feels slow" — slow at p50, or only p99 tail? For all requests or a subset? Is this a measured regression against a baseline, or subjective?
What did I cache, and where is the bottleneck actually — the thing I cached, or something else in the path?
What's the cache hit rate right now? (If I don't know this, that's the first thing to instrument — it's the single most important number.)
What's the cache topology: in-process (local) vs remote (Redis/Memcached)? A remote cache adds a network round-trip.
What's the TTL and eviction policy, and how big is the working set vs the cache?

Assumption for the walkthrough: a read-heavy API where I added a Redis cache in front of a DB query, expecting p95 to drop, and it didn't.

First principle: a cache only helps if it's hit, and if the thing cached was the bottleneck

Caching gives effective_latency ≈ hit_rate × cache_latency + (1 − hit_rate) × (miss_latency), where a miss is usually slower than no cache at all (cache lookup + backend + cache write). So there are only a few ways "added a cache, still slow" happens, and I'd check them in order.

Step 1 — Measure the hit rate (is the cache even working?)

Instrument cache_hits / cache_misses and compute hit rate. This immediately splits the problem:

Low hit rate → the cache isn't earning its keep; go to Step 2.
High hit rate but still slow → the cache works but wasn't the bottleneck, or the cache itself is slow; go to Steps 3–4.

If I can't answer "what's the hit rate," I've already found a gap: caching without observability is flying blind.

Step 2 — Why the hit rate is low

Common root causes:

Cache-key too specific / high cardinality. If the key includes a timestamp, a request ID, a full query string with volatile params, or per-user data that's rarely repeated, almost every request is a miss. Normalize the key to the stable identity of the data.
TTL too short relative to request inter-arrival time — the entry expires before it's reused.
Working set > cache size → constant eviction (thrashing). Check eviction/evicted_keys metrics; the cache is full and LRU is throwing out entries before reuse.
Low reuse / long-tail access pattern. If each item is requested roughly once (e.g., unique search queries), there's little temporal locality to exploit — a cache fundamentally can't help much here; that's a data-access-pattern reality, not a bug.
Cache not actually populated / write path broken — reads miss and the code silently falls through to the DB. Verify writes land (check key count).
Invalidation too aggressive — writes bust more than they should, or a bad key format means every write clears everything.
Step 3 — The cache works, but it wasn't the bottleneck (Amdahl's law)

This is the classic trap. If the cached DB call was only 20% of request time, caching it perfectly caps the improvement at 20% — the request will still "feel slow" because 80% is elsewhere: serialization, an N+1 loop, a second uncached downstream call, TLS handshakes, GC pauses, template rendering, or client-side render time. The fix is to profile the request end-to-end with a distributed trace and cache/optimize the span that actually dominates. Optimizing the non-bottleneck is wasted effort.

Step 4 — The cache itself is the cost

A remote cache adds a network hop and (de)serialization. If values are large (big JSON blobs), serialization + network transfer can rival or exceed the DB fetch, especially at p99. Symptoms: cache latency itself is high in traces. Fixes: store smaller/derived values, use a faster serializer, use an in-process/near cache for the hottest keys (two-tier: local L1 + Redis L2), or compress. Also check: is the Redis node saturated (CPU, single-threaded command queue, big-key O(n) commands, network bandwidth)? A hot Redis becomes its own bottleneck and slows everyone.

Step 5 — Consistency / correctness masking as "slow"

Sometimes "still slow" is really "sometimes wrong then retried." Stale reads, a thundering-herd on expiry (many concurrent misses all hitting the DB at once — a cache stampede — spiking DB latency and thus overall latency right after each TTL boundary), or double-fetching. Check whether p99 spikes align with TTL boundaries; if so, add stampede protection (single-flight/locking, stale-while-revalidate, jittered TTLs).

Step 6 — Measurement / deployment gotchas
Am I measuring a cold cache? Right after deploy the cache is empty; give it time to warm before judging, or pre-warm it.
Is the cache actually in the path for these requests? A feature flag, a code branch, or a bypassed layer can mean it's not being consulted at all. Confirm with a trace or a metric on the cache client.
Is the "slow" perception from the write path or an endpoint I never cached?
Mitigate → verify → fix
Instrument first: hit rate, cache-op latency, evictions, and a trace span for the cache call. You cannot fix what you can't see.
Based on the number: fix the key/TTL/size (Step 2), or re-target the cache at the real bottleneck (Step 3), or shrink values / add an L1 (Step 4), or add stampede protection (Step 5).
Verify on the same latency dashboard that p50/p95/p99 actually moved, and that DB load dropped (a good sign the cache is absorbing reads).
Set an alert on hit rate so a future key/TTL regression is caught automatically.
Summary — what a strong answer emphasizes

The core insight: "added a cache" is a hypothesis, not a fix — validate it with hit rate and a trace. A strong answer (1) checks the hit rate first, (2) reasons about why it's low (key cardinality, TTL, working-set size, reuse pattern, broken write path, over-eager invalidation), (3) invokes Amdahl's law — caching the non-bottleneck can't help, so profile end-to-end, (4) recognizes the cache can be the cost (remote hop, big values, saturated Redis, stampede), and (5) accounts for cold-cache measurement and code-path bypasses. Common mistakes: assuming the cache is hitting without measuring, caching something that wasn't on the critical path, ignoring serialization/network cost of a remote cache, and forgetting stampede/cold-start effects. Follow-up probes: "Hit rate is 90% but it's still slow — now what?" (bottleneck is elsewhere → trace; or Redis/serialization is the cost). "How would you pick the TTL?" (data volatility vs reuse rate; add jitter to avoid synchronized expiry).

**Comparison:**

```
Observation	Likely cause	Fix
Hit rate near 0	Key too specific / high cardinality	Normalize key to stable data identity
Hit rate low, evictions high	Working set > cache size (thrashing)	Bigger cache, shard, or cache fewer/hotter keys
Entries expire before reuse	TTL too short vs reuse interval	Raise TTL (add jitter)
High hit rate, still slow	Cached span wasn't the bottleneck (Amdahl)	Trace end-to-end; cache the dominant span
Cache-op latency high in trace	Remote hop + big values / Redis saturated	Smaller values, L1 local cache, faster serializer
p99 spikes at TTL boundaries	Cache stampede / thundering herd	Single-flight, stale-while-revalidate, jittered TTL
Slow only right after deploy	Cold cache	Pre-warm or wait to warm before judging
Cache seemingly ignored	Code path / flag bypasses cache	Confirm via trace it's actually consulted
```

**Code:**

```
The bug this question often hides: high-cardinality key => ~0% hit rate.
BAD: key includes volatile params => almost every request is a unique key => always a miss.

def get_products_bad(user_id, request_id, ts): key = f"products:{user_id}:{request_id}:{ts}" # request_id + ts make it unique ...

GOOD: key on the STABLE identity of the data; normalize/omit volatile params.

def get_products(category, page): key = f"products:{category}:page={page}" hit = redis.get(key) if hit is not None: metrics.incr("cache.hit") return deserialize(hit) metrics.incr("cache.miss") val = db.query_products(category, page) # the expensive path redis.set(key, serialize(val), ex=jitter(300)) # TTL + jitter (anti-stampede) return val

CODE
Copy

```python
# Instrument first — you can't debug a cache you can't measure.
def hit_rate(window="5m"):
    h = metrics.rate("cache.hit", window)
    m = metrics.rate("cache.miss", window)
    return h / (h + m) if (h + m) else float("nan")
# Alert if hit_rate < 0.7 (regression in key/TTL/size will trip this).
```

**Example input / output:**

```text
Dry-run of the investigation:

Input : "Added Redis in front of GET /products; p95 unchanged at 480ms."

Q: hit rate?  -> 4%.  => cache barely used. Go to Step 2.
Inspect keys  -> keys look like products:{uid}:{reqId}:{ts}
                => reqId + ts make every key unique => ~always miss.
Fix           -> key = products:{category}:page=N
Re-measure    -> hit rate 88%; DB QPS drops 6x; p95 480ms -> 90ms. Verified.

Contrast (hit rate was already high):
Q: hit rate?  -> 92%, yet p95 still 460ms.
Trace         -> cached DB span is only 60ms of 460ms; 380ms is a second,
                 UNCACHED call to a recommendations service.
Root cause    -> Amdahl: cached the wrong (non-dominant) span.
Fix           -> cache/optimize the recommendations call instead -> p95 falls.
```

**Flow:**

1. Low
2. High
3. In cached span
4. In a DIFFERENT span
5. p99 spikes at TTL edges
6. Added cache, app still feelsslow
7. What is the hit rate?(instrument if unknown)
8. Why low?
9. Key too specific / highcardinality
10. TTL too short
11. Working set > cache size ->evictions
12. Low reuse / long-tail access
13. Write path broken / notpopulated
14. Open end-to-end trace
15. Where is the time now?
16. Cache itself is slow: remotehop, big value, Redis saturated-> shrink value / add L1
17. Amdahl: cached thenon-bottleneck -> re-targetcache/optimize the dominantspan
18. Stampede -> single-flight +stale-while-revalidate + jitter
19. Fix, then verify p50/p95/p99moved AND DB load dropped

