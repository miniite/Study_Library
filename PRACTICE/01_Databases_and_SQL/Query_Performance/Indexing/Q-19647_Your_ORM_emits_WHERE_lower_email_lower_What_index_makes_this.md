---
type: "practice"
difficulty: "Deep"
---

# Your ORM emits WHERE lower(email) = lower(?). What index makes this fast?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Functional Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Functional_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with database queries, optimizing performance is crucial. In this scenario, we need to ensure that the WHERE lower(email) = lower(?) query runs efficiently. The key to achieving this lies in understanding how case-insensitive comparisons work and how to design an appropriate index.

#### Triaging the Question

The question is asking us to identify a suitable database index for a specific type of query: WHERE lower(email) = lower(?). The interviewer is probing our knowledge of:

Case-Insensitive Comparisons: Understanding how databases handle case-insensitive queries.
Index Design and Optimization: Knowing which types of indexes are best suited to speed up complex queries.

#### Assumptions
We are working with a relational database (e.g., PostgreSQL, MySQL).
The email column is likely of type VARCHAR.
Case-insensitive comparisons can be challenging due to the nature of string operations and indexing.

#### Solution
from First Principles

To make this query fast, we need an index that supports case-insensitive searches. Most traditional B-tree indexes are not designed for case-insensitive comparisons because they store data in their exact form. However, there are a few strategies we can employ:

1. Lowercase Index Column

One approach is to create an additional column that stores the lowercase version of the email field and index this new column.

SQL
ALTER TABLE users ADD COLUMN email_lower VARCHAR;
UPDATE users SET email_lower = lower(email);
CREATE INDEX idx_email_lower ON users (email_lower);

With this setup, we can modify our query to directly use the indexed column:

SQL
SELECT * FROM users WHERE email_lower = lower(?);

This approach works well for case-insensitive queries because lower() is applied only once when populating the index and not every time the query runs.

2. Full-Text Search Index

Another advanced option is to use full-text search capabilities, which can handle case-insensitivity naturally:

SQL
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE INDEX idx_email_gin ON users USING gin(to_tsvector('english', lower(email)));

Then, the query would look like this:

SQL
SELECT * FROM users WHERE to_tsvector('english', lower(email)) @@ to_tsquery('english', ?);

#### Closing Summary
and Common Mistakes

The most effective approach here is creating a new column for the lowercase version of email and indexing it. This method ensures that the query performs efficiently without overloading the database with complex operations every time.

#### Common Mistakes
Overusing Full-Text Search: While full-text search can be powerful, it might introduce unnecessary overhead if not strictly necessary.
Ignoring Index Maintenance: Regularly updating indexes and ensuring they remain optimal is crucial to performance.
Not Considering Storage Overhead: Creating additional columns or indexes always comes with a cost in terms of storage.
Depth + Language Matching the Level

Given your seniority as an Architect/Principal, you would be expected to delve deeper into the trade-offs between different approaches and consider factors like scalability and maintenance. Understanding these nuances will help you make informed decisions based on the specific requirements and constraints of the project.

**Comparison:**

```
id	name	email	email_lower
1	John Doe	john.doe@example.com	john.doe@example.com
2	Jane Smith	jane.smith@Example.COM	jane.smith@example.com

Optimized Table with Index:

id	name	email	email_lower
1	John Doe	john.doe@example.com	john.doe@example.com
2	Jane Smith	jane.smith@Example.COM	jane.smith@example.com
```

**Code:**

```sql
-- Example SQL code for creating a case-insensitive index
ALTER TABLE users ADD COLUMN email_lower VARCHAR;
UPDATE users SET email_lower = lower(email);
CREATE INDEX idx_email_lower ON users (email_lower);

-- Full-text search example
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE INDEX idx_email_gin ON users USING gin(to_tsvector('english', lower(email)));
```

**Example input / output:**

```text
Input Query:
SELECT * FROM users WHERE lower(email) = lower('John.Doe@example.com');

Optimized Query with Index (using email_lower column):
SELECT * FROM users WHERE email_lower = 'john.doe@example.com';
```

**Data:**

```json
{
  "users": [
    {"id": 1, "name": "John Doe", "email": "john.doe@example.com"},
    {"id": 2, "name": "Jane Smith", "email": "jane.smith@Example.COM"}
  ]
}
```

**Flow:**

1. No
2. Yes
3. User Query
4. Is Index Created forLower(email)?
5. Query Slow
6. Query Optimized Usingemail_lower Index

