---
type: "practice"
difficulty: "Intermediate"
---

# Explain how a UNIQUE index behaves under concurrent inserts of the same key and what error your app should catch.

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Lock Holding and Optimistic Concurrency](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Lock_Holding_and_Optimistic_Concurrency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: In an intermediate-level database context, understanding how a UNIQUE index behaves under concurrent insertions is crucial to ensure data integrity. When multiple transactions attempt to insert identical keys into a table with a UNIQUE constraint, the database engine will reject one or more of these inserts based on its transaction isolation level and settings. Your application should catch a specific error that indicates a violation of the UNIQUE constraint.

#### Triage
The question revolves around how the database handles concurrent insertions when a UNIQUE index is in place. This involves understanding transactional integrity, specifically ACID properties (Atomicity, Consistency, Isolation, Durability), and error handling in applications interacting with databases. We need to assume that we are working with a standard relational database system like PostgreSQL or MySQL.

#### First-Principles Solution

Unique Constraint and Indexing:

A UNIQUE index ensures that all values in the indexed column(s) must be distinct, preventing duplicate entries.

Concurrency Control:

In a multi-threaded environment, multiple transactions can attempt to insert identical values concurrently. The database engine will serialize these transactions using locking mechanisms (e.g., row-level locks).

Transaction Isolation Levels:

Different isolation levels affect how concurrent transactions are handled:
Read Uncommitted: Can see uncommitted changes.
Read Committed: Sees only committed changes.
Repeatable Read: Ensures the same data is seen by multiple reads in a transaction.
Serializable: Ensures that all transactions can be run independently of each other.

Error Handling:

When an attempt to insert a duplicate value is made, the database will raise an error based on its isolation level and locking strategy.
The common error caught by the application would typically be 23505 in PostgreSQL (ER_DUP_ENTRY in MySQL), which corresponds to a unique constraint violation.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Application Logic: Your application should handle this error gracefully, perhaps by retrying the insert with a different value or informing the user about potential duplicates.
Database Settings: Ensure your database settings are correctly configured for the desired isolation level and locking behavior to manage concurrent transactions effectively.

#### Performance Considerations
Monitor how concurrent inserts impact overall performance; consider using optimistic locking or other strategies if needed.

#### Common Mistakes

Ignoring transaction isolation levels can lead to unexpected behaviors.
Not handling unique constraint violations in your application logic can result in silent failures or inconsistent data.
Overlooking database tuning for high concurrency scenarios can degrade system performance.

**Comparison:**

```
id	name
1	Alice
2	Bob
3	ERROR
4	Charlie
```

**Code:**

```

-- Example SQL query to insert a row with a UNIQUE constraint violation INSERT INTO users (id, name) VALUES (1, 'Alice');

CODE
Copy

```python
# Python code snippet to catch and handle the unique constraint error
import psycopg2

def safe_insert(cursor, user_id, username):
    try:
        cursor.execute("INSERT INTO users (id, name) VALUES (%s, %s)", (user_id, username))
    except psycopg2.IntegrityError as e:
        print(f"Failed to insert: {e}")
```

**Example input / output:**

```

Input:

Attempt to insert {"id": 3, "name": "Alice"} into the users table

Output:

Error caught by application:
CODE
Copy
Failed to insert: unique constraint violation on column 'name'

Request:

Insert a row with duplicate name Response:
Application logs error and retries or informs user.
```

**Data:**

```json
{
  "users": [
    {"id": 1, "name": "Alice"},
    {"id": 2, "name": "Bob"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start Transaction
4. Is ID Duplicate?
5. Fail Insertion, Catch Error
6. Proceed with Insertion
7. Log Error and Retry/InformUser

