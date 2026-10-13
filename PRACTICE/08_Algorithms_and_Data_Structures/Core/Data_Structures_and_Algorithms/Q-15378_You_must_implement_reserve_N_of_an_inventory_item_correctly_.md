---
type: "practice"
difficulty: "Deep"
---

# You must implement 'reserve N of an inventory item' correctly under high concurrency without overselling; separate the algorithmic invariant from the locking/CAS mechanism and explain how you avoid a race.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Exactly Once Dedup Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Exactly_Once_Dedup_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the problem of reserving N of an inventory item correctly under high concurrency, we need to ensure that no overselling occurs. This involves maintaining an invariant that guarantees the correct number of items are reserved, while efficiently handling concurrent requests using appropriate synchronization mechanisms.

#### Triaging the Question

The core question is about implementing a robust reservation mechanism for shared resources in a multi-threaded environment. The interviewer is probing your understanding of data structures and algorithms to ensure correctness under concurrency, as well as your ability to design efficient locking strategies.

#### Assumptions
We are working with an inventory system where multiple threads can attempt to reserve items simultaneously.
Each reservation request attempts to reserve N items from the inventory.
The goal is to avoid overselling (i.e., reserving more than what's available).
We need to maintain data consistency across concurrent operations.

#### Solution
Invariant and Locking Mechanism
1. Defining the Algorithmic Invariant

The primary invariant we must uphold is that the inventory should only decrement by N for each reservation request, ensuring no overselling occurs. This can be achieved through atomic operations and careful synchronization.

2. High-Level Approach
Atomic Operation: Use an atomic operation to check if enough items are available.
Reservation Mechanism: If enough items are available, reserve them atomically; otherwise, fail the request.
Concurrency Control: Ensure that these operations are thread-safe using proper locking or compare-and-swap (CAS) mechanisms.
3. Implementation Steps

Atomic Check and Reserve:

PYTHON
def reserve(n_items):
    if inventory >= n_items:  # Check availability atomically
        self.inventory -= n_items  # Decrement inventory atomically
        return True
    else:
        return False

Thread-Safe Mechanism: Use a lock or CAS to ensure that the check and decrement operations are atomic.

4. Detailed Solution
PYTHON
import threading

class InventoryItem:
    def __init__(self, initial_quantity):
        self.inventory = initial_quantity
        self.lock = threading.Lock()

    def reserve(self, n_items):
        with self.lock:  # Ensure the block is thread-safe
            if self.inventory >= n_items:  # Atomic check
                self.inventory -= n_items  # Atomic decrement
                return True
            else:
                return False

#### Closing Summary
and Common Mistakes/Follow-Ups

By separating the algorithmic invariant (checking and decrementing the inventory) from the locking mechanism, we ensure that the operation is safe under high concurrency. The key is to use an atomic check followed by a thread-safe update.

#### Common Mistakes

Failing to use proper synchronization mechanisms, leading to race conditions.
Not considering edge cases where the initial quantity of items might be insufficient for multiple reservation requests.
Overcomplicating the solution with unnecessary complexity or synchronization primitives.

#### Follow-Ups

What happens if you need to handle large-scale distributed systems?
How would you optimize this for a highly concurrent environment?

**Code:**

```python
import threading

class InventoryItem:
    def __init__(self, initial_quantity):
        self.inventory = initial_quantity
        self.lock = threading.Lock()

    def reserve(self, n_items):
        with self.lock:  # Ensure the block is thread-safe
            if self.inventory >= n_items:  # Atomic check
                self.inventory -= n_items  # Atomic decrement
                return True
            else:
                return False
```

**Example input / output:**

```text
# Example usage
inventory = InventoryItem(10)
print(inventory.reserve(5))  # True
print(inventory.reserve(6))  # False (only 5 left, so not enough for another 6)
```

**Flow:**

1. True
2. False
3. Check Inventory Availability
4. Inventory >= N?
5. Decrement Inventory
6. Return False
7. Return True

