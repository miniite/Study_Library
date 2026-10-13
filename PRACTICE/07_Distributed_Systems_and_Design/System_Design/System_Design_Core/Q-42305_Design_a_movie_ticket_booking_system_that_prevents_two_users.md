---
type: "practice"
difficulty: "Intermediate"
---

# Design a movie ticket booking system that prevents two users from booking the same seat.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Payment Reconciliation](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Payment_Reconciliation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a movie ticket booking system that ensures no two users book the same seat, we need to focus on both the data structure and the transactional integrity of our database. The core idea is to use a combination of locking mechanisms and atomic transactions to ensure that once a user books a seat, it remains unavailable until the transaction completes.

#### Triaging the Question

The question is essentially asking for a design solution that prevents concurrent booking conflicts when multiple users try to book the same seat simultaneously. This involves understanding concurrency control, database locking, and possibly implementing optimistic or pessimistic locking strategies. The key assumption here is that we are dealing with a distributed system where multiple users can interact with our application concurrently.

#### First-Principles Solution
Data Model: We need to store seat information and booking status efficiently.
Concurrency Control: Implementing mechanisms like row-level locking or optimistic concurrency control.
Transaction Management: Ensuring that the booking process is atomic and consistent.
Scalability Considerations: Design for high-concurrency scenarios.
Step-by-Step Solution

Data Model:

Define a Seat model with columns such as seat_id, row, column, and booked.
Each seat should have a status indicating whether it is booked or not.
Optionally, include additional fields like booking_start_time and booking_end_time.

Concurrency Control:

Use row-level locking (e.g., SELECT ... FOR UPDATE in SQL) to prevent other transactions from modifying the same row while one transaction is processing.
Implement optimistic concurrency control by checking if a seat has been modified between reading and writing its state.

Transaction Management:

Ensure that all operations related to booking are wrapped within a single database transaction.
Use isolation levels (e.g., SERIALIZEABLE) to prevent dirty reads, write skew, etc.

Scalability Considerations:

Use sharding strategies if the system needs to handle very high traffic.
Implement caching for frequently accessed seat information to reduce database load.

#### Closing Summary
and Common Mistakes

In summary, designing a movie ticket booking system that prevents two users from booking the same seat involves a combination of robust data modeling, concurrency control mechanisms, and transaction management. Key mistakes include not properly handling database transactions or missing optimistic locking checks, which can lead to race conditions.

#### Example
Code

Here's an example of how you might implement this in SQL:

SQL
-- Example SQL for booking a seat
BEGIN TRANSACTION;

SELECT * FROM seats WHERE seat_id = ? FOR UPDATE;  -- Lock the seat

UPDATE seats SET booked = 'true' WHERE seat_id = ? AND booked = 'false';

IF @@ROWCOUNT = 0 BEGIN  -- Check if the update was successful
    ROLLBACK;
    RAISEERROR('Seat is already booked.');
END ELSE BEGIN
    COMMIT;
    SELECT * FROM seats WHERE seat_id = ?;  -- Return updated seat information
END
Closing Thoughts

Remember that this design needs to be adapted based on specific constraints and requirements of your application. Always consider performance, scalability, and fail-safes in your implementation.

**Comparison:**

```
book_seat
SEATS
BOOKINGS
```

**Code:**

```sql
-- Example SQL for booking a seat
BEGIN TRANSACTION;

SELECT * FROM seats WHERE seat_id = ? FOR UPDATE;  -- Lock the seat

UPDATE seats SET booked = 'true' WHERE seat_id = ? AND booked = 'false';

IF @@ROWCOUNT = 0 BEGIN  -- Check if the update was successful
    ROLLBACK;
    RAISEERROR('Seat is already booked.');
END ELSE BEGIN
    COMMIT;
    SELECT * FROM seats WHERE seat_id = ?;  -- Return updated seat information
END
```

**Example input / output:**

```text
Input:
- Booking seat ID: 101

Output:
- Seat booked successfully.
  {
    "seat_id": 101,
    "row": "A",
    "column": 3,
    "booked": true
  }
```

**Data:**

```json
{
  "seat_id": 101,
  "row": "A",
  "column": 3,
  "booked": false
}
```

**Flow:**

1. Seat Available?
2. Seat Not Available?
3. User Initiates Booking
4. Select Seat
5. Lock Selected Seat
6. Check Seat Availability
7. Book Seat & Update Database
8. Return Confirmation
9. Raise Error: Seat AlreadyBooked

