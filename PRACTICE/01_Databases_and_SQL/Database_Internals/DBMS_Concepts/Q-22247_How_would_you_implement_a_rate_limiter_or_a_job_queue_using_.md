---
type: "practice"
difficulty: "Expert"
---

# How would you implement a rate limiter or a job queue using only a relational database?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1 · [↑ Lock Granularity](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Lock_Granularity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Design two classic infrastructure primitives — a rate limiter and a job queue — using only a relational database (Postgres/MySQL), with no Redis, no SQS, no Kafka. The interviewer wants to see that you can lean on SQL's real superpowers — transactions, row locks, atomic UPSERTs, and SELECT ... FOR UPDATE SKIP LOCKED — to get correctness under concurrency, and that you understand the limits (contention, throughput, polling cost) versus a purpose-built system.

What they're probing (LEVEL = expertise): correctness under concurrent workers (no double-dispatch, no lost jobs), atomicity of the "check-and-decrement" for the limiter, avoiding lock convoys / the "thundering herd" of workers, visibility timeouts and retries, cleanup, and honest trade-offs. Naming SKIP LOCKED unprompted is the signal they're looking for.

Clarifying questions I'd ask: What throughput (100/s vs 100k/s)? Is best-effort limiting OK or must it be exact? Single DB or sharded? Do jobs need ordering, priority, delays, dedup? These decide how far the RDB approach stretches. I'll assume moderate scale (up to a few thousand ops/sec) where "just the database" is genuinely the right, operationally-simple choice.

Plan: (1) Rate limiter — fixed window via atomic UPSERT, then sliding-window and token-bucket in SQL, with the concurrency guarantee. (2) Job queue — schema, the FOR UPDATE SKIP LOCKED claim, visibility timeout/retries/DLQ, delays & priority. (3) Trade-offs vs Redis/SQS and when to graduate off the DB.

Part 1 — Rate limiter in a relational DB

Goal: allow at most N actions per key (user/IP/API-key) per time window, correctly even when many requests for the same key hit different app servers simultaneously.

1a. Fixed-window counter (simplest, atomic)

Bucket time into windows (e.g., 60s) and key by (subject, window_start). The entire check-and-increment must be one atomic statement — never SELECT then UPDATE in app code (that's a race). Use an atomic UPSERT:

SQL
INSERT INTO rate_limit (subject, window_start, count)
VALUES ('user:42', date_trunc('minute', now()), 1)
ON CONFLICT (subject, window_start)
DO UPDATE SET count = rate_limit.count + 1
RETURNING count;

The DB serializes concurrent UPSERTs on the same row via its row lock, so the returned count is exact. The app allows the request iff count <= N. This is a single round-trip and cheap. Downside: the boundary burst — a client can do N at the end of one window and N at the start of the next (2N in a short span).

1b. Sliding window (smoother)

Keep a log of request timestamps (or per-sub-bucket counts) and count only those within the trailing window:

SQL
-- count requests in the last 60s, then insert if under limit (in one txn)
WITH recent AS (
  SELECT count(*) AS c FROM rate_events
  WHERE subject = 'user:42' AND ts > now() - interval '60 seconds'
)
INSERT INTO rate_events (subject, ts)
SELECT 'user:42', now() FROM recent WHERE c < 100
RETURNING id;   -- rows returned => allowed; empty => rejected

Wrap in a transaction (or accept a tiny slack) and periodically delete old rows. More accurate, more storage/IO.

1c. Token bucket (allows bursts, enforces average rate)

Store tokens and last_refill. On each request, atomically refill based on elapsed time, then try to spend a token — all in one UPDATE:

SQL
UPDATE token_bucket
SET tokens = LEAST(capacity,
                   tokens + EXTRACT(EPOCH FROM (now() - last_refill)) * refill_per_sec) - 1,
    last_refill = now()
WHERE subject = 'user:42'
  AND LEAST(capacity, tokens + EXTRACT(EPOCH FROM (now() - last_refill)) * refill_per_sec) >= 1
RETURNING tokens;   -- row returned => allowed

The WHERE guard makes the whole thing atomic: it only decrements if a token is actually available; empty result => rate-limited.

Concurrency guarantee (all three): correctness comes from the single atomic statement on one row — the row lock serializes same-key contention. Different keys don't contend. The cost is that a very hot key becomes a lock hotspot (see trade-offs).

Part 2 — Job queue in a relational DB

Goal: producers INSERT jobs; multiple competing workers each grab a different job, process it, and mark it done — with no job dispatched twice, no job lost if a worker dies, plus retries and delays.

2a. Schema

jobs(id, queue, payload, status, run_at, attempts, max_attempts, locked_until, created_at) with status ∈ {ready, processing, done, failed}. Index on (queue, status, run_at).

2b. The claim — SELECT ... FOR UPDATE SKIP LOCKED

The heart of the design. A worker atomically finds one ready job, locks it, and marks it processing — while other workers skip locked rows instead of blocking:

SQL
UPDATE jobs SET status='processing', locked_until = now() + interval '5 min',
                attempts = attempts + 1
WHERE id = (
  SELECT id FROM jobs
  WHERE queue = 'emails' AND status = 'ready' AND run_at <= now()
  ORDER BY run_at
  FOR UPDATE SKIP LOCKED       -- <-- concurrency-safe fan-out
  LIMIT 1
)
RETURNING id, payload;

FOR UPDATE locks the chosen row; SKIP LOCKED makes concurrent workers ignore rows already locked by a peer and pick the next free one. Result: N workers claim N distinct jobs with no blocking and no double-dispatch — this is exactly why it's the canonical answer. (MySQL 8.0+ and Postgres 9.5+ both support it.) On completion the worker runs UPDATE jobs SET status='done' WHERE id=$1.

2c. Crash safety — visibility timeout + retries + DLQ

If a worker dies mid-job, the job would be stuck in processing forever. Guard with locked_until: a claim only considers status='ready' OR (status='processing' AND locked_until < now()) — so an expired lease is re-claimable. A sweeper (or the claim query itself) resets timed-out jobs. On failure, if attempts < max_attempts set it back to ready with backoff (run_at = now() + interval ...); else move to a dead-letter state (failed) for inspection. This gives at-least-once delivery; make handlers idempotent.

2d. Delayed & priority jobs
Delay/schedule: set run_at in the future; the run_at <= now() predicate naturally hides it until due.
Priority: add a priority column and ORDER BY priority DESC, run_at.
2e. Reducing the polling cost

Workers poll (SELECT every X ms), which wastes queries when idle. Postgres offers LISTEN/NOTIFY: the producer NOTIFY new_job on insert and workers LISTEN to wake immediately, eliminating busy-poll latency while keeping the DB as the source of truth. Also VACUUM/partition-drop the done rows or they bloat the table (a real operational gotcha in Postgres MVCC).

Part 3 — Trade-offs and when to graduate

The DB approach wins on operational simplicity, transactional coupling, and exactly-once-with-your-data semantics: you can enqueue a job in the same transaction as the business write, so a job is created iff the order is created — impossible to do atomically with an external queue. It's ideal up to a few thousand jobs/sec.

It loses at high throughput: every claim is a write + VACUUM pressure; a single hot rate-limit key serializes on one row; polling adds load; the table bloats. Redis (INCR/EXPIRE, sorted sets) does rate limiting at 100k+/s in memory; SQS/Kafka give near-infinite queue throughput and built-in retries/DLQ. The engineering judgment: start with the database (fewer moving parts, transactional integrity), and move the hot path to Redis/SQS only when measured contention or throughput demands it — often keeping the DB as the durable record.

Summary — what a strong answer emphasizes
Rate limiter correctness = one atomic statement on one row (UPSERT / guarded UPDATE); fixed-window is simplest, token-bucket allows bursts, sliding-window is smoothest.
Job queue correctness = SELECT ... FOR UPDATE SKIP LOCKED for lock-free fan-out, plus visibility-timeout leases + retries + DLQ + idempotent handlers for crash safety; run_at for delay, priority for ordering, LISTEN/NOTIFY to avoid busy-poll.
Killer advantage: enqueue inside the business transaction. Killer limit: contention/bloat/throughput — graduate to Redis/SQS when the numbers say so.

#### Common mistakes
SELECT-then-UPDATE in app code (race → double dispatch / over-limit); plain FOR UPDATE without SKIP LOCKED (workers block in a convoy); forgetting the visibility timeout (dead workers strand jobs); ignoring MVCC bloat from churned rows; and claiming exactly-once (it's at-least-once → require idempotency). Follow-ups: how SKIP LOCKED interacts with ORDER BY, fairness under starvation, and sharding a hot rate-limit key.

**Comparison:**

```
Concern	Relational DB approach	Redis / SQS
Rate limiter	Atomic UPSERT / guarded UPDATE (1 row)	INCR+EXPIRE / sorted set (in-mem)
Queue fan-out	FOR UPDATE SKIP LOCKED	native pop / visibility timeout
Throughput ceiling	~1k–few-k ops/s (write + bloat)	100k+/s
Transactional with business write	Yes (same txn) — big win	No
Crash-safe retries	visibility timeout + attempts + DLQ	built-in
Ops complexity	Low (one system)	Extra infra
Hot-key contention	Single-row lock hotspot	Handled in memory
Rate-limit algorithm	Burst behavior	Accuracy	Cost
Fixed window	2N boundary burst	Coarse	1 UPSERT
Sliding window	Smooth	High	timestamp log + cleanup
Token bucket	Controlled burst to capacity	High	1 UPDATE
```

**Code:**

```

-- ===== RATE LIMITER: fixed window (atomic UPSERT) ===== CREATE TABLE rate_limit ( subject text, window_start timestamptz, count int NOT NULL, PRIMARY KEY (subject, window_start) );

INSERT INTO rate_limit (subject, window_start, count) VALUES ('user:42', date_trunc('minute', now()), 1) ON CONFLICT (subject, window_start) DO UPDATE SET count = rate_limit.count + 1 RETURNING count; -- app allows iff count <= LIMIT

CODE
Copy

```sql
-- ===== RATE LIMITER: token bucket (single atomic UPDATE) =====
UPDATE token_bucket
SET tokens = LEAST(capacity,
              tokens + EXTRACT(EPOCH FROM (now()-last_refill))*refill_per_sec) - 1,
    last_refill = now()
WHERE subject = 'user:42'
  AND LEAST(capacity,
        tokens + EXTRACT(EPOCH FROM (now()-last_refill))*refill_per_sec) >= 1
RETURNING tokens;                -- row => allowed; empty => limited
SQL
Copy
-- ===== JOB QUEUE: schema + claim + complete/fail =====
CREATE TABLE jobs (
  id           bigserial PRIMARY KEY,
  queue        text        NOT NULL,
  payload      jsonb       NOT NULL,
  status       text        NOT NULL DEFAULT 'ready',   -- ready|processing|done|failed
  priority     int         NOT NULL DEFAULT 0,
  run_at       timestamptz NOT NULL DEFAULT now(),
  locked_until timestamptz,
  attempts     int         NOT NULL DEFAULT 0,
  max_attempts int         NOT NULL DEFAULT 5
);
CREATE INDEX ON jobs (queue, status, run_at);

-- CLAIM one job, skipping peers' locked rows (also reclaims expired leases)
UPDATE jobs SET status='processing',
                locked_until = now() + interval '5 minutes',
                attempts = attempts + 1
WHERE id = (
  SELECT id FROM jobs
  WHERE queue = 'emails'
    AND (status = 'ready'
         OR (status = 'processing' AND locked_until < now()))
    AND run_at <= now()
  ORDER BY priority DESC, run_at
  FOR UPDATE SKIP LOCKED
  LIMIT 1
)
RETURNING id, payload;

-- SUCCESS
UPDATE jobs SET status='done' WHERE id = $1;

-- FAILURE: retry with backoff, or dead-letter
UPDATE jobs
SET status = CASE WHEN attempts >= max_attempts THEN 'failed' ELSE 'ready' END,
    run_at = now() + (interval '10 seconds' * attempts),  -- backoff
    locked_until = NULL
WHERE id = $1;
```

**Example input / output:**

```text
RATE LIMITER (limit = 3 / minute, key user:42, all in same window):
  req1 -> UPSERT returns count=1  -> ALLOW
  req2 -> UPSERT returns count=2  -> ALLOW
  req3 -> UPSERT returns count=3  -> ALLOW
  req4 -> UPSERT returns count=4  -> REJECT (429)
  (two concurrent reqs still get distinct counts: DB serializes the row)

JOB QUEUE (3 workers, 3 ready jobs #101 #102 #103, run simultaneously):
  W1 CLAIM -> locks #101, others SKIP it -> gets 101
  W2 CLAIM -> #101 locked, skipped -> gets 102
  W3 CLAIM -> 101/102 locked -> gets 103
  => 3 distinct jobs, zero double-dispatch, zero blocking.

CRASH CASE:
  W2 claims #102 (locked_until=10:05), then W2 dies.
  At 10:05 next CLAIM sees status=processing AND locked_until<now
  -> reclaims #102 -> reprocessed (at-least-once; handler idempotent).
```

**Data:**

```text
jobs row lifecycle:
  ready --claim--> processing --ok--> done
                        |--crash/timeout (locked_until<now)--> ready (reclaimed)
                        |--error, attempts<max--> ready (backoff)
                        |--error, attempts>=max--> failed (dead-letter)

Sample job payload:
{ "id": 9001, "queue": "emails",
  "payload": {"to":"a@x.com","template":"welcome"},
  "status": "ready", "run_at": "2026-07-07T10:00:00Z",
  "attempts": 0, "max_attempts": 5 }
```

**Flow:**

1. Job queue
2. got job
3. ok
4. error/crash
5. producer INSERT jobstatus=ready
6. (jobs table)
7. worker
8. UPDATE ... WHERE id INSELECT ... FOR UPDATE SKIPLOCKED LIMIT 1
9. process
10. status=done
11. reclaim on locked_until<nowattempts++ or DLQ
12. Rate limiter
13. count <= N
14. count > N
15. request key=user:42
16. atomic UPSERT count+1RETURNING count
17. ALLOW
18. REJECT 429

