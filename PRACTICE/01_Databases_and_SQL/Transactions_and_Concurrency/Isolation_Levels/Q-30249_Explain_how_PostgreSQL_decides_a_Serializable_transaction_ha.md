---
type: "practice"
difficulty: "Expert"
---

# Explain how PostgreSQL decides a Serializable transaction has a 'dangerous structure' (rw-antidependencies) that forces an abort. What are pivot transactions?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Serializable Performance](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/Serializable_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In PostgreSQL, understanding how serializable transactions are handled involves grasping the concept of "rw-antidependencies," which can lead to a transaction being marked as having a 'dangerous structure.' This situation forces an abort because it indicates potential conflicts that could cause non-repeatable reads or phantom reads. Let's break down this complex topic from first principles.

#### Triage

The question asks us to explain how PostgreSQL determines if a serializable transaction has a dangerous structure, specifically focusing on rw-antidependencies and pivot transactions. The core of the question is about understanding the mechanisms used by PostgreSQL to detect and handle potential conflicts in multi-version concurrency control (MVCC) environments.

#### First Principles Solution

PostgreSQL ensures transaction isolation through MVCC, but serializable transactions require a stricter guarantee that no other concurrent transactions can modify data that has been read. This strictness is enforced by checking for rw-antidependencies, which occur when:

A transaction reads a tuple.
Another transaction modifies the same tuple (write).
The original reading transaction then writes to another tuple.

This creates a dependency loop where neither transaction can commit without affecting the other, leading to potential data inconsistencies.

Pivotal Concepts and Examples

#### Cons
ider two transactions T1 and T2:

T1 reads a row with id 1.
Another transaction T3 modifies the same row (writes).
T1 then tries to write another row, creating an rw-antidependency.
Step-by-Step Solution

PostgreSQL uses a technique called "serializable snapshot isolation" to detect these dangerous structures. When a serializable transaction reads data:

It takes a snapshot of the database state at the time of the read.
During writes, it checks if any other transaction has modified the same tuple since the snapshot was taken.

If such an rw-antidependency is detected, PostgreSQL marks the transaction as having a dangerous structure and aborts it to ensure serializability.

Pivot Transactions

A pivot transaction in this context refers to a specific type of transaction that acts as a bridge between other transactions. These are typically created by the system itself during conflict detection processes. For example:

When T1 reads row 1, and T2 writes to row 1 before T1 can write its own data.
PostgreSQL might create an intermediate pivot transaction that represents the conflict.

#### Closing Summary

In summary, PostgreSQL uses sophisticated techniques to ensure serializable transactions do not violate consistency rules. Detecting rw-antidependencies involves complex snapshot and dependency management to prevent dangerous structures that could cause conflicts.

#### Common Mistakes
and Follow-Ups

#### Common mistakes
include misunderstanding how snapshots are taken or ignoring the intricacies of concurrent writes affecting read operations. A deeper dive might explore:

How PostgreSQL manages transaction dependencies.
The exact algorithms used for detecting rw-antidependencies.

#### Trade-offs
between strict serializability and performance in high-concurrency scenarios.

**Comparison:**

```
Serializability	Description
Serializable	Ensures no non-repeatable reads or phantom reads.
Read Uncommitted	Allows dirty reads, not used in PostgreSQL's isolation levels.
Repeatable Read	Ensures that a transaction sees the same data during its execution.
```

**Code:**

```

-- Example of a simple SQL scenario BEGIN; SELECT * FROM my_table WHERE id = 1 FOR UPDATE; -- T1 reads row with id=1 UPDATE my_table SET value = 'new_value' WHERE id = 2; -- T1 tries to write another row

CODE
Copy

```sql
-- Another example with concurrent writes
BEGIN;
SELECT * FROM my_table WHERE id = 1 FOR UPDATE; -- T3 reads row with id=1
UPDATE my_table SET value = 'another_new_value' WHERE id = 1; -- T2 modifies the same row

-- PostgreSQL would detect an rw-antidependency and mark the transaction as dangerous
```

**Example input / output:**

```text
Input:
BEGIN;
SELECT * FROM my_table WHERE id = 1 FOR UPDATE; -- T1 reads row with id=1
UPDATE my_table SET value = 'new_value' WHERE id = 2;

Output (PostgreSQL aborts the transaction due to rw-antidependency):
ERROR: cannot serialize access because statement would cause a conflicting operation

Explanation:
The write by T1 conflicts with the potential future writes by another transaction.
```

**Data:**

```json
{
  "my_table": [
    {"id": 1, "value": "old_value"},
    {"id": 2, "value": "initial_value"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Transaction T1 starts
4. Reads row 1
5. Another transaction T2 writesto row 1
6. Detect rw-antidependency?
7. Abort T1
8. T1 can proceed safely

