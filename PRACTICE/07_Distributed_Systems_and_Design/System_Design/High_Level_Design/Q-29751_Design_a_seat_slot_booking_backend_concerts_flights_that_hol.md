---
type: "practice"
difficulty: "Deep"
---

# Design a seat/slot booking backend (concerts/flights) that holds a seat during the payment window and releases it on timeout at high contention — how do you avoid overbooking and hold leaks?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Checkout System Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Checkout_System_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: To design a seat booking backend system, we need to ensure that seats are held during the payment window without leading to overbooking or holding leaks. This involves using locking mechanisms, asynchronous processing, and timeouts to manage concurrent access.

The core idea is to implement a state machine with different states such as "Available," "Locked for Payment," and "Booked." We will use distributed locks to handle contention and ensure that only one user can lock a seat at any given time. Additionally, we need to carefully manage timeouts to release seats if payment is not completed within the specified duration.

#### Triage
The question is asking us to design a robust backend system for seat booking in high-contention scenarios, ensuring that we avoid overbooking and hold leaks. Key aspects include managing concurrent access, implementing timeouts effectively, and using appropriate data structures and locking mechanisms.

#### First-Principles Solution

System States: Define states for each seat - Available, LockedForPayment, Booked.
Concurrency Management: Use distributed locks to manage seat lockings during the payment window.
Timeout Mechanism: Implement a timeout mechanism to release seats if payment is not completed within a specified duration.
Asynchronous Processing: Ensure that the system can handle multiple concurrent requests efficiently.

Here’s a step-by-step solution:

Define Seat States:

Available: The seat is available for booking.
LockedForPayment: The seat has been locked by a user during the payment window but not yet booked.
Booked: The seat has been successfully booked and cannot be modified.

Locking Mechanism:

Use distributed locks to ensure that only one user can lock a seat at any given time.
Implement a key-value store with a unique identifier for each seat, e.g., seat_id, where the value is one of the defined states.

Payment Window Logic:

When a user initiates a booking request:
Attempt to lock the seat by setting its state to LockedForPayment.
If successful, proceed with payment processing.
Upon successful payment completion, set the seat state to Booked.
If payment fails or times out, release the seat and reset its state to Available.

Timeout Handling:

Use a distributed timeout mechanism (e.g., circuit breakers) to monitor payment status.
Configure timeouts for both payment processing and seat lock expiration.

Asynchronous Processing:

Use asynchronous queues (like RabbitMQ or AWS SQS) to handle concurrent booking requests efficiently.
Ensure that the system can scale horizontally by adding more instances as needed.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

#### Common Mistakes
Overlooking concurrency issues, failing to implement robust locking mechanisms, ignoring timeout handling, and not properly managing state transitions.

#### Follow-Ups
Discussing how to handle partial payment failures, exploring advanced failure detection techniques (e.g., heartbeats), and considering the impact of network partitions.

#### Conclusion

By carefully designing a system that manages seat states, uses distributed locks for concurrency control, implements effective timeout mechanisms, and handles asynchronous processing, we can ensure robust seat booking functionality in high-contention scenarios. This design approach balances performance, scalability, and reliability while avoiding overbooking and hold leaks.

**Comparison:**

| Seat ID | Initial State   | Final State       |
|---------|-----------------|------------------|
| 1       | Available       | Booked           |
| 2       | Available       | Not Booked (due to timeout) |
| 3       | LockedForPayment| Not Booked (due to failed payment) |

**Code:**

```python
from distributed_lock import DistributedLock

class SeatBookingBackend:
    def __init__(self):
        self.seat_states = {}  # {seat_id: state}
        self.lock = DistributedLock()

    def book_seat(self, seat_id):
        with self.lock.acquire(seat_id):  # Acquire lock for the seat
            if self.seat_states.get(seat_id) == 'Available':
                self.seat_states[seat_id] = 'LockedForPayment'
                return True
        return False

    def process_payment(self, seat_id):
        if self.seat_states.get(seat_id) == 'LockedForPayment':
            # Simulate payment processing with a timeout
            import time
            time.sleep(5)  # Simulating delay
            if self.is_payment_successful():  # Assume this function checks the payment status
                self.seat_states[seat_id] = 'Booked'
                return True
        return False

    def is_payment_successful(self):
        # Placeholder for actual payment processing logic
        import random
        return bool(random.getrandbits(1))  # Simulating a random success/failure

    def release_seat_on_timeout(self, seat_id):
        if self.seat_states.get(seat_id) == 'LockedForPayment':
            self.seat_states[seat_id] = 'Available'
```

**Example input / output:**

```text
Input:
- User initiates a booking request for seat_id = 1.
- Simulate a successful payment process.

Output:
- Seat state transitions as follows:
  - [Initial] {id: 1, state: "Available"}
  - [LockedForPayment] {id: 1, state: "LockedForPayment"}
  - [Booked] {id: 1, state: "Booked"}

Timeout Handling:
- If payment is not completed within the timeout period (e.g., 5 seconds), seat state reverts to Available.
```

**Data:**

```json
{
  "seats": [
    { "id": 1, "state": "Available" },
    { "id": 2, "state": "Booked" },
    { "id": 3, "state": "LockedForPayment" }
  ]
}
```

**Flow:**

1. Payment Process
2. Yes
3. Success
4. Timeout
5. No
6. Yes
7. No
8. User Initiates Booking
9. Seat Available?
10. Acquire Lock
11. Process Payment
12. Set State to Booked
13. Release Seat
14. Do Nothing
15. Payment Successful?
16. Set State to Booked
17. Release Seat on Timeout

