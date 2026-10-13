---
type: "practice"
difficulty: "Deep"
---

# How would you handle inventory and prevent overselling in an e-commerce checkout?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Idempotent Order APIs](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Idempotent_Order_APIs/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

How would you handle inventory and prevent overselling in an e-commerce checkout?

#### Triage
— what is really being asked

Overselling means committing to sell more units than physically exist because two or more concurrent buyers were each told "in stock" and both completed checkout. The interviewer is probing my grasp of concurrency control, distributed consistency, reservations vs. deductions, and idempotency under real e-commerce conditions: flash sales, retries, timeouts, and payment that happens asynchronously over seconds. At "deep understanding" level a shallow "use a lock" answer fails — they want the trade-offs between atomic DB updates, pessimistic vs. optimistic locking, reservation with TTL, saga/2PC, and how you keep inventory correct when payment can succeed or fail minutes later.

Clarifying questions: Is this a single-warehouse retailer or multi-warehouse/marketplace? Do we allow backorders? How is payment done — synchronous card auth or async (UPI/redirect) that can take seconds to minutes? What's the peak concurrency (normal vs. flash sale of a hot SKU)? Do we need strict "never oversell" (electronics) or is small oversell tolerable (recoverable)? I'll assume: strict no-oversell, async payment, hot-SKU flash sales, single logical stock counter per SKU (may be sharded across warehouses).

First principles: the invariant and where it can break

The invariant is simple: available >= 0 for every SKU, and every fulfilled order corresponds to a real decrement. Overselling happens when the read ("is it available?") and the write ("take one") are not atomic across concurrent requests — the classic read-modify-write race. So the core of the solution is: the availability check and the decrement must be a single atomic, serialized operation, and every step must be idempotent so retries don't double-count.

Second reality: checkout is not instantaneous. A user adds to cart, we must hold stock while they pay (seconds to minutes), and payment can fail. So we don't just decrement — we reserve, then confirm or release. This gives three states of stock: on_hand, reserved, and available = on_hand - reserved.

The reservation model (the heart of the answer)

Model inventory per SKU as { on_hand, reserved }. Flow:

Add to cart / begin checkout → Reserve. Atomically: if available >= qty, increment reserved by qty and create a reservation row with a TTL (e.g., 10–15 min) and an idempotency key. If not enough available → fail fast ("out of stock").
Payment. Payment runs against the reservation. Because stock is already held, no one else can buy those units.
On payment success → Confirm. Move reserved→sold: on_hand -= qty, reserved -= qty, mark reservation confirmed, emit OrderPlaced.
On payment failure / timeout / abandonment → Release. reserved -= qty, mark reservation released, returning stock to available.
TTL expiry (cart abandoned, no callback): a sweeper/expirer releases stale reservations so held stock isn't lost forever.

This is a two-phase reservation (reserve → confirm/release), which is exactly the saga pattern for inventory and much simpler than distributed 2PC.

Making the atomic step correct — the concurrency control choice

The single most important operation is the atomic conditional decrement. Options, with trade-offs:

Atomic conditional UPDATE (recommended default). UPDATE inventory SET reserved = reserved + :q WHERE sku=:s AND on_hand - reserved >= :q and check rows_affected == 1. The WHERE guard makes the DB serialize it via row locks; if 0 rows updated, it's out of stock. Simple, correct, no explicit locking code, works in one round trip. This is the workhorse.
Pessimistic lock (SELECT ... FOR UPDATE). Lock the row, check, update, commit. Correct but holds a lock for the transaction duration; under a flash sale all buyers of the hot SKU serialize on one row → contention and latency. Use only when logic between read and write is complex.
Optimistic concurrency (version column). Read version, update WHERE version=:v; retry on mismatch. Great for low contention, but on a hot SKU everyone collides and retries → livelock. Bad fit for flash sales.
Redis atomic counter (best for extreme flash-sale hotspots). Keep available in Redis; use an atomic DECRBY in a Lua script (check-and-decrement in one server-side op — Redis is single-threaded so it's serialized). Handles tens of thousands of QPS on one SKU without DB row contention. The DB remains source of truth and is reconciled asynchronously. This is how big flash sales are actually survived.

The unifying principle across all four: compare-and-set in one indivisible step; never read-then-write in two round trips.

Idempotency — the second pillar

Payment webhooks retry; clients double-click; networks time out and clients resend. Without idempotency, a retried "confirm" decrements twice → under-sell/data corruption, and a retried "reserve" double-holds. So:

Every reserve/confirm/release carries an idempotency key (e.g., reservationId or orderId). Store applied keys; an already-applied operation is a no-op returning the prior result.
The payment provider's event id dedupes webhook retries. This turns "at-least-once" delivery into "effectively-exactly-once."
Distributed / multi-warehouse & scale
Single logical counter is a bottleneck for a hot SKU. Shard stock across warehouses/segments (available split into N buckets) and reserve from any non-empty bucket → parallelism. Reconcile totals asynchronously.
Source of truth vs. cache: the displayed "in stock" on search/PDP is eventually consistent (from a cache/index) and used only for display. The authoritative reservation always re-checks live inventory at checkout — never trust the search-time flag. (Some oversell risk exists purely from display lag, which is why the checkout-time atomic reserve is mandatory.)
Ordering/consistency: put inventory in a strongly consistent store (Postgres, or DynamoDB with conditional writes). Emit events (Kafka) for downstream (analytics, replenishment) but keep the decision on the transactional store.
Failure modes & how they're handled
Payment succeeds but confirm fails (crash between steps): the reservation is still held; on recovery, reconcile by re-driving confirm using the idempotency key. Outbox pattern ensures the OrderPlaced event and DB commit are atomic.
Reservation leaks (release never called): TTL sweeper + a reconciliation job that recomputes reserved from live reservation rows and fixes drift.
Redis and DB diverge (flash-sale path): Redis is the fast gate; a periodic reconcile job syncs Redis←DB, and confirmed sales are the ledger. Under-sell (holding a few extra) is acceptable; over-sell is not, so we err toward blocking.
Backorders/oversell tolerance: if the business allows it, permit available to go slightly negative with a policy flag — but that must be an explicit business decision, not an accident.
Refund/return: returns increment on_hand back via a compensating transaction (also idempotent).
Summary — what a strong answer emphasizes

Prevent overselling with a two-phase reservation (reserve → confirm/release) over available = on_hand - reserved, where the reserve step is a single atomic compare-and-decrement (conditional UPDATE ... WHERE available >= qty, or a Redis Lua DECRBY for hot SKUs), every operation is idempotent (idempotency keys / webhook dedup), and reservations have a TTL with a sweeper to reclaim abandoned holds. Pick the concurrency mechanism by contention: conditional-UPDATE by default, pessimistic locks for complex logic, Redis counters for flash-sale hotspots — never naive optimistic locking on a hot SKU. Keep display stock eventually consistent but always re-validate authoritatively at checkout, use the outbox pattern for crash-safe confirm, and run reconciliation/TTL jobs to heal drift.

#### Common mistakes
read-then-write in two steps; decrementing at add-to-cart with no release/TTL; trusting the search-time stock flag; forgetting idempotency so webhook retries corrupt counts; SELECT FOR UPDATE on a hot row during a flash sale (serial bottleneck).

#### Follow-up
probes: "How do you handle 100k concurrent buys of one SKU?" (Redis atomic counter + shard + async DB reconcile); "Payment webhook arrives twice?" (idempotency key no-op); "Service crashes mid-confirm?" (outbox + recovery re-drive); "Multi-item cart, one item OOS?" (reserve all-or-nothing, release the rest).

**Comparison:**

```
Concurrency mechanism	Best for	Weakness
Conditional UPDATE ... WHERE available>=qty	Default; correct + simple	DB row-lock contention at extreme hotspots
SELECT ... FOR UPDATE (pessimistic)	Complex read-then-write logic	Serializes hot SKU → latency in flash sale
Version column (optimistic)	Low-contention SKUs	Livelock/retries on hot SKU
Redis atomic DECRBY (Lua)	Flash-sale hotspots, huge QPS	Needs reconcile with DB source of truth
Stock state	Meaning	Set by
on_hand	Physical units in warehouse	Receiving / confirm / returns
reserved	Held for in-flight checkouts	reserve (+) / confirm,release (−)
available = on_hand − reserved	Sellable right now	derived
Failure	Risk	Mitigation
Retried webhook	Double decrement	Idempotency key / event-id dedup
Crash after pay, before confirm	Held stock, no order	Outbox + recovery re-drive
Abandoned cart	Stock stuck reserved	TTL + sweeper release
Display stale in-stock	Buyer sees OOS at checkout	Authoritative re-check at reserve
```

**Code:**

```

-- Inventory as on_hand + reserved; available is derived CREATE TABLE inventory ( sku TEXT PRIMARY KEY, on_hand INT NOT NULL CHECK (on_hand >= 0), reserved INT NOT NULL DEFAULT 0 CHECK (reserved >= 0), CHECK (reserved <= on_hand) -- DB-level guard: can't reserve more than exists );

CREATE TABLE reservation ( id UUID PRIMARY KEY, -- also the idempotency key sku TEXT NOT NULL, qty INT NOT NULL, status TEXT NOT NULL, -- HELD | CONFIRMED | RELEASED expires_at TIMESTAMPTZ NOT NULL, created_at TIMESTAMPTZ NOT NULL DEFAULT now() );

-- 1) RESERVE: atomic compare-and-decrement. rows_affected==1 => success, 0 => out of stock. UPDATE inventory SET reserved = reserved + :qty WHERE sku = :sku AND on_hand - reserved >= :qty; -- the guard that prevents overselling -- if 1 row: INSERT reservation(id=:resId, ..., status='HELD', expires_at=now()+interval '15 min')

-- 2) CONFIRM (on payment success): idempotent — only acts on a still-HELD reservation UPDATE reservation SET status='CONFIRMED' WHERE id = :resId AND status = 'HELD'; -- 0 rows => already processed, no-op UPDATE inventory SET on_hand = on_hand - :qty, reserved = reserved - :qty WHERE sku = :sku;

-- 3) RELEASE (payment fail / TTL expiry): idempotent UPDATE reservation SET status='RELEASED' WHERE id = :resId AND status = 'HELD'; UPDATE inventory SET reserved = reserved - :qty WHERE sku = :sku;

CODE
Copy

```lua
-- Redis flash-sale gate: atomic check-and-decrement in one server-side op (Lua = serialized)
-- KEYS[1] = stock:{sku}   ARGV[1] = qty   ARGV[2] = reservationId (idempotency)
if redis.call('SISMEMBER', 'applied:'..KEYS[1], ARGV[2]) == 1 then
  return 1                                   -- already reserved with this id -> idempotent OK
end
local avail = tonumber(redis.call('GET', KEYS[1]) or '0')
if avail >= tonumber(ARGV[1]) then
  redis.call('DECRBY', KEYS[1], ARGV[1])
  redis.call('SADD', 'applied:'..KEYS[1], ARGV[2])
  return 1                                   -- reserved
else
  return 0                                   -- out of stock, no oversell
end
JAVASCRIPT
Copy
// Reserve → pay → confirm/release, all idempotent
async function checkout({ sku, qty, orderId }) {
  const reserved = await reserveAtomic(sku, qty, orderId);   // conditional UPDATE or Redis Lua
  if (!reserved) return { status: 'OUT_OF_STOCK' };

  try {
    const pay = await payment.charge(orderId);               // async, may retry/timeout
    if (pay.ok) { await confirm(orderId, sku, qty); return { status: 'PLACED' }; }
    await release(orderId, sku, qty);
    return { status: 'PAYMENT_FAILED' };
  } catch (e) {
    // do NOT release on ambiguous timeout — let TTL sweeper / reconcile decide, then re-drive
    throw e;
  }
}

// Webhook is deduped by provider event id; confirm() is a no-op if already CONFIRMED
async function onPaymentWebhook(evt) {
  if (await alreadyProcessed(evt.id)) return;                // idempotent webhook
  if (evt.type === 'success') await confirm(evt.orderId, evt.sku, evt.qty);
  else await release(evt.orderId, evt.sku, evt.qty);
  await markProcessed(evt.id);
}
```

**Example input / output:**

```text
Race scenario: SKU has on_hand=1, reserved=0 (available=1). Buyer A and Buyer B checkout simultaneously.

T0  A: UPDATE ... SET reserved=reserved+1 WHERE available>=1  -> DB row-locks, rows=1  (A reserved)
T0  B: same UPDATE, but now available = 1-1 = 0, guard fails    -> rows=0  (B: OUT_OF_STOCK)
    Result: exactly ONE reservation. No oversell. ✓

Payment webhook retried twice (at-least-once delivery):
  webhook#1 evt=e42 -> confirm r-8f1 (HELD->CONFIRMED), on_hand 1->0        ✓
  webhook#2 evt=e42 -> alreadyProcessed(e42)==true -> no-op                 ✓ (idempotent)

Abandoned cart:
  A reserves, closes tab, never pays. expires_at passes.
  Sweeper: release HELD reservations where expires_at < now() -> reserved 1->0, available 0->1 ✓

Flash sale (Redis path), stock=1000, 50k concurrent DECRBY via Lua:
  first 1000 return 1 (reserved), remaining 49000 return 0 (out of stock), zero oversell,
  DB reconciled from confirmed ledger asynchronously.                       ✓
```

**Data:**

```json
// Reservation lifecycle records
{ "id": "r-8f1", "sku": "PEG-BLK-9", "qty": 1, "status": "HELD",
  "expires_at": "2026-07-07T10:15:00Z" }
// after payment success
{ "id": "r-8f1", "sku": "PEG-BLK-9", "qty": 1, "status": "CONFIRMED" }

// Inventory row transitions for one SKU (on_hand=5)
// start:    { on_hand: 5, reserved: 0, available: 5 }
// reserve:  { on_hand: 5, reserved: 1, available: 4 }
// confirm:  { on_hand: 4, reserved: 0, available: 4 }
// (or) release from HELD: { on_hand: 5, reserved: 0, available: 5 }
```

**Flow:**

1. reserve (atomic:available>=qty ?reserved+=qty : fail)
2. payment success(on_hand-=qty,reserved-=qty)[idempotent]
3. payment fail OR TTL expiry(reserved-=qty)[idempotent]
4. return/refund(on_hand+=qty) %%compensating txn
5. Available
6. Held
7. Confirmed
8. Payment
9. Inventory (atomic CAS)
10. Checkout Svc
11. Buyer
12. Payment
13. Inventory (atomic CAS)
14. Checkout Svc
15. Buyer
16. alt
17. [reserved (rows=1)]
18. [out of stock (rows=0)]
19. TTL sweeper releases HELD reservations past expires_at
20. checkout(sku, qty)
21. reserve (UPDATE ... WHERE available>=qty)
22. charge (async)
23. webhook success (idempotent by event id)
24. confirm (on_hand-=qty, reserved-=qty)
25. ORDER PLACED
26. OUT OF STOCK

