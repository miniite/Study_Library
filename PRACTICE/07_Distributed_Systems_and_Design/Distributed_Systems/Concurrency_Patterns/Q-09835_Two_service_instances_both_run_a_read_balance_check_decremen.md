---
type: "practice"
difficulty: "Deep"
---

# Two service instances both run a "read balance, check, decrement" flow on the same row and users occasionally overdraft. Walk me through exactly where the race is and three different ways to close it (DB-level, app-level, and design-level).

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Exactly Once via Idempotency](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Exactly_Once_via_Idempotency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are dealing with a common concurrency issue known as "race conditions," particularly when multiple service instances attempt to decrement a shared balance simultaneously. This can lead to overdrafts if not properly managed. Let's break down the race condition and explore three distinct approaches to mitigate it at different levels: database-level, application-level, and design-level.

#### Triage

The core of the problem is that two or more service instances are concurrently trying to decrement a shared balance in a row. If they both read the same initial balance value, then attempt to decrement it without proper synchronization, one might end up with an incorrect final balance, leading to overdrafts.

#### Assumptions
and Key Points
The system uses a single database table for managing balances.
Multiple service instances can access this table concurrently.
We need to ensure that the balance is updated correctly even when multiple instances are accessing it simultaneously.
Detailed Solution
1. Database-Level Approach: Optimistic Locking with Versioning

One way to prevent race conditions is by using optimistic locking via version numbers or timestamps. Each time a service instance reads the balance, it also retrieves the current version number (or timestamp) associated with that row. When updating the balance, it must ensure that the version number has not changed since the last read.

First-Principles:

Read the Balance and Version Number: Service instance 1 reads the balance and its corresponding version.
Decrement the Balance: Service instance 1 attempts to decrement the balance.
Check for Version Conflict: Service instance 1 checks if the version number has changed since it was last read.
Retry or Fail Gracefully: If the version number has changed, service instance 1 retries the operation from step 1.

#### Example
Code:

PYTHON
def update_balance(balance_id):
    # Step 1: Read balance and version
    row = db.select_row_by_id(balance_id)

    if not row:
        return None

    initial_balance = row['balance']
    current_version = row['version']

    # Simulate processing (decrementing the balance)
    new_balance = initial_balance - 50

    try:
        # Step 2: Check and update in a transaction
        with db.transaction():
            updated_row = db.update_row_by_id(balance_id, {'balance': new_balance})

            if updated_row['version'] != current_version:
                raise VersionConflictError("Version conflict detected")

    except VersionConflictError as e:
        return e

# Example usage
update_balance(123)
2. Application-Level Approach: Mutual Exclusion with Locking

A more aggressive approach is to use explicit locking mechanisms at the application level to ensure that only one service instance can read and update the balance at a time.

First-Principles:

Lock the Row: Before accessing the row, acquire an exclusive lock.
Read and Decrement Balance: Perform the decrement operation while holding the lock.
Release Lock: Release the lock after completing the transaction to allow other service instances to access the row.

#### Example
Code:

PYTHON
def update_balance(balance_id):
    # Acquire a shared lock on the balance row
    with db.lock_row_read(balance_id):
        initial_balance = db.get_row_balance(balance_id)

    new_balance = initial_balance - 50

    try:
        # Acquire an exclusive lock and perform the transaction
        with db.lock_row_write(balance_id):
            db.update_row_balance(balance_id, new_balance)
    except LockAcquisitionError as e:
        return e

# Example usage
update_balance(123)
3. Design-Level Approach: Declarative Concurrency Control

A more strategic and scalable approach is to redesign the system to use declarative concurrency control mechanisms. This could involve using distributed locks, leases, or even a dedicated database that handles all balance operations.

First-Principles:

Use a Centralized Service: Design a centralized service that manages all balance operations.
Synchronize Access: Ensure that only one instance can access the central service at a time to update balances.

#### Example
Code:

PYTHON
def update_balance(balance_id):
    # Connect to the centralized service
    with CentralizedService.getConnection() as service:
        new_balance = service.decrement_balance(balance_id, 50)

    return new_balance

# Example usage
update_balance(123)

#### Closing Summary
and Common Mistakes/Follow-Ups

Each of these approaches has its own trade-offs. Optimistic locking is simple but requires careful handling of version conflicts. Application-level locks provide strong guarantees but can introduce performance bottlenecks. Design-level solutions, while more complex to implement, offer the best scalability.

#### Common mistakes
include overlooking the need for synchronization, assuming that a single transactional update will always succeed, and failing to handle versioning correctly in optimistic locking scenarios.

#### Follow-ups
might involve discussing how to measure and mitigate potential lock contention, exploring different lock acquisition strategies (e.g., distributed locks), or delving into more advanced concurrency patterns like leader election algorithms.

**Comparison:**

```
Operation	Optimistic Locking	Application-Level Locking	Design-Level Centralized Service
Complexity	Medium	High	High
Scalability	Good	Poor	Excellent
Performance	Moderate	Low (due to locks)	High
Implementation	Simple but requires versioning	Complex, manages concurrency explicitly	Requires redesign and centralization
```

**Code:**

```

def update_balance(balance_id): # Optimistic Locking Example row = db.select_row_by_id(balance_id)

CODE
Copy
if not row:
    return None

initial_balance = row['balance']
current_version = row['version']

new_balance = initial_balance - 50

try:
    with db.transaction():
        updated_row = db.update_row_by_id(balance_id, {'balance': new_balance})

        if updated_row['version'] != current_version:
            raise VersionConflictError("Version conflict detected")

except VersionConflictError as e:
    return e
CODE
Copy

```python
def update_balance(balance_id):
    # Application-Level Locking Example
    with db.lock_row_read(balance_id):
        initial_balance = db.get_row_balance(balance_id)

    new_balance = initial_balance - 50

    try:
        with db.lock_row_write(balance_id):
            db.update_row_balance(balance_id, new_balance)
    except LockAcquisitionError as e:
        return e
PYTHON
Copy
def update_balance(balance_id):
    # Design-Level Centralized Service Example
    with CentralizedService.getConnection() as service:
        new_balance = service.decrement_balance(balance_id, 50)

    return new_balance
```

**Example input / output:**

```text
# Input
update_balance(123)

# Output
None # If no version conflict

VersionConflictError("Version conflict detected") # If the version has changed
```

**Data:**

```json
{
  "balance_id": 123,
  "balance": 100,
  "version": 1
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Read Balance
6. Version Conflict?
7. Decrement Balance
8. Retry Transaction
9. Lock Row Read
10. Read Balance
11. Version Conflict?
12. Decrement Balance
13. Acquire Write Lock
14. Decrement Balance
15. Release Write Lock
16. Connect to Centralized Service
17. Request Decrement
18. Update Balance
19. Return New Balance

