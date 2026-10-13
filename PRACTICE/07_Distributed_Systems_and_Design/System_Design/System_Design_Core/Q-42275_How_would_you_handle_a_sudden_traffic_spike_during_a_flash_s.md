---
type: "practice"
difficulty: "Deep"
---

# How would you handle a sudden traffic spike during a flash sale?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Rate Limiter Storage](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Rate_Limiter_Storage/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Design for a predictable but extreme, short-lived traffic spike — a flash sale (Big Billion Days, iPhone drop, Swiggy lunch surge). The interviewer wants an end-to-end plan: how to absorb 10–100x normal load without falling over, keep the checkout/payment path correct (no overselling, no double-charge), and degrade gracefully. Level "deep understanding" means I should cover the full stack — CDN, caching, autoscaling, queueing the write path, inventory correctness, DB protection, rate limiting/fairness, and observability — plus the judgment of what to prioritize.

What the interviewer is probing: Do you distinguish the read path (product pages, catalog — huge but cacheable) from the write path (add-to-cart, checkout, inventory decrement, payment — smaller but must be correct)? Do you know autoscaling alone is not enough (it's slow, and the DB is the bottleneck)? Can you protect the system with load shedding and keep inventory consistent?

Clarifying questions I'd state: Is the sale time known in advance (yes → pre-warm/pre-provision)? How limited is stock (deeply limited → fairness + no oversell matters most)? What's the acceptable degradation (can we queue users, show a waiting room)? Is payment third-party (Razorpay/Stripe) with its own limits?

Plan: Pre-event provisioning → protect the read path (CDN/cache) → protect the write path (queue + async) → inventory correctness → database protection → rate limiting/fairness/waiting room → graceful degradation → autoscaling reality → observability → summary and trade-offs.

First principle: separate the read path from the write path

99% of flash-sale traffic is reads (browsing the deal page) and 1% is writes (actually buying). They have opposite requirements:

Reads are massive but cacheable and idempotent → push them to the edge, serve from cache, never hit the origin DB.
Writes are fewer but must be correct and consistent (don't oversell 100 phones to 100,000 people, don't double-charge) → protect with queueing, idempotency, and careful DB access.

Handle them with different strategies. Failing to separate them is the classic mistake.

Protect the read path
CDN + edge caching. Put static assets, images, and even the product/deal HTML/JSON behind a CDN (CloudFront/Fastly/Akamai). A flash-sale landing page can be cached for seconds and served entirely from the edge, cutting origin traffic by 95%+.
Pre-warm the cache and CDN before the sale so the first burst doesn't miss and stampede the origin.
Application cache (Redis) for hot data — product details, price, and a read-only view of remaining stock (approximate is fine for display). Serve "recommended" and catalog data from cache with short TTLs.
Prevent cache stampede: use request coalescing / single-flight so that when a hot key expires, only one request refills it instead of thousands hitting the DB simultaneously.
Protect the write path — queue and go async

The purchase is the dangerous part. Don't try to do a synchronous DB transaction per user under 100x load.

Pattern: accept fast, process asynchronously.

When a user hits "Buy," do a fast, cheap eligibility check (is stock plausibly available? is the user rate-limited?) and enqueue the order request into a durable queue (Kafka/SQS).
Return immediately: "Your order is being processed" (HTTP 202), with a token to poll status.
A pool of workers drains the queue at a controlled rate the database can sustain, doing the real inventory decrement and order creation.

This converts an uncontrollable spike into a smooth, bounded stream — the queue is a shock absorber. The DB sees a steady load, not a tsunami. If the queue backs up, users wait (or you shed) rather than the system crashing.

Inventory correctness — no overselling

This is the correctness crux. Options, in order of scalability:

Atomic decrement in Redis as the source of truth for stock during the sale: DECR stock:{sku} returns the new value; if it goes below 0, reject. Redis single-threaded atomicity prevents oversell and handles enormous throughput. Reconcile to the relational DB asynchronously.
DB with atomic conditional update: UPDATE inventory SET qty=qty-1 WHERE sku=? AND qty>0 and check rows-affected. Correct, but the DB becomes the bottleneck under extreme load — use row-level locking carefully and keep the transaction tiny.
Pre-allocate/token bucket: issue N "purchase tokens" (one per unit of stock) up front; only token holders can check out. Naturally caps demand to supply and gives fairness.

Whatever the store, the reservation must be atomic and paired with a timeout — if the user doesn't pay within, say, 5 minutes, release the reserved unit back.

Idempotency and payment

Under retries and at-least-once queues, the same order can be submitted twice. Use an idempotency key per checkout attempt so the order/charge happens exactly once. Payment gateways (Razorpay/Stripe) accept an idempotency key too. Also respect the gateway's own rate limits — buffer/queue toward it.

Protect the database
Add read replicas and route all reads to them; keep the primary for writes only.
Use connection pooling (PgBouncer) — a spike of connections can kill Postgres faster than the queries themselves.
Keep transactions short; avoid long locks on hot rows (the single hot SKU is a contention point — Redis offloads that).
Shard/partition hot data if a single SKU's row is the hotspot (e.g., split stock into K counters and sum).
Rate limiting, fairness, and a waiting room
Rate limit per user/IP at the edge/API gateway to blunt bots and hoarders (token bucket).
Load shedding: when the system is at capacity, reject early and cheaply at the edge with a friendly "try again" rather than accepting work you can't finish. Shed low-value traffic first.
Virtual waiting room (queue-it style): admit users into the sale in controlled waves. This is the gold standard for deeply limited stock — it makes the spike scheduled and fair, and shows users a fair position instead of a crash.
Graceful degradation

Decide in advance what to turn off under stress: disable recommendations, reviews, personalization, and non-essential analytics; serve a stripped-down deal page; show approximate stock. Keep the buy path alive and drop everything optional. A degraded-but-working checkout beats a pretty-but-down site.

Autoscaling — necessary but not sufficient

Autoscale the stateless tiers (web/API) ahead of time, but understand its limits: scaling takes minutes (instance boot + warmup), and it cannot scale a single primary database. For a known sale, pre-provision ("pre-scale") capacity before the start rather than relying on reactive autoscaling. Warm the load balancers/CDN. Autoscaling handles the tail and the unexpected; pre-provisioning handles the known cliff.

Observability and safety valves
Real-time dashboards on latency, error rate, queue depth, DB connections, cache hit rate.
Feature flags / kill switches to instantly disable heavy features or throttle harder mid-event.
Alerting on queue-depth growth (leading indicator of trouble) before user-facing errors appear.
Game-day / load test the whole flow beforehand at target scale.

#### Trade-offs
and judgment

#### Cons
istency vs. availability: for display stock, eventual/approximate is fine; for the decrement, you need strong atomicity. Split those.
Sync vs. async checkout: async (queue + 202) survives spikes far better but adds "pending" UX complexity and requires idempotency + status polling. For extreme sales it's worth it.
Cost: pre-provisioning wastes money if the sale underperforms; but for a known marquee event the cost of downtime (revenue + brand) dwarfs the over-provision cost.
Summary — what a strong answer emphasizes

Split read (cache/CDN, serve from edge, prevent stampede) from write (queue it, process async at a DB-safe rate, 202 + status). Guarantee no overselling with atomic Redis/DB decrements, reservations-with-timeout, and idempotency keys for exactly-once orders/payments. Protect the DB with replicas, pooling, and offloading the hot counter to Redis. Add rate limiting, load shedding, and a virtual waiting room for fairness; degrade gracefully by shedding optional features; and pre-provision + load-test for a known event rather than trusting reactive autoscaling alone. The winning mental model: the queue is a shock absorber that turns an uncontrollable spike into a controlled, correct stream.

**Comparison:**

```
Concern	Read path (browse)	Write path (buy)
Volume	Very high	Lower
Requirement	Fast, cacheable	Correct, consistent
Main tool	CDN + Redis cache, stampede protection	Queue + async workers, atomic reservation
Failure mode to avoid	Cache stampede on origin	Oversell / double-charge / DB meltdown
Technique	What it protects	Trade-off
CDN / edge cache	Origin & DB from read flood	Slight staleness of deal page
Async queue + 202	DB from write spike	"Pending" UX + idempotency needed
Redis atomic decrement	Inventory correctness	Async reconcile to RDBMS
Rate limit / load shed	Whole system from overload	Some users rejected early
Virtual waiting room	Fairness + scheduled spike	Users wait in line
Pre-provision capacity	Absorb known cliff	Cost if sale underperforms
Graceful degradation	Core buy path	Non-essential features off
```

**Code:**

```

-- Redis: atomic, oversell-proof stock decrement (run as an EVAL script) -- KEYS[1]=stock:{sku}, ARGV[1]=qty requested local remaining = tonumber(redis.call('GET', KEYS[1]) or '0') local want = tonumber(ARGV[1]) if remaining >= want then redis.call('DECRBY', KEYS[1], want) return 1 -- reserved OK else return 0 -- sold out -> reject fast end

CODE
Copy

```sql
-- DB fallback: atomic conditional decrement (no oversell). Check rows-affected == 1.
UPDATE inventory
SET    qty = qty - 1
WHERE  sku = :sku
  AND  qty > 0;
TEXT
Copy
# Async write path (pseudo): accept fast, process at a safe rate, idempotent
POST /checkout
  key = idempotencyKey(user, sku, clientToken)
  if orders.exists(key): return 200 existing        # exactly-once
  if !rateLimiter.allow(user): return 429
  if redisReserve(sku) == 0: return 409 SOLD_OUT     # fast reservation
  queue.publish({key, user, sku})                    # durable enqueue
  return 202 {status: "processing", poll: "/orders/"+key}

# worker (drains queue at DB-sustainable rate)
on message m:
  createOrder(m.key, m.user, m.sku)                  # idempotent by key
  charge(m.user, idempotencyKey=m.key)               # gateway idempotency
  markPaidOrReleaseReservationOnTimeout()
```

**Example input / output:**

```

Sold-out fast-fail example:

CODE
Copy
Request:  POST /checkout  {sku:"IPH-16", user:"U9"}
Redis:    stock:IPH-16 = 0  -> reserve returns 0
Response: 409 { "code": "SOLD_OUT" }   (returned in ~2ms, never touches DB)

Successful async purchase:

CODE
Copy
Request:  POST /checkout  {sku:"IPH-16", user:"U9", idem:"c-771"}
Redis:    stock:IPH-16 100 -> 99 (reserved)   Queue: enqueued c-771
Response: 202 { "status":"processing", "poll":"/orders/c-771" }

Poll:     GET /orders/c-771
Later:    200 { "status":"confirmed", "orderId":"ORD-5521" }

Oversell guard under race (2 buyers, 1 unit left):

CODE
Copy
stock=1
Buyer A EVAL -> remaining 1 >= 1 -> DECR -> 0 -> reserved (1)
Buyer B EVAL -> remaining 0 >= 1 -> false   -> rejected (0)
=> exactly one sale; no oversell (Redis atomicity)
```

**Data:**

```yaml
# Pre-event capacity plan (known sale @ 12:00)
prewarm_at: "11:30"
web_api_tier:
  min_instances_normal: 6
  pre_provisioned_for_sale: 120     # pre-scale, do not rely on reactive autoscale
  autoscale_target_cpu: 55%         # handles the tail beyond pre-provision
cache:
  redis_stock_source_of_truth: true
  cdn_cache_deal_page_ttl_s: 5
db:
  read_replicas: 4
  pgbouncer_pool: 400
queue:
  partitions: 24
  max_worker_drain_rate_per_s: 2000  # bounded by DB write capacity
waiting_room:
  admit_wave_size: 5000
  wave_interval_s: 30
```

**Flow:**

1. cache miss only
2. reads
3. miss, single-flight
4. BUY: fast reserve
5. reserved
6. sold out
7. over capacity
8. Users
9. Virtual Waiting Roomadmit in waves
10. CDN / Edge cachedeal page + assets
11. API Gatewayrate limit + load shed
12. (Redis cacheproduct + stock view)
13. (Read replicas)
14. (Redis stockatomic DECR)
15. [Durable queueKafka/SQS shock absorber]
16. Workersdrain at DB-safe rate
17. (Primary DBorders, idempotent)
18. Payment gatewayidempotency key
19. 409 fast-fail
20. 429 shed / try again

