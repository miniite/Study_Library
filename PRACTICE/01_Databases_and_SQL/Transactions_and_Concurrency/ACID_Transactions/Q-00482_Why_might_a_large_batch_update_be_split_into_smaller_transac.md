---
type: "practice"
difficulty: "Intermediate"
---

# Why might a large batch update be split into smaller transactions?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ ACID Durability and Consistency](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/ACID_Durability_and_Consistency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with large batch updates, splitting them into smaller transactions is often a best practice to ensure data integrity and maintain system stability. This approach helps in adhering to ACID properties—Atomicity, Consistency, Isolation, and Durability—by breaking down the process into manageable chunks. In this overview, we'll explore why such a strategy is beneficial, including the underlying principles and common scenarios where it's applied.

**Comparison:**

| ID | Name  | Department |
|----|-------|-----------|
| 1  | Alice | HR        |
| 2  | Bob   | Engineering|
| 3  | Carol | HR        |
| 4  | Dave  | HR        |
| 5  | Eve   | HR        |
| 6  | Frank | HR        |
| 7  | Grace | HR        |
| 8  | Henry | HR        |

**Code:**

```python
# Example code for initiating a transaction in Python with SQLite
import sqlite3

def initiate_transaction():
    conn = sqlite3.connect('example.db')
    cursor = conn.cursor()

    # Begin a transaction
    conn.execute("BEGIN")
    return conn, cursor
```

**Example input / output:**

```text
Input:
- Large batch update of employee department from 'HR' to 'Marketing'

Output (example):
1. Update employees with ids [1] (1 row)
2. Update employees with ids [2,3,4] (3 rows)
3. Update employees with ids [5,6,7,8] (4 rows)
```

**Data:**

```json
{
  "table": {
    "name": "employees",
    "columns": ["id", "name", "department"],
    "rows": [
      {"id": 1, "name": "Alice", "department": "HR"},
      {"id": 2, "name": "Bob", "department": "Engineering"}
    ]
  }
}
```

**Flow:**

1. Small
2. Large
3. Start Transaction
4. Transaction Size?
5. Perform Update
6. Split into Smaller Transactions
7. Perform Each Small Update
8. Commit Transaction
9. Commit Transaction
10. End Transaction

