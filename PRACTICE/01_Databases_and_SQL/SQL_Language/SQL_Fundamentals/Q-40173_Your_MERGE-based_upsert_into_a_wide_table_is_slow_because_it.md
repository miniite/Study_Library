---
type: "practice"
difficulty: "Deep"
---

# Your MERGE-based upsert into a wide table is slow because it rewrites unchanged rows. How would you add a change-hash guard so only genuinely changed rows are updated, cutting write amplification?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Large Batch Updates](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Large_Batch_Updates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of writing unnecessary rows in an upsert operation, we can implement a change hash guard using SQL. The idea is to compute a hash for each row and compare it with the previous version to determine if the row has genuinely changed. This way, only rows that have actual changes will be updated, significantly reducing write amplification.

#### Triage

The core of the question is about optimizing an upsert operation by ensuring that only truly altered rows are written to the database. The interviewer is likely probing for your understanding of SQL functions, data integrity, and how to minimize unnecessary writes in a transactional system. We will make assumptions such as the existence of a hash_column to store the computed hash, and we'll consider performance trade-offs.

#### Solution
Compute the Hash: For each row, compute a hash that captures all fields except for the primary key or any immutable columns.
Compare Hashes: Compare the current row's hash with the stored hash in the database to determine if the row has changed.
Update Only Changed Rows: If the hashes match, skip the update; otherwise, perform the upsert.
Step-by-Step Solution

Let's assume we have a users table with columns: id, name, email, age, and hash_column. We'll use a hash function to generate the hash value based on name, email, and age.

SQL
-- Step 1: Create a hash function (example using MD5)
CREATE OR REPLACE FUNCTION compute_hash(name VARCHAR, email VARCHAR, age INT)
RETURNS VARCHAR AS $$
SELECT md5(concat_ws('|', name, email, age::text));
$$ LANGUAGE plpgsql;

-- Step 2: Add the computed hash to each row
UPDATE users SET hash_column = compute_hash(name, email, age);

-- Step 3: Implement the upsert with a change hash guard
WITH updated_users AS (
    SELECT id, name, email, age, compute_hash(name, email, age) AS new_hash
    FROM users WHERE (name, email, age) != (old_name, old_email, old_age)
),
upsert_query AS (
    INSERT INTO users (id, name, email, age, hash_column)
    SELECT id, name, email, age, new_hash
    FROM updated_users
    ON CONFLICT (id) DO UPDATE
    SET name = EXCLUDED.name,
        email = EXCLUDED.email,
        age = EXCLUDED.age,
        hash_column = EXCLUDED.new_hash
    WHERE users.hash_column != EXCLUDED.new_hash -- Only update if the hash has changed
)
SELECT * FROM upsert_query;

#### Closing Summary
and Common Mistakes/Follow-Ups

This approach ensures that only rows with actual changes are updated, reducing write amplification. However, there are potential pitfalls:

Hash Function Choice: Ensure the chosen hash function is robust enough to minimize false positives.
Performance Impact: Compute-intensive hash functions can slow down updates; consider caching or indexing strategies.
Concurrency Issues: Handling concurrent updates requires careful consideration of transactions and isolation levels.

In a real-world scenario, you might want to measure the performance impact before rolling out this solution in production. Additionally, consider logging changes for audit purposes.

#### Common Mistakes
Failing to account for hash collisions, leading to incorrect updates.
Overlooking the need to handle concurrent updates properly.
Ignoring the potential overhead of computing hashes on every row.

**Comparison:**

| id | name  | email          | age | hash_column           |
|----|-------|----------------|-----|-----------------------|
| 1  | Alice | alice@example.com | 25  | d41d8cd98f00b204e9800998ecf8427e |
| 2  | Bobby | bob@example.com     | 31  | c5eac0638c7b622d6fa3adbe320ee12f |

**Code:**

```

CREATE OR REPLACE FUNCTION compute_hash(name VARCHAR, email VARCHAR, age INT) RETURNS VARCHAR AS $$ SELECT md5(concat_ws('|', name, email, age::text)); $$ LANGUAGE plpgsql;

CODE
Copy

```sql
WITH updated_users AS (
    SELECT id, name, email, age, compute_hash(name, email, age) AS new_hash
    FROM users WHERE (name, email, age) != (old_name, old_email, old_age)
),
upsert_query AS (
    INSERT INTO users (id, name, email, age, hash_column)
    SELECT id, name, email, age, new_hash
    FROM updated_users
    ON CONFLICT (id) DO UPDATE
    SET name = EXCLUDED.name,
        email = EXCLUDED.email,
        age = EXCLUDED.age,
        hash_column = EXCLUDED.new_hash
    WHERE users.hash_column != EXCLUDED.new_hash -- Only update if the hash has changed
)
SELECT * FROM upsert_query;
```

**Example input / output:**

```sql
-- Initial state of users table
INSERT INTO users (id, name, email, age) VALUES
(1, 'Alice', 'alice@example.com', 25),
(2, 'Bob', 'bob@example.com', 30);

-- Change the data for Bob and recompute hashes
UPDATE users SET name = 'Bobby', age = 31 WHERE id = 2;
SQL
Copy
WITH updated_users AS (
    SELECT id, name, email, age, compute_hash(name, email, age) AS new_hash
    FROM users WHERE (name, email, age) != ('Bob', 'bob@example.com', 30)
),
upsert_query AS (
    INSERT INTO users (id, name, email, age, hash_column)
    SELECT id, name, email, age, new_hash
    FROM updated_users
    ON CONFLICT (id) DO UPDATE
    SET name = EXCLUDED.name,
        email = EXCLUDED.email,
        age = EXCLUDED.age,
        hash_column = EXCLUDED.new_hash
    WHERE users.hash_column != EXCLUDED.new_hash -- Only update if the hash has changed
)
SELECT * FROM upsert_query;

Result: Only the row for Bob is updated, as his data actually changed.
```

**Data:**

```json
{
  "users": [
    {
      "id": 1,
      "name": "Alice",
      "email": "alice@example.com",
      "age": 25,
      "hash_column": "d41d8cd98f00b204e9800998ecf8427e" // Example hash
    },
    {
      "id": 2,
      "name": "Bob",
      "email": "bob@example.com",
      "age": 30,
      "hash_column": "d41d8cd98f00b204e9800998ecf8427e" // Example hash
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Compute Hash
4. Is hash changed?
5. Update row
6. Skip update
7. Record change in logs
8. Continue processing

