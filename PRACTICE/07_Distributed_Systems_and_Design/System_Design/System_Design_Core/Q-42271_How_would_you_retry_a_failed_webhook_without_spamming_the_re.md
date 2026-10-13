---
type: "practice"
difficulty: "Deep"
---

# How would you retry a failed webhook without spamming the receiver?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Idempotent Order APIs](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Idempotent_Order_APIs/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage
— what is really being asked

"How would you retry a failed webhook without spamming the receiver?" This is a deep understanding follow-up to the general webhook question. The interviewer is not asking "do you retry?" — everyone does. They are probing the retry policy design: how do you distinguish transient from permanent failures, how do you space retries so a struggling or down receiver isn't hammered into the ground, how do you avoid the thundering-herd / synchronized retry storm when thousands of your deliveries all fail at once (e.g. the receiver had a 30-second outage), how long you keep trying, and how you fail gracefully. The phrase "without spamming" is the whole point: a naive retry loop turns your own reliability mechanism into an accidental DDoS against your customer.

#### Assumptions
/clarifications I'd state:

Delivery is at-least-once; receiver is idempotent (dedupes on event_id), so retrying a request the receiver actually processed is safe.
Failures come in flavors: transient (5xx, 429, timeout, connection reset) vs permanent (400 malformed, 401/403 bad auth, 410 gone). We only retry transient ones.
We control the sender; the receiver is a third party we must be polite to.

Plan: define what "spamming" means, then build the anti-spam retry policy from four levers — (1) exponential backoff, (2) jitter, (3) a cap + max retry window + DLQ, (4) receiver-aware controls (429/Retry-After, circuit breaking, concurrency limits, per-endpoint isolation).

Why naive retries spam the receiver

Two failure modes cause spam:

Tight retry loops / fixed short intervals. Retrying every 1s a hundred times floods a receiver that is already overloaded — the classic reason a struggling service never recovers.
Synchronized retries (thundering herd). Suppose the receiver is down for 30s and 50,000 of your deliveries fail at t=0. If every one retries at "t + 30s", all 50,000 hit simultaneously the instant the receiver comes back, knocking it over again. Fixed backoff without randomization synchronizes the load.

So a polite retry needs to (a) space attempts out and grow the gap, and (b) de-synchronize attempts across events.

Lever 1 — Exponential backoff

Grow the wait between attempts geometrically: delay = base * factor^attempt. E.g. base=1s, factor≈4: 1s, 4s, 16s, 64s, ~4m, ~17m, ~1h… Early transient blips recover fast; persistent outages get exponentially fewer probes, so an endpoint down for an hour receives a handful of attempts, not thousands. This directly bounds the request rate you impose on a broken receiver.

Lever 2 — Jitter (the key to "without spamming")

Backoff alone still synchronizes if all failures started together. Add randomization so retries spread over a window:

Full jitter: sleep = random(0, base*factor^attempt) — best at spreading load; AWS's recommended default.
Equal/decorrelated jitter are variants that keep a floor while still randomizing.

Full jitter turns "50,000 requests at exactly t+30s" into "50,000 requests spread randomly across a 30s window" — the receiver sees a smooth ramp instead of a spike. This is the single most important trick for not spamming.

Lever 3 — Cap, max window, and dead-letter
Cap the delay (e.g. never wait more than 6h) so events don't retry a year later, but the interval stays long.
Cap total attempts / max age (Stripe retries ~3 days, ~ dozens of attempts on a shrinking schedule; GitHub ~ a handful). After that, stop and move the delivery to a dead-letter queue, mark the endpoint unhealthy, and let the customer manually replay from a dashboard. Retrying forever is spamming — bounding it is part of politeness.
Lever 4 — Receiver-aware, adaptive controls

Backoff+jitter handle blind retrying; a great answer also listens to the receiver:

Honor Retry-After / 429. If the receiver explicitly says "back off N seconds," obey it — that's the receiver telling you its rate limit. Never retry faster than requested.
Circuit breaker per endpoint. After K consecutive failures, "open" the circuit: pause deliveries to that endpoint for a cooldown and send only a single "half-open" probe before resuming. This stops you from throwing the full backlog at a down endpoint the moment it flickers.
Per-endpoint concurrency limit + isolation. Bound in-flight requests per receiver (say ≤ 5). This caps instantaneous pressure and stops one dead customer's retry backlog from starving healthy ones (no head-of-line blocking across endpoints).
Only retry retryable failures. 4xx like 400/401/410 will fail forever — retrying them is pure spam, so fail them fast to DLQ.
Putting it together

The retry schedule is: on transient failure, compute next_retry_at = now + min(cap, base*factor^attempt) ± full_jitter, persist it, and let a scheduler pick it up. Respect Retry-After if present. Trip a per-endpoint circuit breaker on sustained failure; cap in-flight concurrency per endpoint; stop after max attempts/age → DLQ + notify + manual replay. Because delivery is idempotent on the receiver, none of these retries risk double-processing.

What a strong answer emphasizes
Exponential backoff + full jitter is the core anti-spam mechanism — backoff bounds the rate to a broken receiver, jitter de-synchronizes the thundering herd.
Classify failures: retry only transient (5xx/429/timeout), fail permanent (4xx) fast.
Bound it: cap delay, cap attempts/age, then DLQ + manual replay — retrying forever is itself spam.
Listen to the receiver: honor Retry-After/429, circuit-break per endpoint, limit concurrency, isolate endpoints.

#### Common mistakes
fixed-interval retries (synchronized storm), no jitter, retrying non-retryable 4xx, unbounded retries, and ignoring Retry-After. Follow-up probes: "why jitter and not just backoff?" (thundering herd), "what if the receiver is slow, not down?" (timeouts + concurrency cap + circuit breaker), "how does the receiver survive duplicate deliveries?" (idempotency key).

**Comparison:**

```
Lever	What it prevents	Rule of thumb
Exponential backoff	Rate-flooding a broken receiver	delay = base·factor^attempt
Full jitter	Synchronized thundering herd	sleep = rand(0, delay)
Delay cap	Retries spaced yet not absurdly late	max 6h
Attempt/age cap + DLQ	Infinite retry = spam	~3 days / ~12 tries, then DLQ + replay
Honor Retry-After / 429	Ignoring receiver's own limit	obey exact value
Circuit breaker (per endpoint)	Hammering on flaky recovery	open after K fails, half-open probe
Per-endpoint concurrency cap	Instantaneous pressure + HOL blocking	≤ ~5 in-flight/endpoint
Retry only transient failures	Wasting tries on 4xx	5xx/429/timeout=retry; 400/401/410=stop
```

**Code:**

```

import random

BASE = 1.0 # seconds FACTOR = 4.0 CAP = 6 * 3600 # 6h max delay MAX_ATTEMPTS = 12 # then dead-letter

def next_delay(attempt: int) -> float: # exponential backoff with FULL jitter -> de-synchronizes retry storm ceiling = min(CAP, BASE * (FACTOR ** attempt)) return random.uniform(0, ceiling)

def on_failure(delivery, response=None, exc=None): # 1) don't retry permanent client errors if response is not None and response.status_code in (400, 401, 403, 410): return dead_letter(delivery, reason="permanent")

CODE
Copy
# 2) respect an explicit Retry-After / 429 from the receiver
if response is not None and "Retry-After" in response.headers:
    delay = float(response.headers["Retry-After"])
else:
    delay = next_delay(delivery.attempts)

delivery.attempts += 1
if delivery.attempts >= MAX_ATTEMPTS:
    return dead_letter(delivery, reason="exhausted")  # -> notify + manual replay

schedule(delivery, run_at=now() + delay)              # persist next_retry_at
CODE
Copy

```python
# Per-endpoint circuit breaker so a down endpoint isn't hammered on recovery
class Breaker:
    def __init__(self, threshold=5, cooldown=60):
        self.fails = 0; self.open_until = 0
        self.threshold, self.cooldown = threshold, cooldown
    def allow(self):                       # gate before sending
        return now() >= self.open_until    # closed or half-open probe
    def record(self, ok):
        if ok: self.fails = 0
        else:
            self.fails += 1
            if self.fails >= self.threshold:
                self.open_until = now() + self.cooldown  # pause deliveries
```

**Example input / output:**

```text
Receiver outage from t=0 to t=30s; 3 example deliveries fail together.

WITHOUT jitter (fixed 30s backoff):
  all 3 retry at t=30.0s  -> simultaneous spike (thundering herd)

WITH exponential backoff + full jitter (attempt=0, ceiling=1s here shown for a later attempt=5, ceiling=6h):
  delivery A: next in random(0, 1024s)  -> t=+412s
  delivery B: next in random(0, 1024s)  -> t=+37s
  delivery C: next in random(0, 1024s)  -> t=+901s
  => load smeared across the window, receiver ramps up gently

Schedule for one delivery (base=1s, factor=4, cap=6h), jitter omitted for clarity:
  attempt 0 -> ~1s   | 1 -> ~4s  | 2 -> ~16s | 3 -> ~64s
  attempt 4 -> ~4m   | 5 -> ~17m | 6 -> ~1h  | 7+ -> capped 6h
  attempt 12 reached with no 2xx -> DEAD LETTER + auto-disable + email owner
```

**Data:**

```json
// delivery row driving the scheduler
{
  "event_id": "evt_01HZX8",
  "endpoint_id": "we_9fA2",
  "attempts": 3,
  "status": "pending",
  "last_code": 503,
  "next_retry_at": "2026-07-07T10:14:37Z"   // now + backoff + jitter
}
```

**Flow:**

1. 400/401/403/410permanent
2. 429 / Retry-After
3. 5xx / timeout / conn
4. no
5. yes
6. yes
7. no
8. Delivery failed
9. Status class?
10. (Dead-letter + notify)
11. delay = Retry-After
12. attempts < max& age < 3d?
13. delay = backoffbase*factor^attempt+ FULL jitter, capped
14. circuit open for endpoint?
15. wait cooldown / half-openprobe
16. schedule next_retry_atrespect per-endpointconcurrency
17. re-attempt later

