---
type: "practice"
difficulty: "Expert"
---

# How do you handle inventory reservation during checkout so two users can't buy the last unit — compare pessimistic locking, optimistic reservation, and eventual reconciliation.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Checkout System Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Checkout_System_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

In handling inventory reservations during checkout, ensuring that only one user can purchase the last available unit is crucial to prevent conflicts. This problem can be tackled using three primary approaches: pessimistic locking, optimistic reservation, and eventual reconciliation. Each method has its trade-offs in terms of performance, complexity, and reliability.

#### Triaging the Question

The core of this question revolves around understanding how to manage concurrent access to inventory when multiple users try to purchase a limited resource (such as the last available unit). The interviewer is probing your knowledge of concurrency control techniques and their implications on system design. Key assumptions include:

Multiple users can attempt to buy the same item simultaneously.
The goal is to ensure that only one user completes the purchase.
Step-by-Step Solution from First Principles
1. Pessimistic Locking

Pessimistic locking involves acquiring a lock on the resource (inventory unit) before any operations are performed, thus preventing other users from accessing it until the operation is completed.

#### Implementation

Precondition: The system has to support row-level locks or transactional mechanisms.
Workflow:
User A requests to purchase an item.
The system checks if there's enough inventory and acquires a lock on the unit (e.g., via a database transaction).
If the lock is successful, User A can proceed with the purchase; otherwise, the request is rejected or queued until the lock becomes available.

#### Pros

Ensures data consistency by preventing race conditions.
Guarantees that only one user can make changes at any time.

#### Cons

May lead to performance bottlenecks due to frequent locking and unlocking.
Can result in deadlocks if not managed properly (e.g., circular waits).
2. Optimistic Reservation

Optimistic reservation assumes that conflicts are rare and allows multiple users to proceed with their operations, only checking for conflicts at the end.

#### Implementation

Precondition: The system must allow concurrent reads and updates.
Workflow:
User A requests to purchase an item.
The system checks if there's enough inventory (without acquiring a lock).
If the check passes, User A can proceed with the purchase; otherwise, the request is rejected.

#### Pros

Allows for concurrent operations and better scalability.
Reduces contention by avoiding unnecessary locks.

#### Cons

May lead to data inconsistencies if multiple users try to update the same resource concurrently, necessitating additional conflict resolution logic (e.g., retry mechanisms).
3. Eventual Reconciliation

Eventual reconciliation involves a more relaxed approach where users can attempt to purchase an item without immediate locking, and conflicts are resolved post-purchase.

#### Implementation

Precondition: The system must be able to handle retries and ensure eventual consistency.
Workflow:
User A requests to purchase an item.
The system checks if there's enough inventory (without acquiring a lock).
If the check passes, the purchase is processed immediately.
After the purchase, the system performs post-purchase validation and reconciliation:
If another user also tried to buy the same unit during this time, one of them will fail due to insufficient inventory at the point of purchase.

#### Pros

Highly scalable as it doesn't block operations.
Simplifies concurrency management by deferring conflict resolution.

#### Cons

May lead to race conditions and inconsistent states if not managed carefully.
Requires robust error handling and retry logic.

#### Closing Summary

Each method—pessimistic locking, optimistic reservation, and eventual reconciliation—offers different trade-offs in terms of performance, complexity, and data consistency. Pessimistic locking ensures strong consistency but may introduce bottlenecks, while optimistic reservation and eventual reconciliation offer better scalability at the cost of potential inconsistencies. The choice depends on the specific requirements of the system.

#### Common Mistakes
and Follow-Ups
Mistake: Assuming that optimistic reservation is always better because it avoids locks; this can lead to data corruption if not handled properly.

#### Follow-Up
Discussing scenarios where pessimistic locking might be preferable, such as high-stakes transactions in financial systems, versus situations where eventual reconciliation might suffice for less critical resources.
Triaging Recap

The key takeaway is that the choice of approach depends on the system's requirements and constraints. For high-concurrency environments with frequent conflicts, optimistic reservation or eventual reconciliation may be more suitable, whereas for critical operations, pessimistic locking ensures stronger guarantees but at a higher cost in terms of performance.

**Comparison:**

| Method         | Pros                                         | Cons                             |
|----------------|----------------------------------------------|----------------------------------|
| Pessimistic Locking | Ensures data consistency and prevents race conditions. | May lead to performance bottlenecks and deadlocks. |
| Optimistic Reservation | Allows concurrent operations and better scalability. | Potential for data inconsistencies if not handled properly. |
| Eventual Reconciliation | Highly scalable with no immediate locking. | Risk of race conditions and inconsistent states if not managed carefully. |

**Code:**

```

sequenceDiagram participant UserA as User A participant System as System participant DB as Database UserA->>System: Request to purchase item System->>DB: Check inventory and lock (pessimistic) alt Lock successful DB->>System: Inventory sufficient, proceed with purchase System->>UserA: Purchase successful else Lock not available DB->>System: Cannot acquire lock, retry or wait System->>UserA: Request rejected or queued end

CODE
Copy

```mermaid
sequenceDiagram
    participant UserA as User A
    participant System as System
    participant DB as Database
    UserA->>System: Request to purchase item
    System->>DB: Check inventory (optimistic)
    alt Inventory sufficient
        DB->>System: Proceed with purchase
        System->>UserA: Purchase successful
    else Inventory insufficient
        System->>UserA: Purchase failed due to low stock
    end
Database
System
User A
Database
System
User A
alt
[Another user also bought the
same unit]
[No conflict detected]
alt
[Inventory sufficient]
[Inventory insufficient]
Request to purchase item
Proceed with purchase
Purchase successful
Validate and reconcile post-purchase
Reconciliation fails, rollback one transaction
Purchase failed due to conflict
Purchase successful
```

**Example input / output:**

| Input | Output |
|-------|--------|
| User A requests to purchase an item, System checks inventory and locks the unit. | Purchase successful for User A. |
| User B tries to purchase the same item at the same time, System checks inventory but cannot acquire lock due to User A's reservation. | User B is notified that the purchase has been rejected or queued. |

**Data:**

```json
{
  "inventory": {
    "item_id": "12345",
    "quantity": 10
  },
  "purchase_request": {
    "user_id": "67890",
    "item_id": "12345",
    "quantity": 1
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User A requests purchase
6. Inventory Sufficient?
7. System processes purchase,update inventory
8. Request rejected or queued
9. User notified of failure orqueuing
10. User B requests purchase
11. Inventory Sufficient?
12. System processes purchase,update inventory
13. Request rejected due to lowstock

