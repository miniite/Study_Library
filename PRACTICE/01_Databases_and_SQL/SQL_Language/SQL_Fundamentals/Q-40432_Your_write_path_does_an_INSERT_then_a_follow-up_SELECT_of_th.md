---
type: "practice"
difficulty: "Intermediate"
---

# Your write path does an INSERT then a follow-up SELECT of the generated row, but under READ COMMITTED a concurrent process sometimes doesn't see it. How do RETURNING clauses and same-transaction reads eliminate this round-trip and race?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Large Batch Updates](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Large_Batch_Updates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When performing an INSERT operation followed by a SELECT, there's a race condition under certain isolation levels, such as READ COMMITTED. This occurs because the SELECT might run before the INSERT has been committed and visible to other transactions. To address this issue, we can use the RETURNING clause in PostgreSQL (or similar features in other databases) or leverage same-transaction reads to ensure that both operations are part of the same transaction.

To dive into the details, let's first understand the problem and then explore how each solution works.

**Comparison:**

| Solution     | Details                                                                                                                                                                                                 |
|--------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| RETURNING    | The `RETURNING` clause can be used to capture the generated row data directly after an `INSERT`. This ensures that both operations are part of a single transaction.                                               |
| Same-Transaction Reads | By ensuring that both `INSERT` and `SELECT` operations are within the same transaction, we guarantee atomicity and visibility. This approach also helps in reducing round-trips between client-server interactions. |

**Code:**

```sql
-- Example SQL for an INSERT followed by a SELECT under READ COMMITTED isolation level
INSERT INTO orders (customer_id, item) VALUES (1, 'book') RETURNING *;
SELECT * FROM orders WHERE id = (SELECT lastval()); -- lastval() is PostgreSQL-specific function to get the last generated ID
```

**Example input / output:**

```

Consider an example where two transactions are running concurrently:

Transaction A: Starts by inserting a new order.

SQL
Copy
BEGIN;
INSERT INTO orders (customer_id, item) VALUES (1, 'book') RETURNING id; -- Returns the generated ID

Transaction B: Immediately after Transaction A starts, it tries to retrieve the inserted row.

SQL
Copy
SELECT * FROM orders WHERE id = (SELECT lastval()); -- Assumes a single row was just inserted and returns that row

In READ COMMITTED isolation level, Transaction B might not see the changes from Transaction A if the INSERT hasn't been committed by the time the SELECT is executed.

To resolve this issue:

Using RETURNING Clause: The RETURNING clause in PostgreSQL (and similar features like OUTPUT in MySQL) allows capturing the generated row data directly after an INSERT. This ensures that both operations are part of a single transaction and guarantees visibility of the inserted row.
Same-Transaction Reads: Ensuring that both insert and select operations are within the same transaction boundary ensures atomicity and consistency.

Let's examine each solution in more detail:
```

**Data:**

```plaintext
orders table schema:
id SERIAL PRIMARY KEY,
customer_id INT NOT NULL,
item VARCHAR(255) NOT NULL
```

**Flow:**

1. Begin Transaction
2. Insert Row with RETURNINGClause
3. Capture Inserted Data
4. Select Same Data within SameTransaction
5. Ensure Atomicity andConsistency
6. Begin Transaction
7. Insert Row
8. Commit or Rollback
9. Check After Commit Only ifNeeded

