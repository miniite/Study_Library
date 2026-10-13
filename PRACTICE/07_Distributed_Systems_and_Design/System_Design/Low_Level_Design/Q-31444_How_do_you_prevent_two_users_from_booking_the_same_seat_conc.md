---
type: "practice"
difficulty: "Deep"
---

# How do you prevent two users from booking the same seat concurrently?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Rule Engines and Pricing](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Rule_Engines_and_Pricing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's being asked: In a BookMyShow-style system, guarantee that when two users click the same seat for the same show at the same moment, exactly one succeeds. This is a concurrency / mutual-exclusion problem over a shared resource (a ShowSeat row). At "deep understanding" level the interviewer wants me to (1) name the exact race, (2) walk through the real mechanisms — DB pessimistic lock, optimistic/conditional update, unique constraint, distributed lock (Redis), and a temporary hold/reservation — with their trade-offs, (3) reason about the hold window and payment step (the seat is locked for minutes, not milliseconds), and (4) discuss failure modes: crashed holder, expired lock, and the difference between correctness and user experience.

The race, precisely: Two requests both read ShowSeat.status = AVAILABLE, both decide "free," both write BOOKED. This is a lost-update / check-then-act race. Any correct solution must make the check-and-set atomic at some serialization point.

Clarifying assumptions: Seats are assigned; a booking has a hold phase (user selects seats) then a pay phase (seconds to minutes later); source of truth is a relational DB (Postgres/MySQL); we may have a Redis layer. Correctness requirement: no seat ever confirmed to two bookings. UX requirement: don't hold seats forever if the user abandons.

From first principles — the mechanisms

The whole problem reduces to: make "if AVAILABLE then set HELD" a single atomic operation, then treat the hold as a lease with an expiry. There are four standard ways to get that atomicity; strong candidates know all four and when each fits.

1. Pessimistic DB lock (SELECT ... FOR UPDATE). In one transaction, SELECT status FROM show_seat WHERE id IN (...) FOR UPDATE. This row-locks the chosen seats; a concurrent transaction touching the same rows blocks until the first commits. Then check all are AVAILABLE, UPDATE ... SET status='HELD', commit. The DB serializes access to those rows — the second txn now reads HELD and fails. Correct and simple. Cost: it holds DB locks and a connection for the duration of the transaction, so you must keep the transaction short — you lock only long enough to flip AVAILABLE→HELD, not across the whole payment. Deadlock risk if you lock multiple seats in inconsistent order → always lock seats sorted by id.

2. Optimistic / conditional update (compare-and-set). No explicit lock. Do a single conditional statement: UPDATE show_seat SET status='HELD', booking_id=?, hold_expires_at=? WHERE id IN (...) AND status='AVAILABLE' and check the affected-row count. If it equals the number of seats requested, you won all of them; if it's less, someone beat you to at least one — you roll back (or release the ones you did get) and tell the user "seat just taken." This is often the best default: no long locks, one round-trip, and the DB's row-level atomicity guarantees only one updater flips a given row from AVAILABLE. A version column (WHERE version = v) generalizes it for richer state.

3. Unique constraint as the final backstop. Even with the above, make correctness impossible to violate at confirm time: a table seat_booking(show_id, seat_id) UNIQUE. When you confirm, insert one row per (show, seat); the DB rejects the second insert with a duplicate-key error. This is a hard, storage-level guarantee independent of application logic — belt-and-suspenders for the money step.

4. Distributed lock (Redis) for the hold. SET lock:show:<show>:seat:<seat> <bookingId> NX PX 600000 — acquire a lease with a TTL for each seat; only if all locks are acquired do you proceed. The TTL means a crashed/abandoned holder auto-releases after the window (no stuck seats). Redis is fast and central, good when you want the hot path off the DB. Caveat: single-node Redis can lose a lock on failover; for strict safety use Redlock or, more pragmatically, treat Redis as the fast hold and the DB conditional-update + unique constraint as the authoritative guard — the classic "cache lock for UX, DB for correctness" pattern.

Putting it together — the hold model. The key realization at this level: booking isn't a single atomic op, it's two phases separated by human time (select → pay, up to ~8–10 min). So:

On seat select: atomically flip AVAILABLE→HELD (mechanism 2, or 1) with an hold_expires_at = now+8min and a booking_id. Optionally also take a Redis lease.
The user pays within the window. On payment success, flip HELD→BOOKED (and insert into the unique seat_booking table). On the confirm, verify the hold still belongs to this booking and hasn't expired.
A sweeper / TTL releases holds that expire: a background job (or lazy check) flips HELD→AVAILABLE where hold_expires_at < now. Redis TTL does this automatically for the cache lock.

Why not just one big transaction across payment? Because payment can take minutes and involves an external gateway — you cannot hold a DB row lock or a transaction open that long (connection exhaustion, gateway timeouts, deadlocks). The hold pattern externalizes the lock into application state (a HELD status + expiry) precisely so the fast, safe DB lock is held only for milliseconds.

Failure modes and how the design handles them:

Holder crashes / abandons: hold_expires_at + sweeper (and Redis TTL) auto-releases the seat — no permanent lock.
Payment succeeds but confirm race: the unique constraint on (show, seat) makes a duplicate confirm impossible; the loser's payment is refunded.
Clock/expiry edge: on confirm, re-check booking_id and expiry inside the transaction so a just-expired hold that was reclaimed by someone else can't be confirmed.
Deadlock on multi-seat locks: always acquire in sorted seat-id order.
Idempotency: confirm is idempotent on bookingId so a retried payment webhook doesn't double-book.

Correctness vs. UX. Correctness is guaranteed by the atomic conditional update + unique constraint. UX is improved by holds (seats don't vanish under your cursor), fast Redis locks, and clear "seat just taken, pick another" messaging with an updated seat map. Both matter; the interviewer wants you to distinguish the guarantee (DB atomicity) from the nicety (hold window, Redis speed).

Summary — what a strong answer emphasises

Make "check AVAILABLE then set HELD" atomic at a single serialization point — via SELECT ... FOR UPDATE (pessimistic) or, preferably, a conditional UPDATE ... WHERE status='AVAILABLE' checking affected rows (optimistic CAS) — hold the DB lock for milliseconds only, and model the multi-minute booking as a HELD lease with hold_expires_at released by a sweeper/TTL. Add a UNIQUE(show_id, seat_id) constraint as the storage-level backstop at confirm, and optionally a Redis SET NX PX lease for a fast, auto-expiring hold. Never hold a DB transaction open across payment. Common mistakes: check-then-act in application code without atomicity, locking across the payment step, forgetting hold expiry (stuck seats), and inconsistent lock ordering (deadlock). Likely follow-ups: "what if Redis fails?" (DB remains authoritative), "how do you release abandoned holds?" (TTL + sweeper), and "how do you make confirm idempotent?".

**Comparison:**

```
Mechanism	How it serializes	Lock duration	Pros	Cons / when
SELECT ... FOR UPDATE (pessimistic)	Row lock in txn	Milliseconds (short txn only)	Simple, strong	Holds conn/lock; deadlock if unordered; never across payment
Conditional UPDATE ... WHERE status='AVAILABLE' (optimistic CAS)	Row-level atomic write	None held	1 round-trip, no long lock, scales	Must check affected rows; retry on loss
UNIQUE(show_id, seat_id) at confirm	Storage constraint	n/a	Hard guarantee, app-independent	Only catches at insert; use as backstop
Redis SET NX PX lease	Central CAS + TTL	Whole hold window (auto-expire)	Fast, off-DB, self-releasing	Failover can drop lock → keep DB authoritative
Concern	Wrong approach	Correct approach
Check-then-act race	read AVAILABLE in app, then write	atomic conditional UPDATE / FOR UPDATE
Long payment step	one txn across gateway call	HELD status + expiry, short DB lock only
Abandoned holds	hold forever	hold_expires_at + sweeper / Redis TTL
Multi-seat deadlock	lock in click order	lock seats sorted by id
Retried payment	confirm twice	idempotent confirm + unique (show,seat)
```

**Code:**

```

-- (A) OPTIMISTIC / conditional CAS — the default. One statement, atomic per row. -- Flip only seats that are still AVAILABLE; check affected rows == requested count. UPDATE show_seat SET status = 'HELD', booking_id = :bookingId, hold_expires_at = NOW() + INTERVAL '8 minutes' WHERE show_id = :showId AND seat_id IN (:seatIds) -- e.g. (A1, A2) AND status = 'AVAILABLE'; -- if ROW_COUNT() < len(seatIds): someone won a seat first -> ROLLBACK the whole hold, -- release any partial grabs, tell user "seat just taken".

-- (B) PESSIMISTIC alternative — lock rows, then decide. Keep the txn SHORT. BEGIN; SELECT id, status FROM show_seat WHERE show_id = :showId AND seat_id IN (:seatIds) ORDER BY seat_id -- consistent order => no deadlock FOR UPDATE; -- blocks concurrent holders on these rows -- if all AVAILABLE: UPDATE show_seat SET status='HELD', booking_id=:bookingId, hold_expires_at = NOW() + INTERVAL '8 minutes' WHERE show_id=:showId AND seat_id IN (:seatIds); COMMIT; -- lock released here (held only for ms)

-- (C) STORAGE-LEVEL BACKSTOP at confirm — makes double-book impossible. CREATE TABLE seat_booking (show_id, seat_id, booking_id, PRIMARY KEY (show_id, seat_id)); -- UNIQUE per (show,seat) -- confirm step: BEGIN; INSERT INTO seat_booking (show_id, seat_id, booking_id) VALUES (:showId, :seatId, :bookingId); -- duplicate -> unique violation -> loser fails UPDATE show_seat SET status='BOOKED' WHERE show_id=:showId AND seat_id=:seatId AND booking_id=:bookingId AND hold_expires_at > NOW(); -- re-check hold still ours COMMIT;

CODE
Copy
```python
# (D) Redis lease for a FAST, auto-expiring hold (UX layer). DB stays authoritative.
def acquire_holds(redis, show_id, seat_ids, booking_id, ttl_ms=600000):
    acquired = []
    for sid in sorted(seat_ids):                       # sorted -> deadlock-free
        key = f"lock:show:{show_id}:seat:{sid}"
        if redis.set(key, booking_id, nx=True, px=ttl_ms):
            acquired.append(key)
        else:                                          # someone holds it -> roll back
            for k in acquired:
                redis.delete(k)
            return False                               # "seat just taken"
    return True                                        # all seats leased; TTL auto-frees on crash
```

**Example input / output:**

```text
Concurrent hold — two users, seat A1, exact same moment
--------------------------------------------------------
t0  A1.status = AVAILABLE
t1  req_1: UPDATE ... WHERE status='AVAILABLE'  -> DB serializes -> 1 row -> WON A1
    req_2: UPDATE ... WHERE status='AVAILABLE'  -> 0 rows (A1 now HELD) -> LOST A1
t2  req_1 booking b_900 = PENDING_PAYMENT (A1 held, expires t0+8m)
    req_2 sees 0 affected -> "A1 just taken", seat map refreshed, user picks A5

Abandoned hold (correctness of release)
--------------------------------------------------------
t0     b_900 holds A1, hold_expires_at = t0+8m ; user closes tab
t0+8m  sweeper: UPDATE show_seat SET status='AVAILABLE', booking_id=NULL
                WHERE status='HELD' AND hold_expires_at < NOW()  -> A1 free again
       (Redis lease on A1 also auto-expired via PX TTL)

Duplicate confirm blocked by unique constraint
--------------------------------------------------------
b_900 confirm: INSERT seat_booking(s_501, A1) -> OK -> A1 BOOKED
retry/webhook: INSERT seat_booking(s_501, A1) -> UNIQUE violation -> no double-book
```

**Data:**

```json
// ShowSeat row transitions under contention
{
  "show_seat": { "show_id": "s_501", "seat_id": "A1", "status": "AVAILABLE" },
  "request_1": { "booking_id": "b_900", "action": "hold A1,A2" },
  "request_2": { "booking_id": "b_901", "action": "hold A1,A3", "same_instant": true },
  "outcome": {
    "A1": "HELD by b_900 (CAS won by request_1)",
    "b_900": "gets A1 + A2 -> PENDING_PAYMENT",
    "b_901": "affected_rows(1) < requested(2) -> rollback, A3 released, user re-picks"
  }
}
```

**Flow:**

1. DB (show_seat)
2. User 2
3. User 1
4. DB (show_seat)
5. User 2
6. User 1
7. "Seat just taken" -> refresh map, pick another
8. hold expiry sweeper frees any HELD past hold_expires_at
9. UPDATE A1 SET HELD WHERE status=AVAILABLE
10. UPDATE A1 SET HELD WHERE status=AVAILABLE
11. 1 row affected (WON)
12. 0 rows affected (LOST)
13. Pay within 8-min hold window
14. confirm: INSERT seat_booking(A1) UNIQUE, set BOOKED

