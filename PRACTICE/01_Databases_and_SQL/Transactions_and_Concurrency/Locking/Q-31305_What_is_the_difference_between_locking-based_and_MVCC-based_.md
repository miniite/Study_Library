---
type: "practice"
difficulty: "Deep"
---

# What is the difference between locking-based and MVCC-based concurrency control?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ MVCC](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Locking/MVCC.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

The key differences between locking-based and MVCC (Multi-Version Concurrency Control) based concurrency control lie in their approach to managing concurrent transactions, how they handle data isolation levels, and their impact on system performance and scalability.

#### Triaging the Question

This question asks us to compare two primary methods of handling concurrency control in database systems: locking-based and MVCC. The core concept here is understanding how these mechanisms ensure that multiple transactions can run concurrently without interfering with each other's data integrity and consistency. We need to discuss their operational principles, benefits, and trade-offs.

#### First-Principles Solution
Locking-Based Concurrency Control

In a locking-based system:

Transaction Isolation Levels: Typically support lower levels like READ UNCOMMITTED or SERIALIZABLE.
Lock Granularity: Can be at the row level (row-level locks), table level, or even page level.
Performance Impact: May suffer from performance bottlenecks due to excessive lock contention and potential deadlocks.

#### Implementation
Details:
Shared Locks (S-locks): Allow multiple transactions to read a resource simultaneously but prevent any write operations.
Exclusive Locks (X-locks): Permit only one transaction at a time to write, preventing both reads and writes by other transactions.
Multi-Version Concurrency Control (MVCC)

In an MVCC-based system:

Transaction Isolation Levels: Generally support higher levels like REPEATABLE READ or SERIALIZABLE.
Data Versions: Store multiple versions of the same data, allowing each transaction to see a snapshot from its own start time.
Performance Impact: Typically better performance due to reduced lock contention and fewer blocking operations.

#### Implementation
Details:
Timestamps: Each row version is assigned a timestamp or sequence number, indicating when it was inserted or updated.
Read-Only Operations: Non-blocking reads can often be performed by simply scanning the version of data that existed at the transaction's start time.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Locking-based systems are simpler to implement but can suffer from performance issues due to lock contention. MVCC, on the other hand, offers better scalability and concurrency but requires more complex management of multiple versions of data. Both methods have their use cases depending on the specific requirements and constraints of the database environment.

#### Common mistakes
include:

Overlooking the impact of locking on transactional consistency and performance.
Not understanding the trade-offs between different levels of isolation in terms of read/write operations.
Failing to appreciate how MVCC's versioning can lead to increased storage overhead, though modern systems often optimize this efficiently.
Triaging Follow-Ups
Lock Granularity: In what scenarios would you prefer finer-grained locking over coarser-grained locking?
MVCC Storage Overhead: How does the extra storage required by MVCC impact system performance and scalability?
Deadlock Management: What are some strategies to manage deadlocks in a locking-based system?

**Comparison:**

```
Lock Type	Description
Shared	Allows multiple readers.
Exclusive	Only one writer at a time.
```

**Code:**

```python
# Example of implementing an exclusive lock in Python
def acquire_exclusive_lock(resource):
    if not resource.is_locked:
        resource.lock()
        return True
    return False

def release_exclusive_lock(resource):
    resource.unlock()

class ResourceLock:
    def __init__(self, name):
        self.name = name
        self.locked_by = None

    def lock(self):
        self.locked_by = threading.current_thread().name

    def unlock(self):
        if self.locked_by == threading.current_thread().name:
            self.locked_by = None
```

**Example input / output:**

```text
Input:
- Transaction T1: SELECT * FROM table_A FOR UPDATE
Output:
- Resource 'table_A' is locked by transaction T1.
```

**Data:**

```json
{
  "lock": {
    "resource_name": "table_A",
    "locked_by": "transaction_1"
  }
}
```

**Flow:**

1. Yes
2. No
3. Transaction Starts
4. Is Data Locked?
5. Acquire Lock
6. Execute Read/Write Operation
7. Release Lock if Exclusive

