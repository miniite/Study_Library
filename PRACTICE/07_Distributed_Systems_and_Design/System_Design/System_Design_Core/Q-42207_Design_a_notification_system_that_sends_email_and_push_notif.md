---
type: "practice"
difficulty: "Intermediate"
---

# Design a notification system that sends email and push notifications.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Notification System](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Notification_System.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Design a platform notification service that reliably delivers email and push notifications (and is easily extended to SMS/WhatsApp/in-app later). The interviewer wants to see whether I treat this as a serious distributed system — not "call SendGrid in a request handler." They're probing for: decoupling with a queue, fan-out to multiple channels, template/personalization, user preferences and opt-out, idempotency/dedup, retries with backoff, rate limiting per provider, delivery tracking, and observability. At intermediate level I should present a clean architecture, justify each component, and cover the top 3–4 failure modes.

Clarifying questions I'd ask:

Volume and shape: steady-state throughput and peak (e.g., "1M push/day, bursts of 50k/min on a marketing blast")? This drives queue/partition sizing.
Latency SLA: is this transactional (OTP → must be seconds) or bulk (marketing → minutes is fine)? Usually both, so I'll support priority.
Which providers? Email via SES/SendGrid, push via APNs (iOS) + FCM (Android). Do we own device-token management?
Do we need scheduling ("send at 9am local time"), batching/digests, and quiet-hours?
Delivery guarantee: at-least-once is standard; exactly-once is impractical across third parties, so I'll aim for at-least-once + idempotent dedup.

#### Assumptions
~5M users, ~2M notifications/day, mixed transactional + marketing, multi-channel, need preferences, retries, and audit trail.

From first principles to the design
Step 1 — Decouple the caller from delivery

The core insight: a notification is a side effect triggered by a business event (order shipped, payment failed, new message). The producing service must not block on a slow third-party API (APNs, SES). So the entry point is a thin Notification API / event ingestion that validates the request, assigns a notification_id, persists it, drops it on a message queue, and returns 202 Accepted. Producers are decoupled from delivery latency and outages.

Step 2 — Normalize into an internal event, not a channel call

Producers send intent ("user X, event order_shipped, data {order_id, eta}"), not "send this email body." The service decides channels, renders templates, and respects preferences. This keeps business services dumb and centralizes policy.

Step 3 — Preference + routing layer

A Preference service answers: which channels has the user enabled for this notification category (transactional vs promotional)? Is the user in quiet hours? Are they globally unsubscribed (email) or has the push token been revoked? Transactional (OTP, security) usually ignores marketing opt-outs but still honors hard bounces/token invalidation. Output: a fan-out list like [{channel: email, addr}, {channel: push, device_tokens[]}].

Step 4 — Fan-out to per-channel workers via per-channel queues

Each channel has its own queue and worker pool (email-worker, push-worker). Reasons: providers have different rate limits, latencies, and failure semantics; a slow email provider shouldn't back up push. Workers render the template, call the provider SDK, and record the result.

Step 5 — Templating + personalization

A Template service stores versioned templates per channel/locale (e.g., Handlebars/MJML for email). Workers fetch the compiled template and interpolate event data. Versioning lets us A/B and roll back copy without a code deploy.

Step 6 — Reliability: idempotency, retries, DLQ
Idempotency: dedup on (user_id, event_key) or a client-supplied idempotency_key stored with a TTL so retried producers don't double-send.
Retries: transient provider errors (5xx, 429, timeout) → exponential backoff with jitter, capped attempts. Permanent errors (invalid token, hard bounce) → no retry; mark token dead and update preferences.
DLQ: exhausted messages go to a dead-letter queue for inspection/replay + alerting.
Step 7 — Delivery tracking + provider webhooks

Providers send async callbacks (delivered, opened, bounced, complaint). A webhook ingestion endpoint updates the notification status and feeds suppression lists. This closes the loop for analytics and deliverability hygiene.

Step 8 — Scale, priority, and rate control
Priority queues: a high (transactional/OTP) and bulk (marketing) lane so a marketing blast never starves OTPs.
Provider rate limiting: a token-bucket per provider so we stay under APNs/SES throughput caps; excess is naturally buffered in the queue.
Horizontal scale: stateless workers autoscale on queue depth (Kafka lag / SQS ApproximateNumberOfMessages).
Component summary (request flow)
CODE
Producer → Notification API (validate, persist, enqueue) → Router/Preference (fan-out)
   → per-channel queue → channel worker (render + send via provider)
   → provider (SES/APNs/FCM) → webhook → status store

Data stores: Notifications DB (status, audit — Postgres or DynamoDB keyed by notification_id), Preferences store (fast KV), Template store, suppression/token store (Redis/DB).

What a strong answer emphasizes
Async + queue is the backbone — 202 fast-return, workers absorb provider latency/outage.
Multi-channel via fan-out with per-channel queues — isolation and independent scaling.
Preferences + suppression are first-class — legal (unsubscribe) and deliverability (bounces).
At-least-once + idempotency, retries with backoff, DLQ — you will double-attempt; dedup keeps it invisible.
Priority lanes so transactional never waits behind bulk.
Observability: per-provider success rate, queue lag, p99 send latency, bounce/complaint rate.

#### Common mistakes
/ follow-up probes
Sending inline in the request path (blocks the user, cascades on provider outage).
Ignoring device-token lifecycle → APNs "Unregistered" errors pile up.
No dedup → users get 3 copies after a producer retry.
Treating exactly-once as achievable across third parties (it isn't; do idempotent at-least-once).

#### Follow-ups
often go to: how to send at "9am in user's local timezone" (schedule store + timezone shard), digest/batching, and multi-region failover of the queue.

**Comparison:**

```
Concern	Transactional (OTP, order)	Promotional (marketing)
Priority lane	high	bulk
Respects marketing opt-out	No (still honors hard bounces/dead tokens)	Yes
Quiet hours	No	Yes
Latency SLA	seconds	minutes
Retry policy	aggressive, capped	limited, drop on heavy failure
Delivery guarantee	Feasible?	Approach
At-most-once	Yes	fire-and-forget (risk: lost notifications)
Exactly-once	Not across 3rd parties	approximate via idempotency key + dedup
At-least-once	Yes (recommended)	queue redelivery + idempotent workers
```

**Code:**

```python
# Push worker: at-least-once send with backoff, dedup, and dead-token handling.
import time, random

def handle_push(msg, store, fcm):
    idem = msg["idempotency_key"]
    if store.seen(idem):                 # dedup: already delivered/attempted terminally
        return "duplicate_skip"

    for attempt in range(1, 6):          # max 5 attempts
        try:
            resp = fcm.send(token=msg["device_token"],
                            title=msg["title"], body=msg["body"],
                            data=msg["data"])
            store.mark(idem, status="SENT", provider_id=resp.message_id)
            return "sent"
        except FcmUnregistered:          # permanent: token dead -> stop, suppress
            store.invalidate_token(msg["device_token"])
            store.mark(idem, status="FAILED_TOKEN")
            return "dead_token"
        except (FcmRateLimited, FcmServerError, TimeoutError):  # transient -> retry
            backoff = min(2 ** attempt, 30) + random.uniform(0, 0.5)  # jitter
            time.sleep(backoff)
    store.mark(idem, status="FAILED")    # exhausted -> DLQ
    raise SendToDLQ(msg)
```

**Example input / output:**

```

Worked example — one intent fanning out to two channels:

Input (POST /notifications):

CODE
Copy
{ "event": "order_shipped", "user_id": "u_10293",
  "idempotency_key": "order_shipped:o_88123", "priority": "high",
  "data": { "order_id": "o_88123", "eta": "2026-07-09" } }

Immediate response:

CODE
Copy
HTTP 202 Accepted
{ "notification_id": "n_5f3a", "status": "QUEUED" }

Processing trace:

CODE
Copy
1. API persists n_5f3a, enqueues to router.high
2. Router reads prefs: transactional -> email:true, push:true  -> fan-out 2 msgs
3. email-worker: render template v3 (en) -> SES accept -> status EMAIL:SENT
   webhook later -> EMAIL:DELIVERED
4. push-worker: FCM returns "Unregistered" -> invalidate token, status PUSH:FAILED_TOKEN (no retry)
5. Producer accidentally re-POSTs same idempotency_key -> dedup -> no second send

Final state: email delivered, push suppressed, exactly one attempt per channel despite duplicate producer call.
```

**Data:**

```

// Producer -> Notification API (intent, not channel-specific body) { "event": "order_shipped", "user_id": "u_10293", "idempotency_key": "order_shipped:o_88123", "priority": "high", "data": { "order_id": "o_88123", "eta": "2026-07-09", "tracking_url": "https://t/abc" } }

CODE
Copy
```json
// Notification record (status store)
{
  "notification_id": "n_5f3a",
  "user_id": "u_10293",
  "category": "transactional",
  "channels": [
    { "channel": "email", "status": "DELIVERED", "provider": "ses", "attempts": 1 },
    { "channel": "push",  "status": "FAILED_TOKEN", "provider": "fcm", "attempts": 1 }
  ],
  "created_at": "2026-07-07T10:12:00Z"
}
YAML
Copy
# User preference document
user_id: u_10293
timezone: Asia/Kolkata
channels:
  transactional: { email: true,  push: true,  sms: true }
  promotional:   { email: false, push: true,  sms: false }
quiet_hours: { start: "22:00", end: "08:00" }   # applies to promotional only
```

**Flow:**

1. POST intent, 202
2. webhook
3. webhook
4. exhausted
5. exhausted
6. Producer service
7. Notification API
8. (Notifications/status DB)
9. Priority routerhigh / bulk
10. Preference + suppression
11. [email queue]
12. [push queue]
13. email workers
14. push workers
15. (Templates)
16. SES/SendGrid
17. APNs/FCM
18. Webhook ingest
19. [DLQ]

