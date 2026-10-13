---
type: "practice"
difficulty: "Expert"
---

# When is a distributed lock the wrong abstraction, and when should you instead use a lease, a fencing token, or make the operation idempotent? Give a concrete example for each.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Locks and Fencing](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/Distributed_Locks_and_Fencing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Distributed locks are powerful but not always the best choice in distributed systems. This answer explores when you should avoid using a lock and opt for other solutions like leases, fencing tokens, or idempotent operations.

TL;DR:
Distributed Locks: Use when exclusive access is critical.
Leases: Prefer for short-term needs where expiration helps manage timeouts and liveness.
Fencing Tokens: Optimal for ensuring single instance of an operation in a distributed system.
Idempotent Operations: Ideal when the operation can be retried without affecting outcomes.

#### Triaging the Question

The question seeks to understand when different mechanisms are more appropriate than using a distributed lock. The core is about choosing between exclusive access (locks), temporary control (leases), mutual exclusion tokens (fencing tokens), and ensuring idempotency in operations.

#### Assumptions
Locks: Typically used for critical sections where only one instance should be performing the operation.
Leases: Used when an operation needs to happen, but it's acceptable if it doesn’t occur after a timeout.
Fencing Tokens: Ensures that no two instances can proceed with an operation simultaneously by marking state.
Idempotent Operations: Ensure idempotency through state handling.
Step-by-Step Solution:
1. Distributed Locks

Distributed locks are ideal when you need exclusive access to a resource, such as updating data in a shared database or managing critical sections of code that should not be run concurrently by multiple nodes. However, they can lead to deadlocks and livelocks if misused.

#### Example
Consider a financial transaction system where ensuring that only one node processes a transaction at a time is crucial.

PYTHON
from distributed_lock import DistributedLock

lock = DistributedLock("transaction_id")

with lock.acquire():
    # Perform the transaction logic here
    update_database(transaction_id)
2. Leases

Leases are more suitable for scenarios where you need to perform an operation but it's acceptable if it doesn’t complete within a certain time frame or if multiple nodes can potentially execute the same operation.

#### Example
In a distributed cache invalidation system, you want to invalidate entries in different caches.

PYTHON
from lease import Lease

lease = Lease("cache_key", timeout=30)

if not lease.is_expired():
    # Invalidate the cache entry here
    invalidate_cache_entry(cache_key)
3. Fencing Tokens

Fencing tokens are used when you need to ensure that only one instance can proceed with an operation, and it's critical that no two instances execute the same operation simultaneously.

#### Example
In a distributed coordination system where ensuring single-instance execution is crucial.

PYTHON
from fencing_token import FencingToken

token = FencingToken("operation_id")

if token.acquire():
    # Perform the critical operation here
    execute_critical_operation()
4. Idempotent Operations

Idempotent operations are designed to be safe even if they are retried multiple times, making them ideal for distributed systems where retries might occur.

#### Example
Updating a system status in a log.

PYTHON
def update_system_status(status):
    # Logic to ensure idempotence by checking current state and updating only when necessary
    pass

update_system_status("online")

#### Closing Summary

Choosing the right mechanism depends on your specific use case. Distributed locks are best for exclusive access, leases handle temporary operations with timeouts, fencing tokens ensure single-instance execution, and idempotent operations simplify retry scenarios.

#### Common Mistakes
and Follow-Ups:
Overusing Locks: Not recognizing when a lock is unnecessary or could lead to deadlocks.
Misunderstanding Leases: Assuming that because something times out, it should never be retried.
Ignoring Fencing Tokens: Overlooking the need for mutual exclusion in critical sections of code.
Failing to Ensure Idempotency: Not considering retries and ensuring operations are safe when repeated.

**Comparison:**

```
Mechanism	Use Case
Distributed Locks	Exclusive access, critical sections
Leases	Temporary operations, retries, and liveness
Fencing Tokens	Single-instance execution, mutual exclusion
Idempotent Ops	Safe retries, ensuring operations are idempotent
```

**Code:**

```python
# Example of a distributed lock implementation (simplified)
from distributed_lock import DistributedLock

lock = DistributedLock("transaction_id")

with lock.acquire():
    # Perform the transaction logic here
    update_database(transaction_id)

# Example of a lease implementation (simplified)
from lease import Lease

lease = Lease("cache_key", timeout=30)

if not lease.is_expired():
    # Invalidate the cache entry here
    invalidate_cache_entry(cache_key)

# Example of a fencing token implementation (simplified)
from fencing_token import FencingToken

token = FencingToken("operation_id")

if token.acquire():
    # Perform the critical operation here
    execute_critical_operation()

# Idempotent function example
def update_system_status(status):
    if get_current_status() != status:
        set_new_status(status)
```

**Example input / output:**

```text
Input: Perform transaction with id=123
Output: Transaction processed (exclusive access ensured)

Input: Invalidate cache entry for key=456
Output: Cache invalidated after lease expiration

Input: Execute critical operation with id=789
Output: Operation executed (single-instance guarantee)
```

**Data:**

```json
{
  "lock": {
    "id": "transaction_id",
    "acquire_time": "2023-10-01T14:00:00Z"
  },
  "lease": {
    "key": "cache_key",
    "timeout": 30,
    "expiry_time": "2023-10-01T14:15:00Z"
  },
  "fencing_token": {
    "id": "operation_id",
    "acquired": true
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Identify Use Case
6. Is Exclusive Access Needed?
7. Distributed Locks
8. Is Retrying Allowed?
9. Leases
10. Fencing Tokens
11. Idempotent Operations

