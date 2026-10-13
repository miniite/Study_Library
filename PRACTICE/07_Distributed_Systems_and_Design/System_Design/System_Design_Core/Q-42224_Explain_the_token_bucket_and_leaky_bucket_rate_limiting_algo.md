---
type: "practice"
difficulty: "Deep"
---

# Explain the token bucket and leaky bucket rate limiting algorithms.

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Rate Limiter Storage](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Rate_Limiter_Storage/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

This is a deep-understanding question aimed at practitioners/architects: "Explain the token bucket and leaky bucket rate limiting algorithms." Given the company set (Stripe, Google, Cloudflare, Razorpay, CRED), the interviewer wants more than definitions. They're probing whether I truly understand the mechanics, the difference in the traffic shape each produces, when to pick one over the other, and how each is implemented correctly (especially atomically, in a distributed system). A weak answer conflates the two; a strong answer nails the key distinction: token bucket controls the input and permits bursts up to a cap; leaky bucket controls the output and enforces a smooth, constant rate.

Plan: define each algorithm precisely with its state and update rule → work a concrete numeric trace of each → contrast the traffic they produce → give correct implementation notes (atomicity, distributed state) → say when to use which → summarize with common pitfalls.

Token bucket

Mental model: a bucket holds up to B tokens (the capacity / burst size). Tokens are added at a steady refill rate r tokens/second, up to the cap B (overflow tokens are discarded). Each incoming request must remove one token (or cost tokens) to proceed. If there's a token, the request is allowed and a token is consumed; if the bucket is empty, the request is rejected (or queued).

State: (tokens, last_refill_timestamp). You don't need a background timer; you compute refill lazily on each request:

CODE
elapsed = now - last_refill
tokens  = min(B, tokens + elapsed * r)
last_refill = now
if tokens >= cost: tokens -= cost -> ALLOW
else -> REJECT (retry_after = (cost - tokens) / r)

Key property: it permits bursts. If a client is idle, the bucket refills to B; then the client can fire up to B requests instantly, after which it's throttled to the sustained rate r. This matches real API clients that batch a few calls then idle. Average long-run rate is bounded by r; short-term burst is bounded by B. So a limit like "100 req burst, 10 req/sec sustained" is B=100, r=10.

Leaky bucket

Mental model: think of a bucket with a hole in the bottom that leaks at a constant rate (the leak/output rate). Incoming requests are water poured in; they queue in the bucket (up to its capacity). Requests drain out at a fixed constant rate regardless of how bursty the arrivals are. If the bucket overflows (queue full), new requests are dropped.

There are two flavors:

Leaky bucket as a queue (meter/shaper): requests wait and are released at exactly the output rate → produces a perfectly smooth, constant output stream. Adds latency (queuing) but protects a fragile downstream.
Leaky bucket as a counter: implemented like a decreasing counter that "leaks" over time; functionally similar to a token-bucket-style admission check but conceptually focused on the output rate.

State: the queue (or a level that leaks over time) and the constant drain rate. The defining behavior is that output is always steady; bursts are absorbed and smoothed, not passed through.

The crucial difference
Token bucket allows bursty output up to B, then limits the average. It controls admission; it does not smooth traffic. Rejected requests are dropped immediately (you get a fast 429).
Leaky bucket produces smooth output at a constant rate. It controls the egress shape; bursts are queued and metered out (adding delay) or dropped on overflow. It's a traffic shaper.

Put differently: token bucket answers "have you exceeded your allowance?" (and lets you spend it in bursts). Leaky bucket answers "I will only emit at rate X no matter how you arrive" (and smooths spikes into a steady stream).

#### Implementation
notes (what an interviewer digs into)
Atomicity in a distributed fleet. State (tokens/queue level) must live in a shared store (Redis) and be updated atomically, or concurrent requests double-spend. Use a Redis Lua script so refill+consume is one atomic step. Per-server in-memory counters are wrong across a load-balanced fleet.
Lazy refill vs background thread. Token bucket's lazy computation (elapsed × r) avoids a timer and is the standard approach. Leaky-bucket-as-queue needs an actual dequeue mechanism at the drain rate (or the lazy "leak = elapsed × rate" trick for the counter form).
Time source. Use a single authoritative clock (Redis TIME) to avoid per-node drift.
Overflow/rejection behavior. Token bucket → reject with 429 + Retry-After = (cost - tokens)/r. Leaky bucket → either queue (adds latency, bounded by capacity/rate) or drop on full queue.
Cost weighting. Both can charge more tokens for expensive endpoints (e.g., a payment costs 5 tokens, a health check costs 0).
When to use which
Choose token bucket for public/user-facing APIs where short bursts are legitimate and desirable and you want to reject excess quickly (Stripe/Razorpay-style API keys with "burst then sustained"). It's the default for most API rate limiting.
Choose leaky bucket when you must protect a downstream that can only handle a steady rate — smoothing spiky ingress into a constant, predictable outflow (e.g., writing to a legacy system, dialing an external provider with a strict fixed QPS, or shaping network traffic). You accept added latency to guarantee smoothness.
In practice many limiters (and even the classic "GCRA" used by some CDNs) are refinements/equivalents of these two; sliding-window counters are a cheaper cousin of token bucket for pure admission control.
Summary — what a strong answer emphasizes

The one-sentence discriminator: token bucket = burst-tolerant admission control (average bounded by r, burst bounded by B); leaky bucket = constant-rate output shaping (smooths/queues bursts, drops on overflow). Nail the state and update rule for each, show a numeric trace, and cover atomic distributed implementation (Redis + Lua), the time source, and the reject-vs-queue behavior.

#### Common mistakes
(1) claiming they're the same — they aren't; token bucket lets bursts through, leaky bucket does not. (2) Non-atomic read-then-write causing over-admission. (3) Per-server memory in a distributed system. (4) Forgetting overflow behavior (429/Retry-After vs queue-and-drop). Follow-up probes: "Which would you use in front of a payments provider that allows exactly 50 QPS?" → leaky bucket (smooth, never exceed). "Which for a user-facing REST API?" → token bucket (allow bursts, cap average).

**Comparison:**

```
Property	Token bucket	Leaky bucket
What it controls	Admission (input)	Output shape (egress)
Allows bursts?	Yes, up to capacity B	No — bursts are smoothed/queued
Output pattern	Bursty then avg = r	Constant at leak rate
On excess	Reject fast (429 + Retry-After)	Queue (adds latency) or drop on overflow
Adds latency?	No	Yes (queuing form)
State	tokens + last-refill ts	queue level + last-leak ts
Best for	User/public APIs, burst-friendly	Protecting a strict fixed-rate downstream
Typical impl	Lazy refill, Redis+Lua atomic	Queue drained at fixed rate / leaking counter
```

**Code:**

```python
import time

class TokenBucket:
    """Burst up to `capacity`, sustained rate `refill` tokens/sec. Lazy refill."""
    def __init__(self, capacity, refill):
        self.capacity = capacity
        self.refill = refill               # tokens per second
        self.tokens = capacity             # start full
        self.ts = time.monotonic()

    def allow(self, cost=1):
        now = time.monotonic()
        # lazily add tokens for elapsed time, capped at capacity
        self.tokens = min(self.capacity,
                          self.tokens + (now - self.ts) * self.refill)
        self.ts = now
        if self.tokens >= cost:
            self.tokens -= cost
            return True, self.tokens, 0
        retry_after = (cost - self.tokens) / self.refill
        return False, self.tokens, retry_after

class LeakyBucketQueue:
    """Constant-output shaper: enqueue if room; drain at fixed `leak_rate`."""
    def __init__(self, capacity, leak_rate):
        self.capacity = capacity           # max queued requests
        self.leak_rate = leak_rate         # requests emitted per second
        self.level = 0.0                   # current queued volume
        self.ts = time.monotonic()

    def admit(self):
        now = time.monotonic()
        # leak out volume for elapsed time (constant drain)
        self.level = max(0.0, self.level - (now - self.ts) * self.leak_rate)
        self.ts = now
        if self.level + 1 <= self.capacity:
            self.level += 1                # accepted, will drain at leak_rate
            return True
        return False                       # overflow -> drop
```

**Example input / output:**

```text
Token bucket dry-run — `capacity=5`, `refill=1/s`, client idle then bursts:

| t (s) | tokens (after refill) | request | result | tokens after |
|---|---|---|---|---|
| 0 | 5 (full) | 5 rapid reqs | all ALLOW (burst) | 0 |
| 0 | 0 | 6th req | REJECT, retry_after=1s | 0 |
| 2 | 2 (refilled +2) | 2 reqs | both ALLOW | 0 |

=> Output is **bursty**: 5 requests fired instantly, then throttled to 1/s.

Leaky bucket dry-run — `capacity=5`, `leak_rate=1/s`, same 5 rapid arrivals at t=0:

| t (s) | level (after leak) | arrival | result | emitted so far |
|---|---|---|---|---|
| 0 | 0 | 5 arrive | all queued (level=5, full) | 0 |
| 0 | 5 | 6th arrival | DROP (overflow) | 0 |
| 1 | 4 | — | 1 emitted | 1 |
| 2 | 3 | — | 1 emitted | 2 |
| 5 | 0 | — | all emitted | 5 |

=> Output is **smooth**: exactly 1 request/sec emitted regardless of the burst arrival.
```

**Flow:**

1. Leaky Bucket (constant-rate shaper)
2. no
3. yes
4. requests arrive bursty
5. queue full?
6. enqueue
7. drop overflow
8. drain at fixed leak rate
9. smooth constant output
10. Token Bucket (burst-tolerant admission)
11. yes
12. no
13. refill r tokens/s up to cap B
14. (bucket: tokens)
15. request
16. tokens >= cost?
17. ALLOW, consume token
18. REJECT 429 + Retry-After

