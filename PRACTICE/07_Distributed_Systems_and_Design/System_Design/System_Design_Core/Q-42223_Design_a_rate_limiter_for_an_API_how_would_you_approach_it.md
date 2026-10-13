---
type: "practice"
difficulty: "Intermediate"
---

# Design a rate limiter for an API — how would you approach it?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Rate Limiter Storage](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Rate_Limiter_Storage/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

This is an intermediate, practitioner system-design question: "Design a rate limiter for an API — how would you approach it." Given the company set (Stripe, Razorpay, PhonePe, Cloudflare, Google), the interviewer is probing whether I can (1) clarify the requirements before jumping to an algorithm, (2) pick an appropriate algorithm and justify it, (3) reason about where the limiter lives and how it stays correct across many app servers, and (4) handle real-world concerns: response behavior, distributed state, atomicity, failure modes, and configuration. They are not looking for a single "right" answer — they want a structured design with trade-offs.

Clarifying questions I'd ask first (and my assumptions):

What are we protecting against? Abuse/DDoS, fair usage across tenants, or protecting a downstream (a payments provider)? → assume fair usage + abuse protection.
What's the limit dimension? Per API key, per user, per IP, per endpoint, or a combination? → assume per API key, with per-endpoint overrides.
Scale? → assume tens of thousands of RPS across a fleet of stateless app servers behind a load balancer, so state must be shared.
Do we need hard limits or graceful throttling? → hard limit returning 429, with Retry-After.
Single region or global? → start single-region, note the multi-region wrinkle at the end.
Step 1 — Where does the limiter live?

There are three common placements:

At the edge / API gateway (Cloudflare, Kong, Envoy, AWS API Gateway). Best for coarse, IP/key-level protection; keeps bad traffic off your servers entirely.
In application middleware (a per-request check before your handler). Best when limits are tied to business logic (plan tier, per-endpoint cost).
As a dedicated rate-limit service that the gateway/app calls. Best at large scale and for consistency across many services.

A good design often does both: cheap coarse limiting at the edge, plus precise, business-aware limiting in the app using a shared store. I'll design the app-level limiter with a shared Redis store, which is the industry-standard choice.

Step 2 — Pick an algorithm

The main candidates: fixed window, sliding window log, sliding window counter, token bucket, leaky bucket. For an API I'd choose token bucket (or sliding-window counter) because it allows bursts up to a cap while enforcing a steady average rate — which matches real API usage (clients send a few requests together, then idle). Key trade-offs are in the table.

Fixed window (count per calendar minute) is trivial but has a boundary burst flaw: a client can send N at 0:59 and N at 1:00 — 2N in two seconds.
Sliding window counter fixes most of that by weighting the previous window; it's cheap and accurate enough — a very common production pick.
Token bucket naturally models "average rate + burst allowance" and is what I'd default to for a public API.
Step 3 — The data model and atomicity

For a distributed fleet, the limiter state (tokens remaining, last-refill timestamp) must live in a shared, fast store — Redis. The critical requirement is atomicity: read-modify-write of the counter must be a single atomic operation, or two concurrent requests both read "1 token left," both decrement, and you overspend. Solve this with a Redis Lua script (executed atomically) or INCR+EXPIRE for window counters. Lua also lets me do the token refill math server-side in one round trip.

Key design: rl:{apiKey}:{endpoint} as the Redis key, with a TTL so idle keys expire and memory stays bounded.

Step 4 — Response behavior (this is what separates good from great)
Return HTTP 429 Too Many Requests when the limit is exceeded.
Include headers so well-behaved clients self-throttle: X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset, and Retry-After on a 429.
Make the check fail-open or fail-closed deliberately. If Redis is down, do you block all traffic (fail-closed, safest for a payments backend) or allow it through (fail-open, safest for availability)? State the choice; I'd fail-open with alerting for a general API, fail-closed for a limiter protecting a fragile downstream.
Step 5 — Correctness and operational concerns
Time source: use the store's clock (Redis TIME) or a monotonic source; don't trust per-server clocks that drift.
Hot keys: a single huge tenant can create a hot Redis key. Mitigate with local pre-check + sharding, or per-node token leasing.
Configurability: limits should be data-driven (per plan tier, per endpoint), reloadable without a deploy.
Multi-region: global limits across regions are expensive (cross-region consensus). Common pragmatic answer: enforce per-region limits (limit/regions) or use an eventually-consistent global counter — and say you're accepting slight over-admission for latency.
Step 6 — Reference implementation (token bucket in Redis via Lua)

The Lua script (in code_blocks) atomically: computes how many tokens to refill based on elapsed time, caps at bucket size, and either consumes one token (allow) or rejects (429). One network round trip, race-free.

Summary — what a strong answer emphasizes
Clarify first: what dimension (key/IP/user), edge vs app, hard vs soft.
Algorithm with justification: token bucket for burst+average; sliding-window counter as a cheaper alternative; call out the fixed-window boundary bug.
Shared atomic state in Redis via a Lua script — atomicity is the crux in a distributed fleet.
Client-friendly responses: 429 + Retry-After + rate-limit headers.
Failure modes: fail-open vs fail-closed, hot keys, clock source, TTLs, and the multi-region trade-off.

#### Common mistakes
to avoid: using a non-atomic read-then-write (overspends under concurrency); using fixed windows and ignoring the boundary burst; keeping counters in per-server memory (inconsistent across the fleet); and forgetting Retry-After. Follow-up probes usually go into "how do you make it atomic," "what if Redis dies," and "how do you do this globally" — all covered above.

**Comparison:**

```
Algorithm	Allows bursts?	Boundary-burst bug?	Memory / cost	Accuracy	Best for
Fixed window	Yes (at edges)	Yes (2N over boundary)	Very low (1 counter)	Low	Simplest, rough limits
Sliding window log	No	No	High (stores each timestamp)	Highest	Small volumes, exactness
Sliding window counter	Slight	Mostly fixed	Low	Good	General API (cheap + accurate)
Token bucket	Yes, up to cap	No	Low (tokens + ts)	Good	Public API, burst + avg rate
Leaky bucket	No (smooths output)	No	Low	Good	Shaping steady downstream load
```

**Code:**

```

-- token_bucket.lua (Redis Lua: atomic, race-free) -- KEYS[1] = rate-limit key, e.g. rl:{apiKey}:{endpoint} -- ARGV[1] = bucket capacity (max tokens / burst) -- ARGV[2] = refill rate (tokens per second) -- ARGV[3] = current time (seconds, from redis TIME) -- ARGV[4] = tokens requested (usually 1) -- returns: {allowed(1/0), tokens_remaining, retry_after_seconds}

local capacity = tonumber(ARGV[1]) local rate = tonumber(ARGV[2]) local now = tonumber(ARGV[3]) local requested = tonumber(ARGV[4])

local state = redis.call('HMGET', KEYS[1], 'tokens', 'ts') local tokens = tonumber(state[1]) local ts = tonumber(state[2])

if tokens == nil then -- first time: full bucket tokens = capacity ts = now end

-- refill based on elapsed time, capped at capacity local delta = math.max(0, now - ts) tokens = math.min(capacity, tokens + delta * rate)

local allowed = 0 local retry_after = 0 if tokens >= requested then tokens = tokens - requested allowed = 1 else retry_after = math.ceil((requested - tokens) / rate) end

redis.call('HMSET', KEYS[1], 'tokens', tokens, 'ts', now) redis.call('EXPIRE', KEYS[1], math.ceil(capacity / rate) + 1) -- idle keys expire return {allowed, tokens, retry_after}

CODE
Copy

```javascript
// Express middleware calling the Lua script (Node + ioredis)
const LIMIT = 100;      // burst capacity
const RATE  = 10;       // tokens/sec => sustained 10 rps

function rateLimit(redis, sha) {
  return async (req, res, next) => {
    const key = `rl:${req.apiKey}:${req.route.path}`;
    const now = Date.now() / 1000;
    let allowed, remaining, retry;
    try {
      [allowed, remaining, retry] =
        await redis.evalsha(sha, 1, key, LIMIT, RATE, now, 1);
    } catch (e) {
      // Redis down: FAIL-OPEN for a general API (log + alert). For a payments
      // limiter protecting a fragile downstream, prefer fail-closed (return 429).
      console.error('rate-limiter store error, failing open', e);
      return next();
    }
    res.set('X-RateLimit-Limit', LIMIT);
    res.set('X-RateLimit-Remaining', Math.floor(remaining));
    if (allowed) return next();
    res.set('Retry-After', retry);
    return res.status(429).json({ error: 'rate_limited', retry_after: retry });
  };
}
```

**Example input / output:**

```text
Dry run — `capacity=5`, `rate=1 token/sec`, requests of 1 token each:

| t (s) | tokens before | event | allowed? | tokens after | Retry-After |
|---|---|---|---|---|---|
| 0.0 | 5 (full) | req | yes (200) | 4 | — |
| 0.0 | 4 | req | yes | 3 | — |
| 0.0 | 3 | req | yes | 2 | — |
| 0.0 | 2 | req | yes | 1 | — |
| 0.0 | 1 | req | yes | 0 | — |
| 0.0 | 0 | req | **no (429)** | 0 | 1 |
| 3.0 | 0 → refill +3 = 3 | req | yes | 2 | — |

Observed HTTP:
- Request 6 → `429 Too Many Requests`, `Retry-After: 1`, `X-RateLimit-Remaining: 0`.
- Request 7 (after 3s idle) → `200 OK`, `X-RateLimit-Remaining: 2` (bucket refilled by elapsed time).
```

**Data:**

```yaml
# Data-driven limits: reloadable config, per plan tier and per endpoint override
tiers:
  free:    { capacity: 60,   refill_per_sec: 1 }
  pro:     { capacity: 600,  refill_per_sec: 10 }
  enterprise: { capacity: 6000, refill_per_sec: 100 }
endpoint_overrides:
  "POST /v1/payments":   { capacity: 20, refill_per_sec: 2 }   # expensive, tighter
  "GET  /v1/health":     { unlimited: true }
```

**Flow:**

1. blocked
2. pass
3. Redis error
4. yes
5. no
6. allowed=1
7. allowed=0
8. Incoming API request
9. Edge gatewaycoarse IP/key limit
10. 429 at edge
11. App middleware
12. EVALSHA token_bucket.lua onRedisatomic refill + consume
13. fail-open?
14. handler runs 200
15. 429 + Retry-After + headers

