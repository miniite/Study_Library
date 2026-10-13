---
type: "practice"
difficulty: "Intermediate"
---

# You need email lookups to be case-insensitive and fast. How would you approach indexing for WHERE lower(email) = ?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Functional Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Functional_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure efficient and case-insensitive email lookups, we need to carefully design our database schema and indexes. The core challenge is that SQL does not natively support case-insensitive comparisons directly in the index, which means we need a workaround or specific strategies depending on the RDBMS being used.

#### Triage
Understanding the Question

The question asks how to implement an efficient query for WHERE lower(email) = 'example@example.com'. This requires us to handle two main aspects:

Case Insensitivity: The lower() function converts the email column to lowercase before comparing.
Performance: Ensuring that this operation is fast, which often means optimizing our database indexing strategy.

#### First-Principles Solution
Step 1: Identify the Indexing Strategy

Since SQL databases typically do not support case-insensitive operations directly in indexes, we need an alternative approach. One common method is to store a normalized version of the email in a separate column that can be indexed for efficient lookups.

Normalization Approach:

Create a new column email_normalized which stores the lowercase version of the email.
Use this column in your index and queries.
Step 2: Schema Design

Let's assume we are using a PostgreSQL database, where we can create an index on the normalized email column:

SQL
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(100),
    email_normalized VARCHAR(100)
);

-- Populate the email_normalized column before creating the index
UPDATE users SET email_normalized = lower(email);

-- Create a btree index on the email_normalized column for faster lookups
CREATE INDEX idx_email_normalized ON users (email_normalized);
Step 3: Query Optimization

With the normalized column and index in place, our query will be optimized as follows:

SQL
SELECT * FROM users WHERE email_normalized = 'example@example.com';

This query leverages the indexed email_normalized column for fast lookups.

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Closing Summary
The approach involves storing a normalized version of the email in a separate column and creating an index on this column. This allows us to perform case-insensitive queries efficiently while ensuring performance is maintained.

#### Common Mistakes

Not Normalizing the Column: Failing to normalize the email column can lead to redundant data storage and less efficient queries.
Indexing Directly on Lowercase Function: While it might seem intuitive, directly indexing a function result like lower(email) can be problematic due to the functional dependency. Instead, we store the normalized value separately.

#### Follow-Ups

#### Performance Considerations
How would this change in a very large dataset with millions of records? Would additional indexes or materialized views be needed?
Storage Overhead: Are there any storage overhead concerns when adding an extra column and index?

**Code:**

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(100),
    email_normalized VARCHAR(100)
);

UPDATE users SET email_normalized = lower(email);

CREATE INDEX idx_email_normalized ON users (email_normalized);
```

**Example input / output:**

```text
-- Populate the table and index
INSERT INTO users (email) VALUES ('Example@example.com');
INSERT INTO users (email) VALUES ('Another.email@domain.com');

-- Query to verify indexing works as expected
SELECT * FROM users WHERE email_normalized = 'example@example.com';
```

**Flow:**

1. Yes
2. No
3. User Inserts
4. Is Email Normalized?
5. Index Created
6. Query Performance Optimal
7. Normalize Emails
8. Create Index

