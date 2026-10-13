---
type: "practice"
difficulty: "Deep"
---

# What is the difference between READ COMMITTED and REPEATABLE READ isolation levels?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Isolation Levels in Transactions](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Isolation_Levels_in_Transactions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database systems, transaction isolation levels like READ COMMITTED and REPEATABLE READ are crucial in managing how transactions see data from other concurrent transactions. READ COMMITTED is a simpler level where a transaction only sees changes made by transactions that have committed before it starts. In contrast, REPEATABLE READ ensures that once a row is read within a transaction, it will always return the same value until the transaction ends, even if another transaction modifies the data.

To understand these concepts better, let's break them down from first principles and discuss their implications on transaction behavior.

**Comparison:**

```
Isolation Level	Behavior Description	Example Scenario
READ COMMITTED	Transaction sees only committed data.	User1 starts with value=50, User2 updates to 60, User1 sees new value as 60.
REPEATABLE READ	Transaction sees the same data until end.	User1 starts with value=50, User2 updates to 60, User1 always sees original value as 50 until transaction ends.
```

**Code:**

```

sequenceDiagram participant User1 as Transaction 1 participant User2 as Transaction 2 participant DB as Database User1->>DB: SELECT * FROM table WHERE id=1 FOR UPDATE; @ READ COMMITTED alt Another transaction modifies the row before this one commits User2->>DB: UPDATE table SET value=value+10 WHERE id=1; User1->>DB: [reads updated value] @ READ COMMITTED else No modifications by other transactions User1->>DB: reads original value end

CODE
Copy
```mermaid
sequenceDiagram
    participant User1 as Transaction 1
    participant User2 as Transaction 2
    participant DB as Database
    User1->>DB: SELECT * FROM table WHERE id=1 FOR UPDATE; @ REPEATABLE READ
    alt Another transaction modifies the row before this one commits
        User2->>DB: UPDATE table SET value=value+10 WHERE id=1;
        User1->>DB: [reads original value] @ REPEATABLE READ
    else No modifications by other transactions
        User1->>DB: reads updated value only if no changes occurred since initial read
    end
```

**Example input / output:**

```text
**Input (for READ COMMITTED):**
- Initial state: `SELECT * FROM table WHERE id=1 FOR UPDATE;` returns a row with value 50.
- Another transaction updates the value to 60 (`UPDATE table SET value=value+10 WHERE id=1;`).
- The original transaction reads again and sees the updated value of 60.

**Input (for REPEATABLE READ):**
- Initial state: `SELECT * FROM table WHERE id=1 FOR UPDATE;` returns a row with value 50.
- Another transaction updates the value to 60 (`UPDATE table SET value=value+10 WHERE id=1;`).
- The original transaction reads again and sees the initial value of 50, as no changes occurred since its first read.
```

**Data:**

```json
{
  "table": {
    "id": 1,
    "value": 50
  }
}
```

**Flow:**

1. Transaction 1 Starts
2. Initial Read
3. Value = 50
4. No Modifications Yet
5. Read Again
6. Value = 50
7. Commit Transaction
8. Transaction 2 Starts
9. Modifies Value to 60
10. Commits Changes
11. Transaction 1 Reads Again
12. Value = 60 for READCOMMITTED, Value = 50 forREPEATABLE READ

