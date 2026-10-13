---
type: "practice"
difficulty: "Deep"
---

# You want to support case-insensitive uniqueness on username at scale. Compare a functional unique index on lower(username) versus a citext column for your team.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Functional Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Functional_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When implementing case-insensitive uniqueness for usernames in a database, we have two primary approaches to consider: using a functional unique index on lower(username) or utilizing a citext column. Both methods serve the purpose of achieving case-insensitive uniqueness but come with different trade-offs and implications.

#### Triage

The question is essentially comparing two strategies for ensuring that usernames are unique regardless of their case, while also considering performance, scalability, and ease of implementation. The interviewer might be probing your understanding of SQL features like lower(), index types, and the behavior of citext columns in PostgreSQL (which was likely implied given the context).

#### Assumptions
We are working with a relational database system, specifically PostgreSQL.
Case-insensitive uniqueness means that "Username" and "username" should be treated as equivalent for uniqueness constraints.
The dataset size is large enough to consider performance implications.

#### First-Principles Solution
1. Functional Unique Index on lower(username)

To create a unique index using a functional expression, we can use the following SQL statement:

SQL
CREATE UNIQUE INDEX idx_unique_username ON users (lower(username));

This approach ensures that every combination of lowercase usernames is treated as unique.

#### Pros

Simple and straightforward to implement.
Works well with standard database engines without requiring special support or types.

#### Cons

Every time the username column changes, the index must be rebuilt because it uses a function on the data. This can lead to performance issues during updates.
The functional dependency means that operations involving this index might be slower due to the additional overhead of the lower() function.
2. Using citext Column

PostgreSQL provides the citext type, which is case-insensitive and behaves like a regular text column but internally converts all insertions and comparisons to lowercase before processing them.

#### Pros

Easier to maintain because you don't need to worry about updating or rebuilding indexes.
No additional function calls are needed for queries and updates, making it generally faster.
Built-in support means better performance and reliability.

#### Cons

Requires PostgreSQL with the citext extension installed.
May have slightly different behavior compared to standard text columns in certain scenarios (e.g., case sensitivity in LIKE clauses).

#### Closing Summary

For most practical purposes, using a citext column is preferable due to its simplicity and performance benefits. However, the choice ultimately depends on your specific requirements and constraints.

#### Common Mistakes/Follow-Ups
Overlooking Index Maintenance: Using functional indexes can lead to unnecessary index rebuilds during updates.
Incorrect Query Performance: Misunderstanding how citext interacts with other PostgreSQL features could lead to suboptimal query performance.
Misalignment of Requirements: Not fully understanding the business requirements for case-insensitive uniqueness might result in choosing a solution that doesn’t meet all needs.

#### Conclusion

By comparing these two approaches, we can make an informed decision based on our specific database environment and use cases. In general, citext is a robust and efficient choice when dealing with case-insensitive uniqueness constraints.

**Comparison:**

| Username | Lowered    |
|----------|------------|
| testUser | testuser   |
| TESTUSER | testuser   |

**Code:**

```sql
-- Functional unique index on lower(username)
CREATE UNIQUE INDEX idx_unique_username ON users (lower(username));

-- citext column for case-insensitive uniqueness
ALTER TABLE users ADD COLUMN username_citext citext;
UPDATE users SET username_citext = lower(username);
CREATE UNIQUE INDEX idx_unique_username_citext ON users (username_citext);

-- Example query using citext
SELECT * FROM users WHERE username_citext = 'TestUser';
```

**Example input / output:**

```text
-- Creating and checking the functional unique index on lower(username)
CREATE UNIQUE INDEX idx_unique_username ON users (lower(username));
SELECT * FROM users WHERE username = 'testuser';

-- Using citext for case-insensitive uniqueness
ALTER TABLE users ADD COLUMN username_citext citext;
UPDATE users SET username_citext = lower(username);
CREATE UNIQUE INDEX idx_unique_username_citext ON users (username_citext);
SELECT * FROM users WHERE username_citext = 'testuser';
```

**Data:**

```json
{
  "users": [
    { "id": 1, "username": "testuser" },
    { "id": 2, "username": "TESTUSER" }
  ]
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[Create Functional Unique Index] --> B1{Is Case-Sensitive?};
4. B1 Yes --> C[Rebuild Index];
5. B1 No --> D[Use Lower Function in Queries];
6. E[Add citext Column] --> F1{Is Case-Sensitive?};
7. F1 Yes --> G[Convert and Update Data];
8. F1 No --> H[Create Unique Index on citext];
9. I[Query with citext] --> J[Return Results]

