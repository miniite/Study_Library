---
type: "practice"
difficulty: "Deep"
---

# You're asked to define what 'strong consistency' means in your platform's public API docs. How do you translate isolation-level semantics into promises customers can build on?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Isolation with Read Replicas](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/Isolation_with_Read_Replicas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

Strong consistency in a database context guarantees that all transactions see the same state of data at any point in time. In our platform's API documentation, we need to clearly articulate this guarantee and how it impacts transactional behaviors like RU (Read Uncommitted), RC (Read Committed), RR (Repeatable Read), and Serializable.

#### Triaging the Question

The question is essentially asking for a clear definition of 'strong consistency' that aligns with our isolation levels in the database. We need to ensure that our documentation accurately reflects how these isolation levels behave under strong consistency, providing customers with a robust foundation for building reliable applications.

#### Assumptions

The platform supports various transactional modes (RU, RC, RR, Serializable).
Customers are familiar with basic concepts of transaction management and database consistency.
We aim to provide a detailed yet concise explanation that covers both the theoretical and practical aspects.

#### First Principles Solution

Strong consistency ensures that all transactions see consistent data at any point in time. This means that once a write operation is committed, it will immediately be visible to subsequent reads within the same transaction or any other transaction that starts after the write.

RU (Read Uncommitted):
Behavior: Allows dirty reads where uncommitted writes can be read.
In Strong Consistency Context: Not supported under strong consistency as it violates the principle of ensuring all transactions see consistent data.
Read
Read
RU
Transaction 1
Uncommitted Write
Transaction 2
Strong Consistency

#### Cons
istent Data Only
Isolation Level
RC (Read Committed):
Behavior: Ensures that a transaction can only read committed data.
In Strong Consistency Context: Meets the strong consistency requirement as it guarantees that all reads will see committed data.
Read
Read
RC
Transaction 1
Committed Write
Transaction 2
Strong Consistency

#### Cons
istent Data Only
Isolation Level
RR (Repeatable Read):
Behavior: Ensures that a transaction sees the same data throughout its execution.
In Strong Consistency Context: Also meets the requirement as it provides repeatable reads by preventing uncommitted writes from being read.
Read
Read
RR
Transaction 1
Committed Write
Transaction 2
Strong Consistency

#### Cons
istent Data Only
Isolation Level
Serializable:
Behavior: Ensures that transactions are executed as if they were run serially, without interleaving.
In Strong Consistency Context: Fully supports strong consistency by ensuring that all transactions behave as if they were serialized.
Read
Read
Serializable
Transaction 1
Committed Write
Transaction 2
Strong Consistency

#### Cons
istent Data Only
Isolation Level

#### Closing Summary
+ Common Mistakes/Follow-Ups:

In summary, under strong consistency, all isolation levels (RC, RR, and Serializable) are valid as they ensure that transactions see consistent data. The only level not compatible with strong consistency is RU due to its nature of allowing dirty reads.

#### Common Mistakes

Failing to understand the implications of strong consistency on each isolation level.
Misinterpreting what repeatable read means in a non-strongly consistent context.
Assuming that serializable always requires multiple transactions, whereas it simply enforces strict ordering and visibility rules.

#### Follow-Ups

What happens if we need to balance strong consistency with performance?
How do we handle data races under RC or RR?
What are the trade-offs between using different isolation levels in a highly concurrent environment?
Self-check:

The answer is correct and complete.

**Comparison:**

| Isolation Level | Behavior                         | Strong Consistency Support |
|-----------------|----------------------------------|----------------------------|
| RU              | Dirty Reads allowed             | Not Supported              |
| RC              | Only committed data can be read | Supported                  |
| RR              | Repeatable reads                | Supported                  |
| Serializable    | Serial execution of transactions| Fully Supported           |

**Example input / output:**

```text
Input:
Transaction 1 reads an uncommitted write in RU mode.

Output:
Error: Uncommitted data cannot be read in strong consistency.
```

**Data:**

```json
{
    "transaction1": {
        "readAction": "B[Uncommitted Write]",
        "isolationLevel": "RU"
    },
    "transaction2": {
        "readAction": "B",
        "isolationLevel": "RC"
    }
}
```

**Flow:**

1. Violates Strong Consistency
2. Transaction 1
3. Read Uncommitted
4. Error
5. Transaction 2
6. Read Committed
7. Consistent Data Only
8. Transaction 3
9. Repeatable Read
10. Consistent Data Only
11. Serializable Transaction
12. Serial Execution
13. Fully Consistent

