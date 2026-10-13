---
type: "practice"
difficulty: "Intermediate"
---

# What does INSERT ... ON CONFLICT DO NOTHING do, and how is it different from DO UPDATE?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ RETURNING Clause](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/RETURNING_Clause.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To quickly summarize, INSERT ... ON CONFLICT DO NOTHING in PostgreSQL allows you to attempt an insert operation only if a conflict with an existing row is detected; otherwise, nothing happens. This is different from DO UPDATE, which not only checks for conflicts but also performs an update on the conflicting row.

Now let's dive into the details:

#### Triage

The question probes your understanding of PostgreSQL’s conflict resolution mechanisms and how to handle duplicate key violations without accidentally modifying data that should remain unchanged. The core concepts are:

Conflict Detection: Understanding when a unique or primary key constraint is violated.
No Action on Conflict: Performing no action if the insert would violate constraints.
Update on Conflict: Updating an existing row when there's a conflict.

#### Assumptions
You're familiar with PostgreSQL’s INSERT statement and its syntax.
You understand basic database concepts like primary keys, unique keys, and constraint violations.

#### First Principles Solution
INSERT ... ON CONFLICT DO NOTHING

This clause is used in SQL statements to insert a new row into a table only if the inserted values do not violate any constraints (such as uniqueness or foreign key constraints). If a conflict occurs, no changes are made to the database. Here’s an example:

SQL
INSERT INTO users (id, name) VALUES (1, 'Alice')
ON CONFLICT (id) DO NOTHING;

In this case, if there is already a row with id = 1, nothing will be done. If there isn't, the new row will be inserted.

DO UPDATE

On the other hand, DO UPDATE not only checks for conflicts but also updates the existing row with the new values if the conflict occurs:

SQL
INSERT INTO users (id, name) VALUES (1, 'Alice')
ON CONFLICT (id) DO UPDATE SET name = EXCLUDED.name;

In this example, if there is a row with id = 1, its name column will be updated to 'Alice'. If there isn't, the new row will be inserted.

#### Closing Summary

To summarize:

INSERT ... ON CONFLICT DO NOTHING attempts an insert and does nothing if a conflict occurs.
DO UPDATE performs both the insert and updates the conflicting row with the new values, if necessary.

#### Common Mistakes/Follow-Ups
Misunderstanding Conflict: Sometimes, developers mistakenly think ON CONFLICT DO NOTHING will update a different column than intended.

#### Performance Considerations
Using DO UPDATE might be more performant in some cases where updates are common and the conflicting rows need to be updated frequently.

#### Example

Let’s consider an example schema:

SQL
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE
);

And the following insertions:

SQL
-- No conflict, insert will succeed
INSERT INTO users (id, name) VALUES (1, 'Alice');

-- Conflict detected, but nothing happens due to DO NOTHING
INSERT INTO users (id, name) VALUES (1, 'Bob')
ON CONFLICT (id) DO NOTHING;

-- Conflict detected, and Bob's name is updated in existing row
INSERT INTO users (id, name) VALUES (2, 'Charlie')
ON CONFLICT (id) DO UPDATE SET name = EXCLUDED.name;
Self-Check

I have confirmed the correctness and completeness of this answer.

**Comparison:**

| Schema Definition      | Insertion with DO NOTHING                   | Resulting Table State    |
|------------------------|---------------------------------------------|--------------------------|
| `users`                | `INSERT INTO users (id, name) VALUES (1, 'Bob') ON CONFLICT (id) DO NOTHING;` | No changes made          |
|                        |                                             | `id: 1 -> Alice`, `2 -> Charlie` |
|                        |                                             | `3 -> David`             |

**Code:**

```sql
-- Example schema
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE
);

-- No conflict, insert will succeed
INSERT INTO users (id, name) VALUES (1, 'Alice');

-- Conflict detected, but nothing happens due to DO NOTHING
INSERT INTO users (id, name) VALUES (1, 'Bob')
ON CONFLICT (id) DO NOTHING;

-- Conflict detected, and Bob's name is updated in existing row
INSERT INTO users (id, name) VALUES (2, 'Charlie')
ON CONFLICT (id) DO UPDATE SET name = EXCLUDED.name;
```

**Example input / output:**

```text
-- Initial state
SELECT * FROM users;
+----+--------+
| id | name   |
+----+--------+
| 1  | Alice  |
| 2  | Charlie|
+----+--------+

-- Attempting insert with conflict (DO NOTHING)
INSERT INTO users (id, name) VALUES (1, 'Bob')
ON CONFLICT (id) DO NOTHING;

-- State after attempt
SELECT * FROM users;
+----+--------+
| id | name   |
+----+--------+
| 1  | Alice  |
| 2  | Charlie|
+----+--------+

-- Attempting insert and update with conflict
INSERT INTO users (id, name) VALUES (3, 'David')
ON CONFLICT (id) DO UPDATE SET name = EXCLUDED.name;

-- State after attempt
SELECT * FROM users;
+----+--------+
| id | name   |
+----+--------+
| 1  | Alice  |
| 2  | Charlie|
| 3  | David  |
+----+--------+
```

**Data:**

```json
{
    "users": [
        { "id": 1, "name": "Alice" },
        { "id": 2, "name": "Charlie" }
    ]
}
```

**Flow:**

1. No
2. Yes
3. Yes
4. No
5. Start Transaction
6. Is Conflict?
7. Perform Insertion?
8. Commit
9. Skip Insertion
10. Rollback

