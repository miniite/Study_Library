---
type: "practice"
difficulty: "Expert"
---

# What is a cache stampede and how would you prevent it?

**Difficulty:** Expert · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Read-through and Write-behind Caching](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Define a cache stampede precisely, explain the failure mechanism (why it's dangerous — it can take down the backing store and cascade), and lay out the full toolbox of prevention techniques with their trade-offs. Because the level is "expertise" (Architect/Principal-adjacent), a strong answer doesn't just name "use a lock" — it distinguishes the sub-variants of the problem, compares mitigations on consistency/latency/complexity, and shows judgment about when each applies at scale.

Clarifying framing / assumptions: distributed system, a shared remote cache (Redis/Memcached) in front of an expensive backend (a DB query, an aggregation, or a downstream API call), high read QPS. The cached value is hot (requested thousands of times per second).

What a cache stampede is (definition)

A cache stampede (a.k.a. thundering herd, dog-piling, or cache miss storm) happens when a popular cache entry becomes unavailable — most commonly because it expires (TTL elapses) or is evicted/invalidated — and, in the small window before it's repopulated, a flood of concurrent requests all miss simultaneously and all stampede to the expensive backend to recompute the same value at once.

The essence: for a hot key, the intended behavior is "1 request recomputes, N−1 read from cache." During a stampede it degenerates to "all N requests recompute the identical value at the same time." If a key gets 10,000 req/s and takes 200ms to recompute, then at the instant of expiry roughly 2,000 concurrent requests hit the DB with the identical query — wasteful at best, and at worst enough to overload the DB.

Why it's dangerous (the failure cascade):

TTL expires → mass miss → DB load spikes ~N×.
The recompute now takes longer because the DB is overloaded → the repopulation window widens → more requests pile in during that window → positive feedback loop.
DB latency climbs → app connection pools exhaust → timeouts → clients retry, amplifying load further (a metastable failure — the system can't recover even after the trigger passes, because retries and slow recomputes sustain the overload).
This can take down not just the slow endpoint but the whole DB, affecting unrelated traffic. Real outages (the classic "Facebook memcache" and many others) trace to exactly this.

There are three related-but-distinct sub-problems worth separating, because they need different fixes:

Expiry stampede: a key with a fixed TTL expires and many readers miss at once. (The main case.)
Cold-start / cache-fill stampede: the cache is empty (fresh deploy, flush, new node) and many distinct keys all miss together.
Miss on a never-cached / non-existent key (cache penetration): requests for keys that don't exist in the backend either (e.g., malicious IDs) bypass the cache entirely and always hit the DB.
Prevention techniques (with trade-offs)
1. Request coalescing / single-flight (locking / mutex)

On a miss, only one request is allowed to recompute; the others wait for the result (or briefly serve stale). Implemented with a distributed lock (e.g., Redis SET key_lock uuid NX EX 5) or an in-process single-flight per node. The winner recomputes, writes the cache, releases the lock; losers re-read.

#### Pros
guarantees exactly-once recompute; simplest correct behavior.

#### Cons
the waiters block (adds latency to that batch); needs a lock TTL to avoid a crashed holder deadlocking everyone; a distributed lock has its own failure modes (fencing, clock skew — the classic Redlock debate). Per-node single-flight (e.g., Go's singleflight) is cheap and avoids a distributed lock but still lets one recompute per node — usually acceptable and much simpler.
2. Stale-while-revalidate (serve-stale + async refresh)

Keep the value past its logical expiry with a soft TTL < hard TTL. On a read after soft-expiry, serve the stale value immediately and trigger one background refresh; only after the hard TTL is it truly gone. Readers almost never block, and only one refresh runs.

#### Pros
best latency (no one waits), naturally coalesces the refresh, resilient if the backend is momentarily down (serve stale). Widely used (HTTP stale-while-revalidate, Varnish "grace mode").

#### Cons
you serve slightly stale data — only acceptable where bounded staleness is fine (most read caches). Needs care to ensure only one refresh is triggered.
3. Probabilistic early expiration (XFetch)

Instead of everyone expiring at the same instant, each reader independently decides to recompute early, with a probability that rises as the TTL approaches. Using the recompute cost delta and a gain factor beta: recompute if now − delta·beta·ln(rand()) ≥ expiry. This spreads recomputes over time so effectively one lucky reader refreshes before expiry while others keep hitting the cache — no synchronized cliff, no lock.

#### Pros
elegant, lock-free, no synchronized thundering herd; probabilistically one refresh.

#### Cons
occasional slightly-early recompute (small extra backend load); needs storing the recompute duration.
4. TTL jitter / randomization

Never set the same TTL on many keys populated together — add random jitter (e.g., TTL = base ± rand(0..20%)). This directly prevents cold-start and batch-populated stampedes where thousands of keys expire on the same second.

#### Pros
trivial, near-free, essential hygiene.

#### Cons
doesn't help a single hot key (only one key), so combine with #1/#2/#3.
5. Cache warming / pre-computation

For predictable hot keys, proactively refresh them on a schedule (a background job recomputes and writes before expiry) or pre-warm after a deploy/flush so the cache is never cold under live traffic.

#### Pros
hot keys effectively never expire under user traffic; smooth backend load.

#### Cons
operational cost; only feasible for a known set of hot keys.
6. Negative caching + Bloom filter (for penetration)

For the "non-existent key" variant, cache the miss (a short-TTL "null" marker) so repeated lookups for a bad key don't keep hitting the DB, and/or front the cache with a Bloom filter of existing keys to reject definitely-absent keys in memory.

#### Pros
stops penetration attacks / bad-ID floods.

#### Cons
negative TTL must be short to reflect real inserts; Bloom filter has false positives (never false negatives) and must be kept in sync.
7. Load shedding, timeouts, circuit breakers, and bounded jittered retries

Defense-in-depth so a stampede can't metastasize: timeouts on the backend call, a circuit breaker that trips when the DB is unhealthy, exponential backoff with jitter on client retries (un-jittered retries synchronize and cause their own herd), and load shedding (return 503 / serve stale) when saturated. These don't prevent the herd but stop it from becoming a full outage.

Choosing — the judgment part
Default for a hot read key where bounded staleness is acceptable: stale-while-revalidate + jittered TTL (great latency, coalesced refresh). Add probabilistic early expiration for the very hottest keys to avoid a cliff entirely.
When staleness is not acceptable (must be fresh): single-flight/lock so exactly one recompute runs and others wait — accept the small latency hit.
For mass-expiry / cold-start: jitter + warming.
For bad-key floods: negative caching + Bloom filter.
Always layer in timeouts + jittered-backoff retries + circuit breaker as the safety net.

These compose: a robust hot-key path is often stale-while-revalidate, with a single-flight guard on the refresh, jittered TTLs, and a circuit breaker on the backend. Don't over-engineer a cold key that gets 2 req/min — reserve the machinery for genuinely hot keys.

Summary — what a strong (expert) answer emphasizes

Define the stampede as synchronized mass misses on a hot key all recomputing the same value, and stress the danger: it's a positive-feedback / metastable failure that can overload the backing store and cascade via retries — not just wasted CPU. Then present the toolbox and when each applies: single-flight/locking (exactly-once, but blocks), stale-while-revalidate (best latency, tolerates staleness), probabilistic early expiration (lock-free, no cliff), TTL jitter + warming (cold-start / batch expiry), negative caching + Bloom filter (penetration), all backstopped by timeouts, circuit breakers, and jittered-backoff retries. The senior signal is (1) separating the sub-problems (expiry vs cold-start vs penetration), (2) comparing on consistency vs latency vs complexity, and (3) composing a couple of techniques rather than reaching for a heavyweight distributed lock for everything.

#### Common mistakes
/ follow-up probes: treating a distributed lock as the only answer (heavy, has its own failure modes) instead of the lighter per-node single-flight or stale-while-revalidate; forgetting retry jitter (un-jittered retries create their own herd); setting identical TTLs across many keys (mass-expiry cliff); and ignoring the metastable-failure risk. Likely probes: "How does your lock behave if the holder crashes mid-recompute?" (lock TTL + fencing token, or fall back to serve-stale), and "How would you protect against a flood of requests for keys that don't exist?" (negative caching + Bloom filter).

**Comparison:**

```
Technique	Prevents which variant	Consistency	Latency impact	Complexity	Best for
Single-flight / lock	Expiry (hot key)	Strong (fresh)	Waiters block briefly	Medium (distributed lock has own risks)	Must-be-fresh hot key
Stale-while-revalidate	Expiry (hot key)	Bounded staleness	Best (no blocking)	Medium	Read cache, staleness OK
Probabilistic early expiry (XFetch)	Expiry cliff	Bounded staleness	Very low	Medium	Very hot single key
TTL jitter	Cold-start / mass expiry	Neutral	None	Trivial	Batch-populated keys
Cache warming / precompute	Cold-start, hot keys	Fresh	None (async)	Ops cost	Known hot set
Negative cache + Bloom	Penetration (bad keys)	Short-lived	Low	Medium	Bad-ID / attack floods
Timeout + circuit breaker + jittered retry	Cascade/metastability	n/a	Sheds load	Medium	Safety net (always)
```

**Code:**

```

import math, random, time

--- Technique 1: single-flight via a Redis lock (exactly-once recompute) ---

def get_with_singleflight(key, recompute, ttl=300): val = redis.get(key) if val is not None: return deserialize(val) lock = f"lock:{key}" # NX = only if not set; EX = lock auto-expires so a crashed holder can't deadlock. if redis.set(lock, "1", nx=True, ex=5): try: val = recompute() # only ONE caller reaches here redis.set(key, serialize(val), ex=jitter(ttl)) return val finally: redis.delete(lock) else: time.sleep(0.05) # brief backoff, then re-read v = redis.get(key) return deserialize(v) if v is not None else recompute() # fallback

--- Technique 4: TTL jitter (kills synchronized mass expiry) ---

def jitter(ttl, pct=0.2): return int(ttl * (1 + random.uniform(-pct, pct)))

--- Technique 3: probabilistic early expiration (XFetch, lock-free, no cliff) ---

def get_xfetch(key, recompute, ttl=300, beta=1.0): packed = redis.get(key) # stored as {value, delta, expiry} if packed: val, delta, expiry = unpack(packed) # recompute early with rising probability as expiry approaches if time.time() - delta * beta * math.log(random.random()) < expiry: return val start = time.time() val = recompute() delta = time.time() - start # remember recompute cost expiry = time.time() + ttl redis.set(key, pack(val, delta, expiry), ex=jitter(ttl)) return val

CODE
Copy

```python
# --- Technique 2: stale-while-revalidate (soft TTL < hard TTL; serve stale, refresh once) ---
def get_swr(key, recompute, soft=300, hard=360):
    entry = redis.get(key)                         # {value, soft_expiry}
    if entry:
        value, soft_expiry = unpack2(entry)
        if time.time() < soft_expiry:
            return value                           # fresh
        # stale-but-usable: serve now, refresh in background (guarded by single-flight)
        if redis.set(f"refresh:{key}", "1", nx=True, ex=10):
            enqueue_async(lambda: _refresh(key, recompute, soft, hard))
        return value                               # nobody blocks
    return _refresh(key, recompute, soft, hard)    # truly cold -> compute inline (add lock)
```

**Example input / output:**

```text
Illustration of the herd and a fix:

Hot key "leaderboard:top" : 10,000 req/s, recompute = 200ms, TTL = 300s.

WITHOUT protection (expiry stampede):
  t=300.000s  TTL expires
  t=300.000..300.200s  ~2,000 concurrent requests all MISS
                       => ~2,000 identical DB queries fire at once
  DB CPU spikes -> recompute now takes 900ms -> window widens ->
  more requests pile in -> pool exhaustion -> timeouts -> client retries
  => metastable overload (does NOT self-heal when TTL trigger passes).

WITH single-flight + jitter + stale-while-revalidate:
  Only 1 request recomputes (200ms); the other ~1,999 read cache or serve stale.
  DB sees 1 query instead of ~2,000. No latency cliff, no cascade.
```

**Data:**

```text
# Negative caching + Bloom filter (penetration variant)
bloom = BloomFilter(existing_ids)     # in-memory; false positives possible, no false negatives
def get_user(id):
    if id not in bloom:               # definitely absent -> reject without touching DB
        return None
    hit = redis.get(f"user:{id}")
    if hit == NULL_MARKER: return None    # negative cache: known-missing, short TTL
    if hit is not None:     return hit
    row = db.get_user(id)
    redis.set(f"user:{id}", row or NULL_MARKER, ex=(300 if row else 30))  # cache miss briefly
    return row
```

**Flow:**

1. Backend/DB
2. Cache
3. Many clients
4. Backend/DB
5. Cache
6. Many clients
7. WITHOUT protection - stampede
8. WITH single-flight + stale-while-revalidate
9. GET hot_key (TTL just expired)
10. MISS (to ALL N callers)
11. N identical recompute queries at once
12. overload -> slower -> window widens -> retries -> cascade
13. GET hot_key (soft-expired)
14. serve STALE immediately (no block)
15. SET refresh_lock NX (only 1 wins)
16. 1 recompute query
17. value
18. SET hot_key (jittered TTL)

