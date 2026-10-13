---
type: "practice"
difficulty: "Deep"
---

# How would you explain to a peer why a unique index also enforces a data rule, not just speed?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Adding Unique Indexes Safely](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Adding_Unique_Indexes_Safely.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a database environment, an index is a data structure that speeds up query operations by allowing the database engine to retrieve rows from a table more efficiently. A unique index ensures that no duplicate values exist in the indexed column(s), which means it enforces a data rule as well. To understand why this is so, let's break down how indexes work and their dual role.

Firstly, an index organizes the data in such a way that allows for faster retrieval. When you create a unique index on a table, the database engine stores the indexed values in a special structure (e.g., B-tree) that speeds up queries where these columns are involved. Now, to enforce uniqueness, whenever a new value is inserted or updated, the database must check if this value already exists in the index. If it does, an error occurs because of the constraint.

Let's delve into how a unique index works from first principles:

Index Creation: When you create a unique index on a column or columns, say column_name, the database engine creates an internal structure that stores these values in a sorted manner.
Insert/Update Operations: Whenever you insert a new row with data for this column, the engine checks if the value already exists in the index. If it does, the operation fails because of the unique constraint. This check is efficient due to the index's structure.
Deletion and Updates: When updating an existing row, the database engine checks the old and new values to ensure that no duplicates are introduced.

Now, let’s consider a practical example with a schema:

SQL
CREATE TABLE users (
    id INT PRIMARY KEY,
    username VARCHAR(255) UNIQUE NOT NULL,
    email VARCHAR(255)
);

Here, username is indexed as unique. When you attempt to insert or update a record where the username already exists, the database will raise an error due to the constraint.

**Code:**

```sql
-- Example of inserting data into users table
INSERT INTO users (id, username, email) VALUES (1, 'john_doe', 'john@example.com');

-- Attempting to insert a duplicate username
INSERT INTO users (id, username, email) VALUES (2, 'john_doe', 'jane@example.com');
```

**Example input / output:**

```text
-- Successful insertion
INSERT INTO users (id, username, email) VALUES (1, 'john_doe', 'john@example.com');

-- Unsuccessful insertion due to unique constraint violation
ERROR 1062 (23000): Duplicate entry 'john_doe' for key 'username'
```

**Data:**

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    username VARCHAR(255) UNIQUE NOT NULL,
    email VARCHAR(255)
);
INSERT INTO users (id, username, email) VALUES (1, 'john_doe', 'john@example.com');
-- This will fail with a unique constraint violation
INSERT INTO users (id, username, email) VALUES (2, 'john_doe', 'jane@example.com');
```

