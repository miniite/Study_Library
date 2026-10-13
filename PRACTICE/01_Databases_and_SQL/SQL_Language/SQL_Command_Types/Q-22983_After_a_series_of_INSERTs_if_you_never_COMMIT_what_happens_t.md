---
type: "practice"
difficulty: "Intermediate"
---

# After a series of INSERTs, if you never COMMIT, what happens to the data?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Transaction Control and Prepared Transactions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Transaction_Control_and_Prepared_Transactions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with SQL databases, understanding how transactions and concurrency control work is crucial. If you execute a series of INSERT statements without committing them, those changes will remain in memory (or a transactional log) but won't be visible to other sessions until the transaction is explicitly committed. In this answer, I'll explain what happens under the hood when no COMMIT is issued and discuss some common database management systems like MySQL, PostgreSQL, and SQL Server.

**Code:**

```sql
-- Example in MySQL
START TRANSACTION;
INSERT INTO users (name, email) VALUES ('John Doe', 'john@example.com');
-- No COMMIT statement here
```

**Example input / output:**

```text
Input:
- A series of `INSERT` statements executed within a transaction without committing.
Output:
- Data remains in the database's transaction log or memory but is not visible to other transactions until committed.
```

**Flow:**

1. Database
2. User
3. Database
4. User
5. START TRANSACTION
6. INSERT INTO users (name, email) VALUES ('John Doe', 'john@example.com')
7. [No COMMIT]
8. Data stored in transaction log or memory

