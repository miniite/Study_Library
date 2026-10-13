---
type: "practice"
difficulty: "Deep"
---

# A time-of-check-to-time-of-use (TOCTOU) bug lets two requests both pass a "seat is available" check and both book it. Explain TOCTOU generally and how you'd atomically check-and-reserve in the DB.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Exactly Once via Idempotency](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Exactly_Once_via_Idempotency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TOCTOU, or time-of-check-to-time-of-use, is a classic concurrency issue where a piece of data looks valid at one point in time but becomes invalid by another due to changes between the check and use phases. In the context of seat booking, this means two requests might independently pass an "is seat available" check, only for both to try to book it, leading to an out-of-stock situation.

To prevent this race condition, we need to ensure that the reservation operation is atomic—meaning it should complete in a single step without interruption. Here's how you would implement such a solution using database transactions and locking mechanisms.

#### Triaging the Question

The key aspects of the question are:

Understanding TOCTOU: What it means for seat booking.
Atomic Check-and-Reserve Logic: How to ensure that only one request can successfully book a seat, even when multiple requests are checking availability simultaneously.

#### Assumptions

We use SQL as our database language.
The booking process involves updating the seat count in a table.
The application uses a relational database with support for transactions and row-level locking.

#### First Principles Solution
Check Availability: Initially, check if there are available seats.
Lock the Row: If available, lock the specific seat or reservation row to prevent other requests from modifying it while you process your booking.
Update State: Atomically update the state of the seat (decrementing the count) and mark it as reserved.
Commit Transaction: Ensure all changes are committed only if no race condition is detected.
Step-by-Step Implementation

Initial Check:

Query to check if there are available seats (available_seats > 0).

Lock the Row (using row-level locking):

Use SELECT ... FOR UPDATE in SQL to lock the specific seat or reservation entry.

Update State:

Perform the update operation, decrementing the available_seats and setting a flag like is_reserved.

Commit Transaction: Commit all changes only if no other transactions have modified the row since the initial check.

Code Example
SQL
-- Example SQL code for atomic check-and-reserve

START TRANSACTION;

SELECT available_seats FROM seats WHERE seat_id = ? FOR UPDATE;  -- Step 1: Check and lock

IF available_seats > 0 THEN
    UPDATE seats SET available_seats = available_seats - 1, is_reserved = 1 WHERE seat_id = ? AND available_seats > 0;  -- Step 2-3: Lock + Update state
    COMMIT;  -- Step 4: Commit transaction if no race condition detected
ELSE
    ROLLBACK;
END IF;

COMMIT;  -- Ensure all changes are committed only here.

#### Closing Summary
and Common Mistakes

#### Closing Summary
By using row-level locking (SELECT ... FOR UPDATE) during the check phase, we ensure that other transactions do not modify the seat's state until our transaction completes. This atomic process prevents TOCTOU bugs by ensuring that no two requests can simultaneously book the same seat.

#### Common Mistakes

Forgetting to use FOR UPDATE in the initial SELECT query.
Not committing or rolling back based on the outcome of the lock and update operations.
Ignoring the possibility of other concurrent transactions modifying the state between check and reserve steps.
Additional Considerations
Deadlocks: Ensure that your application can handle potential deadlocks, which may arise if two requests are locking each other in a circular pattern. Implement timeout mechanisms or backoff strategies to break such cycles.
Performance: Row-level locking can impact performance on highly concurrent systems. Consider using optimistic concurrency control or distributed locks as alternatives.

**Comparison:**

```
SEATS
int
seat_id
PK
int
available_seats
bool
is_reserved
```

**Code:**

```sql
START TRANSACTION;

SELECT available_seats FROM seats WHERE seat_id = ? FOR UPDATE;

IF available_seats > 0 THEN
    UPDATE seats SET available_seats = available_seats - 1, is_reserved = 1 WHERE seat_id = ? AND available_seats > 0;
    COMMIT;
ELSE
    ROLLBACK;
END IF;

COMMIT;
```

**Example input / output:**

```text
-- Initial state:
SELECT * FROM seats;
-- Output:
-- +---------+--------------+
-- | seat_id | available_seats |
-- +---------+-----------------+
-- |       1 |               5 |
-- |       2 |               3 |
-- +---------+-----------------+

-- First request checks and reserves seat_id = 1
START TRANSACTION;
SELECT available_seats FROM seats WHERE seat_id = 1 FOR UPDATE; -- Locks the row

IF available_seats > 0 THEN
    UPDATE seats SET available_seats = available_seats - 1, is_reserved = 1 WHERE seat_id = 1 AND available_seats > 0;
COMMIT;

-- Output after first request commits:
-- +---------+--------------+
-- | seat_id | available_seats |
-- +---------+-----------------+
-- |       1 |               4 |
-- |       2 |               3 |
-- +---------+-----------------+

-- Second request checks and tries to reserve the same seat
START TRANSACTION;
SELECT available_seats FROM seats WHERE seat_id = 1 FOR UPDATE; -- Locks the row

IF available_seats > 0 THEN
    UPDATE seats SET available_seats = available_seats - 1, is_reserved = 1 WHERE seat_id = 1 AND available_seats > 0;
ELSE
    ROLLBACK;
END IF;

-- Output after second request rolls back:
-- +---------+--------------+
-- | seat_id | available_seats |
-- +---------+-----------------+
-- |       1 |               4 |
-- |       2 |               3 |
-- +---------+-----------------+
```

**Data:**

```json
{
  "seats": [
    { "seat_id": 1, "available_seats": 5 },
    { "seat_id": 2, "available_seats": 3 }
  ]
}
```

**Flow:**

1. Lock Mechanism
2. Yes
3. No
4. Start Transaction
5. Check Availability
6. Lock Row
7. Update State & Commit
8. Rollback and End
9. Row-Level Locking (FORUPDATE)
10. End Transaction

