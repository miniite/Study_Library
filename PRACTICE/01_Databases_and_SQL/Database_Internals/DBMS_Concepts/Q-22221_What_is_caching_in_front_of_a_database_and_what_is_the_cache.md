---
type: "practice"
difficulty: "Intermediate"
---

# What is caching in front of a database and what is the cache-aside pattern?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Distributed Transactions and 2PC](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Distributed_Transactions_and_2PC.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Two linked things — (1) why we put a cache (e.g. Redis/Memcached) in front of a database, and (2) explain the cache-aside (a.k.a. lazy-loading) pattern in detail: its read path, write path, failure modes, and how it compares to the other caching strategies. This is "intermediate experience" for a backend/fullstack SDE, so the interviewer wants a practitioner answer: not just the definition, but the invalidation/consistency pitfalls, TTL choices, the thundering-herd/stampede problem, and when not to use it.

What the interviewer wants to hear:

The motivation: latency, DB load reduction, cost, read-heavy skew.
The exact cache-aside algorithm (read: check cache → miss → load DB → populate cache; write: update DB → invalidate cache).
The consistency subtlety — why we delete rather than update the cache on writes, and the classic race that still causes stale data.

#### Trade-offs
vs read-through / write-through / write-behind, and mitigations (TTL, jitter, stampede protection).

Plan: Motivation → cache-aside read & write flows step by step → why delete-not-update → failure modes (stale data, stampede, cold start) and fixes → comparison table → summary.

Why cache in front of a database

A database (especially a relational one with joins, indexes, and durability) is comparatively slow and expensive per query. A cache is an in-memory key-value store (Redis, Memcached) that serves reads in sub-millisecond time. We add it because:

Latency: an in-memory GET is ~0.1–1 ms vs 5–50 ms for a DB query — often 10–100× faster.
Reduced DB load / cost: most workloads are read-heavy and skewed (a small set of "hot" keys — a trending product, a celebrity's profile — gets most traffic). Caching those absorbs the bulk of reads so the DB isn't the bottleneck, letting you scale reads cheaply without over-provisioning the DB.
Throughput / spike absorption: a cache handles flash-sale or viral traffic that would otherwise overwhelm the DB.

The cost is a new consistency problem: the cache is a copy of the data that can go stale, so caching strategy is really about how you keep the copy fresh.

The cache-aside (lazy-loading) pattern

In cache-aside, the application code — not the cache — owns the logic of talking to both the cache and the DB. The cache is a "dumb" side store the app reads and writes around. This is the most common caching pattern and the default with Redis + a SQL/NoSQL DB.

Read path:

Application looks up the key in the cache.
Cache hit → return the value directly (fast; DB untouched).
Cache miss → application queries the database, gets the value, writes it into the cache (usually with a TTL), and returns it. The next read of that key is a hit.

Write path (on update/delete of the underlying data):

Application writes to the database (the source of truth).
Application invalidates (deletes) the key from the cache — it does not try to overwrite the cache with the new value.

That's cache-aside: lazy because data is loaded into the cache only when first requested (on demand), never proactively.

Why delete the cache on write, not update it

This trips people up. On a write we evict the stale key rather than write the new value into the cache, for two reasons:

Correctness under concurrency: if two requests update the same row and both then write their value to the cache, they can interleave so the cache ends up holding the older value. Deleting is idempotent and safe — worst case is an extra miss (a cheap DB reload), not permanent staleness.
Efficiency / avoiding wasted work: many written keys are never read again soon; computing and pushing the new cached value (often a joined/derived object) is wasted effort. Delete now, let the next reader lazily repopulate only if the data is actually needed.

(This is the widely used "write to DB, then invalidate cache" rule.)

Failure modes and how a practitioner handles them

1. Stale reads (the fundamental cache-aside race). Even with delete-on-write, a classic race exists:

Request A reads, gets a cache miss, loads value v1 from DB (but hasn't written it to cache yet).
Request B updates the DB to v2 and deletes the cache key.
Request A now writes its stale v1 into the cache.
Result: cache holds v1 while DB holds v2 — stale until TTL expiry.

Mitigations: always set a TTL (bounds how long any staleness can last), use delete-after-write ordering carefully, or for stronger needs use techniques like delayed double-delete (delete cache, write DB, then delete cache again after a short delay) or versioned keys. Accept that cache-aside is eventually consistent, not strongly consistent.

2. Cache stampede / thundering herd. A hot key expires (or the cache is cold) and thousands of concurrent requests all miss simultaneously → all hammer the DB with the same query at once, possibly toppling it. Fixes: request coalescing / single-flight (one request loads, others wait), mutex/lock on repopulation, probabilistic early re-computation, and TTL jitter (randomize expiry so keys don't all expire together).

3. Cold start. After a deploy/restart the cache is empty, so latency and DB load spike until it warms. Fixes: cache warming (preload hot keys) or gradual ramp.

4. TTL choice. Too long → more staleness; too short → more misses and DB load. Choose per-key based on how tolerant the data is to staleness (a product price: short; a country list: long).

#### Trade-offs
vs other strategies (the practitioner comparison)
Cache-aside (lazy): app manages both stores; cache only holds requested data; resilient (cache down → app still reads DB, just slower). Downsides: first read of any key is always a miss (added latency), and app code carries the cache logic. Best for read-heavy, general-purpose caching.
Read-through: the cache library sits inline and loads from the DB on a miss itself; app just talks to the cache. Cleaner app code, but couples you to a cache that understands your DB.
Write-through: write to cache and DB synchronously on every write → cache always fresh, but every write pays double latency and you cache data that may never be read.
Write-behind (write-back): write to cache, flush to DB asynchronously → very fast writes, but risk of data loss if the cache dies before flushing. Used where write throughput dominates and some loss is tolerable.
Summary — what a strong answer emphasizes
Why cache: in-memory reads are 10–100× faster and shield a read-heavy, hot-key-skewed workload from an expensive DB — better latency, less DB load, cheaper scaling.
Cache-aside read: check cache → hit returns; miss → load DB, populate cache with TTL, return.
Cache-aside write: update DB, then delete (invalidate) the cache key — delete not update, for concurrency-safety and to avoid wasted work.
Know the pitfalls: it's eventually consistent (stale-read race), needs TTLs, and must be protected against stampede (single-flight + TTL jitter) and cold start (warming).
Be able to contrast with read-through / write-through / write-behind and say when each fits.

#### Common mistakes
/ follow-up probes:

Saying "update the cache on write" without noting the concurrency race that makes delete the safer default.
Forgetting TTLs (unbounded staleness) or ignoring cache stampede.

#### Follow-ups
"How do you keep cache and DB consistent?" (delete-on-write + TTL, delayed double-delete, or CDC-driven invalidation), "What happens on a hot-key expiry under load?" (stampede → single-flight/jitter), "Cache-aside vs write-through — which for a write-heavy counter?" (write-behind or in-cache atomic increment).

**Comparison:**

```
Strategy	Read on miss	Write behavior	Freshness	Main risk	Best for
Cache-aside (lazy)	App loads DB, fills cache	Update DB, delete cache key	Eventually consistent	Stale-read race, cold miss	Read-heavy general purpose
Read-through	Cache loads DB itself	Usually paired w/ write-through	Same as cache-aside	Cache/DB coupling	Cleaner app code
Write-through	(cache pre-filled on write)	Write cache and DB synchronously	Always fresh	Slow writes, caches unread data	Read-after-write, low write rate
Write-behind	(as above)	Write cache, flush DB async	Cache fresh, DB lags	Data loss if cache dies	Write-heavy, loss-tolerant
Failure mode	Cause	Fix
Stale reads	Concurrent read-miss + write race	TTL, delete-on-write, delayed double-delete, versioned keys
Cache stampede	Hot key expires, mass simultaneous miss	Single-flight/mutex, TTL jitter, early recompute
Cold start	Empty cache after restart/deploy	Cache warming, gradual ramp
```

**Code:**

```python
import json

# Cache-aside with Redis in front of a SQL DB
TTL_SECONDS = 300  # bound staleness

def get_user(user_id, redis, db):
    key = f"user:{user_id}"

    # 1. Try the cache
    cached = redis.get(key)
    if cached is not None:
        return json.loads(cached)          # cache HIT

    # 2. Cache MISS -> load from DB
    row = db.query_one("SELECT * FROM users WHERE id = %s", user_id)
    if row is None:
        return None

    # 3. Populate cache with a TTL, then return
    redis.set(key, json.dumps(row), ex=TTL_SECONDS)
    return row

def update_user(user_id, changes, redis, db):
    # 1. Write to the source of truth first
    db.execute("UPDATE users SET name = %s WHERE id = %s",
               changes["name"], user_id)

    # 2. INVALIDATE (delete) the cache key -- do NOT overwrite it.
    #    Next read lazily repopulates from the fresh DB value.
    redis.delete(f"user:{user_id}")

# --- Stampede protection sketch: single-flight on repopulation ---
def get_user_safe(user_id, redis, db):
    key = f"user:{user_id}"
    cached = redis.get(key)
    if cached is not None:
        return json.loads(cached)

    lock = f"lock:{key}"
    if redis.set(lock, "1", nx=True, ex=5):   # only one loader wins
        try:
            row = db.query_one("SELECT * FROM users WHERE id = %s", user_id)
            # TTL with jitter so hot keys don't all expire together
            import random
            redis.set(key, json.dumps(row), ex=TTL_SECONDS + random.randint(0, 60))
            return row
        finally:
            redis.delete(lock)
    else:
        # someone else is loading; brief wait then re-read the cache
        import time; time.sleep(0.05)
        cached = redis.get(key)
        return json.loads(cached) if cached else None
```

**Example input / output:**

```text
Read path dry-run (key user:42):

Request 1 (cold):
  GET cache["user:42"] -> nil                (MISS)
  SELECT * FROM users WHERE id=42 -> {id:42,name:"Asha"}   (DB hit ~20ms)
  SET  cache["user:42"] = {...} EX 300
  return {id:42,name:"Asha"}

Request 2 (warm, moments later):
  GET cache["user:42"] -> {id:42,name:"Asha"}  (HIT, ~0.3ms, DB untouched)
  return {id:42,name:"Asha"}

Write path dry-run:
  update_user(42, name="Asha Rao")
  UPDATE users SET name='Asha Rao' WHERE id=42   (DB = source of truth)
  DEL cache["user:42"]                            (invalidate)

Next read:
  GET cache["user:42"] -> nil (MISS, because we deleted)
  SELECT ... -> {id:42,name:"Asha Rao"}   (fresh)
  SET cache["user:42"] = {...} EX 300
  return {id:42,name:"Asha Rao"}
```

**Flow:**

1. WRITE
2. App update/delete
3. Write to database - source oftruth
4. DELETE / invalidate cache key
5. Next read lazily repopulates
6. READ
7. Hit
8. Miss
9. App reads key
10. In cache?
11. Return cached value
12. Query database
13. Populate cache with TTL
14. Return value

