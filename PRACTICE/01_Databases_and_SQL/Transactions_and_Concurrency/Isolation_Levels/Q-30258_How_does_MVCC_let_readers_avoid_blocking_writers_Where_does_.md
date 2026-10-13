---
type: "practice"
difficulty: "Deep"
---

# How does MVCC let readers avoid blocking writers? Where does the old row version live in PostgreSQL versus Oracle?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Phantoms and Double Booking](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Phantoms_and_Double_Booking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

MVCC allows readers to safely view a snapshot of database states from the past without blocking writers, by managing and storing multiple versions of data. This is achieved through different isolation levels such as Read Uncommitted (RU), Read Committed (RC), Repeatable Read (RR), and Serializable.

In this context, we will focus on how MVCC works in PostgreSQL versus Oracle, particularly in terms of where old row versions are stored and how readers can access these versions without blocking writers. Let's break down the concepts from first principles and then discuss the specifics for each database system.

**Comparison:**

```
Isolation Level	PostgreSQL MVCC Behavior	Oracle MVCC Behavior
Read Uncommitted (RU)	Reads all versions of data, including uncommitted transactions.	Similar behavior, but with more granular control over when uncommitted rows are visible.
Read Committed (RC)	Reads the latest committed version of each row.	Reads only committed data. Rows with uncommitted changes are not seen.
Repeatable Read (RR)	Sees a consistent snapshot of all versions from the beginning of the transaction. Uncommitted writes do not affect the read result.	Uses multiversioning, but may still see uncommitted changes under certain conditions.
Serializable	Ensures serializability by acquiring locks on all rows accessed and keeping them until the end of the transaction.	More restrictive locking mechanism ensures ACID properties at the cost of performance.
```

**Code:**

```
MERMAID
Copy
flowchart TD
    A[Reader Access] --> B{Read Committed (RC)}
    B --> C1[(Reads latest committed version)]
    B --> C2{(Does not see uncommitted writes)}
    A --> D{Serializable}
    D --> E1[(Uses a snapshot of the database state at transaction start time)]
    D --> E2{(Sees all transactions that started before it began)}
    A --> F[Read Uncommitted (RU)]
    F --> G1[(Sees all versions, including uncommitted writes)]
```

**Example input / output:**

```text
No specific input-output examples are required here, but a dry-run scenario can be helpful to illustrate the concepts.
```

**Flow:**

1. Transaction Start
2. (PostgreSQL)
3. Reads Latest CommittedVersion
4. Returns Result
5. (Oracle)
6. (Uses Snapshot at TransactionStart)
7. Returns Result

