---
type: "practice"
difficulty: "Intermediate"
---

# Design a hotel or a meeting-room booking system. Model rooms slots and bookings.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pub Sub Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Pub_Sub_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Design a booking system for a hotel (rooms over nights) or meeting rooms (rooms over time slots). The core of the problem is modelling — entities (room, slot/date range, booking), the availability rule (no double-booking), and the booking flow that stays correct under concurrent requests. At the intermediate level the interviewer wants a clean class model, a sound availability check, correct handling of the race where two people book the last room at once, and awareness of the overlap-interval subtlety.

What the interviewer is probing: Can you model rooms/slots/bookings without conflating them? Do you know that "is this slot free?" is really an interval-overlap query? Most importantly — do you understand that check-then-book is a race condition and needs a locking or unique-constraint strategy? Do you handle cancellation, room types vs. specific room assignment, and search?

#### Assumptions
I'll state (and would confirm):

Meeting-room flavor: bookings are [start, end) time intervals on a specific room; hotel flavor is the same with day-granularity [checkIn, checkOut).
Half-open intervals: a booking 10:00–11:00 and 11:00–12:00 do not conflict (end is exclusive).
Users book a room type/category; the system assigns a concrete room (hotels), or book a specific room directly (meeting rooms). I'll model both.
Single-region relational store; concurrency is the hard part.

Plan: entities → the overlap rule → availability query → the concurrency-safe booking flow → cancellation/status lifecycle → search & scaling → summary.

Entities (the model)
Room — id, roomTypeId, name/number, floor, capacity, status (active/maintenance).
RoomType — id, name ("Deluxe" / "Conference-A"), capacity, basePrice, amenities. Hotels sell a type; the concrete room is assigned later. This decoupling lets you oversell/allocate flexibly.
Booking — id, roomId (nullable until assigned), roomTypeId, userId, start, end (half-open interval), status, price, createdAt. This is the reservation of an interval on a room.
BookingStatus — PENDING → CONFIRMED → CHECKED_IN → COMPLETED, plus CANCELLED, NO_SHOW. A state machine keeps transitions legal (you can't confirm a cancelled booking).
User/Guest — id, contact, payment reference.
(Optional) Inventory/AvailabilityCounter per (roomType, date) — a fast counter of remaining rooms, useful for hotels to answer "any Deluxe on the 5th?" in O(1) and to gate overbooking.

The key relationships: a RoomType has many Rooms; a Room has many Bookings over disjoint intervals; a Booking belongs to one user and (eventually) one room.

The availability rule (interval overlap)

Two intervals [s1, e1) and [s2, e2) overlap iff s1 < e2 AND s2 < e1. This is the single most important correctness detail — candidates often write buggy overlap logic. With half-open intervals, back-to-back bookings (e1 == s2) are not a conflict, which is exactly what you want for check-in/check-out and consecutive meetings.

"Is room R free for [s, e)?" becomes: does any active booking on R overlap [s, e)? → the SQL is WHERE room_id = R AND status IN (PENDING,CONFIRMED,...) AND start < e AND s < end. If that returns nothing, R is free.

The booking flow — and the race condition (the crux)

Naive flow: (1) query availability, (2) if free, insert booking. This is a check-then-act race: two requests for the last Deluxe on the 5th both read "1 available," both insert, and you've double-booked. Three sound fixes, cheapest to strongest:

Unique constraint / exclusion constraint (best for correctness). Put a database constraint that makes overlapping bookings on the same room impossible. Postgres has EXCLUDE USING gist (room_id WITH =, tsrange(start,end) WITH &&) — the DB itself rejects any overlapping insert atomically. The loser gets a constraint violation and retries. No application-level lock needed; the DB is the arbiter.

Pessimistic lock (SELECT ... FOR UPDATE). In a transaction, lock the room's rows (or the availability-counter row for (type,date)), re-check inside the lock, then insert. Serializes concurrent bookers on the same resource. Simple, correct, but holds a lock — watch for contention/deadlocks (always lock rooms in a consistent order).

Optimistic concurrency. Version the availability counter; decrement with UPDATE ... WHERE version = v; if 0 rows affected, someone else won — retry. Good when contention is low.

For an interview I'd lead with the exclusion/unique constraint ("make the invalid state unrepresentable in the DB") and mention FOR UPDATE as the portable alternative. Wrap the read-decide-insert in one transaction either way.

For hotels selling a type: hold a (roomType, dateRange) inventory row, decrement atomically to reserve a unit, then assign a concrete room (immediately or at check-in). This avoids locking every room.

Cancellation & lifecycle

Cancellation flips status to CANCELLED and (for counter-based inventory) increments the availability counter back / releases the interval — freeing the slot for others. Don't hard-delete bookings; keep them for audit, no-show, and refund logic. A background job expires PENDING bookings whose payment window elapsed (releasing held inventory), which prevents "phantom" holds from starving availability. The status state-machine enforces legal transitions and drives refunds (cancellation policy: full refund > 24h out, etc.).

Search, pricing, scaling
Search ("Deluxe rooms, Jul 5–8, 2 guests"): filter room types by capacity/amenities, then availability. With per-(type,date) counters this is a fast counter read; without them it's the overlap query per candidate room. Index (room_id, start, end) (or a range GiST index) so overlap checks are logarithmic, not full scans.
Pricing can be a strategy (base + seasonal/dynamic surge), kept separate from booking so rules evolve independently.
Scaling: partition/shard by hotel or region; the concurrency contention is naturally scoped to one room/type/date, so a single DB with the exclusion constraint scales far. Add a read replica for search. For very high demand (event ticket-like), move the availability counter to a fast atomic store (Redis) with the DB as the durable record.
Failure modes to name

Buggy overlap operator (using <= and double-booking back-to-back slots); check-then-act race (the classic); orphaned PENDING holds (need TTL expiry); time-zone/DST bugs (store UTC, render local); clock skew across nodes; and cancellation not releasing inventory. Also idempotency: a retried booking request must not create two bookings — use an idempotency key.

Summary — what a strong answer emphasizes

Separate RoomType (what's sold) from Room (what's assigned) from Booking (an interval reservation), with a status state-machine. State the half-open overlap rule (s1 < e2 && s2 < e1) precisely — this is the most-tested detail. Then, unprompted, raise the concurrency race and solve it with a DB exclusion/unique constraint (or SELECT FOR UPDATE) inside a transaction — "make double-booking impossible in the store," not just in app code. Round it out with cancellation that releases inventory, PENDING-hold expiry, idempotency, UTC storage, and range indexing for fast search. Common mistakes: buggy overlap comparison, ignoring the last-room race, hard-deleting bookings, and modelling only concrete rooms so hotels can't sell by type. Follow-up probes: "how do you prevent overselling under 10k concurrent requests for one room?" (constraint + retry, or Redis atomic counter), "recurring meetings?" (expand to interval instances or store a rule + materialize), and "what if a room goes into maintenance mid-booking?" (block new overlaps, relocate affected bookings).

**Comparison:**

```
Concurrency strategy	Correctness	Contention cost	When to use
DB exclusion / unique constraint	Strongest (DB enforces)	Low; loser retries	Default — "invalid state unrepresentable"
Pessimistic SELECT FOR UPDATE	Strong	Holds lock; deadlock risk	Portable DBs without range-exclude
Optimistic (versioned counter)	Strong w/ retry	Low when contention low	Type-level inventory, low collision
Naive check-then-insert	Broken (race)	—	Never
Entity	Key fields	Responsibility
RoomType	capacity, basePrice, amenities	What is sold; search/pricing unit
Room	roomTypeId, number, active	Assignable physical unit
Booking	roomId?, start, end, status	Interval reservation + lifecycle
AvailabilityCounter	(type,date), remaining	Fast availability, oversell gate
```

**Code:**

```

// Domain model (meeting-room / hotel share the same interval-booking core). enum BookingStatus { PENDING, CONFIRMED, CHECKED_IN, COMPLETED, CANCELLED, NO_SHOW }

record RoomType(long id, String name, int capacity, Money basePrice, Set<String> amenities) {} record Room(long id, long roomTypeId, String number, int floor, boolean active) {}

class Booking { long id, userId, roomTypeId; Long roomId; // roomId null until assigned Instant start, end; // half-open [start, end) BookingStatus status; Money price; }

// Core rule: half-open interval overlap. Back-to-back (e1==s2) does NOT conflict. static boolean overlaps(Instant s1, Instant e1, Instant s2, Instant e2) { return s1.isBefore(e2) && s2.isBefore(e1); }

CODE
Copy

```java
// Concurrency-safe booking. Strategy A: rely on a DB exclusion constraint (preferred).
class BookingService {
    private final DataSource db;

    Booking book(long userId, long roomId, Instant start, Instant end, String idempotencyKey) {
        if (!start.isBefore(end)) throw new IllegalArgumentException("start must precede end");
        try (var conn = db.getConnection()) {
            conn.setAutoCommit(false);
            // idempotency: same key returns the existing booking instead of a duplicate
            var existing = findByIdempotencyKey(conn, idempotencyKey);
            if (existing != null) { conn.commit(); return existing; }

            // The DB has: EXCLUDE USING gist (room_id WITH =, tsrange(start,end) WITH &&)
            // So this INSERT atomically fails if ANY overlapping active booking exists.
            var b = insertBooking(conn, userId, roomId, start, end,
                                  BookingStatus.CONFIRMED, idempotencyKey);
            conn.commit();
            return b;
        } catch (OverlapConstraintViolation e) {
            throw new SlotUnavailableException(roomId, start, end); // loser of the race
        }
    }
}
SQL
Copy
-- Strategy A: make double-booking impossible in the store (PostgreSQL).
CREATE EXTENSION IF NOT EXISTS btree_gist;
CREATE TABLE booking (
  id           BIGSERIAL PRIMARY KEY,
  room_id      BIGINT NOT NULL REFERENCES room(id),
  user_id      BIGINT NOT NULL,
  during       TSRANGE NOT NULL,                    -- [start, end)
  status       TEXT NOT NULL,
  idem_key     TEXT UNIQUE,                         -- idempotency
  -- no two ACTIVE bookings on the same room may have overlapping ranges:
  EXCLUDE USING gist (room_id WITH =, during WITH &&)
         WHERE (status IN ('PENDING','CONFIRMED','CHECKED_IN'))
);

-- Strategy B (portable): pessimistic lock — re-check availability inside a transaction.
BEGIN;
SELECT 1 FROM room WHERE id = :roomId FOR UPDATE;            -- lock the resource
SELECT count(*) FROM booking                                 -- overlap check
  WHERE room_id = :roomId
    AND status IN ('PENDING','CONFIRMED','CHECKED_IN')
    AND lower(during) < :end AND :start < upper(during);
-- if count = 0:
INSERT INTO booking(room_id, user_id, during, status)
VALUES (:roomId, :userId, tsrange(:start, :end, '[)'), 'CONFIRMED');
COMMIT;
```

**Example input / output:**

```text
Overlap rule (half-open [start,end)):
  book A: Room204 10:00-11:00   -> CONFIRMED
  book B: Room204 11:00-12:00   -> CONFIRMED   (e1==s2, NOT a conflict)
  book C: Room204 10:30-11:30   -> SlotUnavailable (10:30<11:00 && 10:00<11:30 -> overlap)

Concurrency race (last Deluxe on Jul 5, 2 threads):
  T1: book(room=204, Jul5)  ┐ both start, both would pass a naive check
  T2: book(room=204, Jul5)  ┘
  With EXCLUDE constraint: first INSERT commits -> CONFIRMED;
                           second INSERT hits overlap -> SlotUnavailableException (retry/other room)
  Result: exactly one booking. No double-booking.

Idempotent retry:
  book(..., idemKey="req-9f2a")  -> creates booking 88123
  book(..., idemKey="req-9f2a")  -> returns SAME booking 88123 (no duplicate)

Cancellation:
  cancel(88123) -> status=CANCELLED, counter.remaining 0 -> 1  (slot freed for others)
```

**Data:**

```json
// RoomType (what the hotel sells)
{ "id": 10, "name": "Deluxe", "capacity": 2, "basePrice": 4500,
  "amenities": ["wifi","ac","breakfast"] }

// Room (a concrete assignable unit of that type)
{ "id": 204, "roomTypeId": 10, "number": "204", "floor": 2, "active": true }

// Booking (an interval reservation; half-open range)
{ "id": 88123, "userId": 501, "roomTypeId": 10, "roomId": 204,
  "start": "2026-07-05T14:00:00Z", "end": "2026-07-08T11:00:00Z",
  "status": "CONFIRMED", "price": 13500, "idemKey": "req-9f2a" }

// Availability counter (fast type-level answer, hotel oversell gate)
{ "roomTypeId": 10, "date": "2026-07-05", "total": 12, "booked": 11, "remaining": 1 }
```

**Flow:**

1. yes
2. no
3. ok
4. overlap violation
5. book(room, [start,end),idemKey)
6. idemKey seen?
7. return existing booking
8. BEGIN transaction
9. INSERT booking (EXCLUDE gistenforces no overlap)
10. COMMIT -> CONFIRMED
11. ROLLBACK -> SlotUnavailable(retry / other room)
12. payment ok
13. hold expires / user cancels
14. arrival
15. cancel (release inventory)
16. not arrived
17. check-out
18. PENDING
19. CONFIRMED
20. CANCELLED
21. CHECKED_IN
22. NO_SHOW
23. COMPLETED

