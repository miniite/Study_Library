---
type: "practice"
difficulty: "Intermediate"
---

# You need case-insensitive uniqueness on email but also fast case-insensitive lookups; compare a citext column, a unique index on lower(email), and a generated normalized column, and which you'd ship.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Indexing Multi-join Queries](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Indexing_Multi-join_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When implementing case-insensitive uniqueness on emails while ensuring fast case-insensitive lookups in PostgreSQL, we need to weigh the trade-offs between using citext columns, a unique index on lower(email), and creating a generated normalized column. Each approach has its strengths and weaknesses, making it crucial to consider performance, maintainability, and query flexibility.

#### Triage
What We’re Probing

The interviewer is probing your understanding of PostgreSQL data types and indexes, as well as your ability to balance performance requirements with constraints like case-insensitivity. You should explain the differences between citext, a unique index on lower(email), and a generated normalized column while considering their implications.

#### First-Principles Solution
citext Column: This data type automatically converts all values inserted into it to lowercase, ensuring that uniqueness is case-insensitive.
Unique Index on lower(email): We create an index on the lower(email) function to maintain a unique constraint while keeping the original email format for lookups.
Generated Normalized Column: A column derived from lower(email) can be used to enforce uniqueness and provide fast case-insensitive lookups.
citext Column

#### Pros
Simplifies data management as it handles normalization internally.
Ensures that all email insertions are automatically converted to lowercase, making the table schema simpler.

#### Cons
Potential performance overhead due to the automatic conversion process.
Lack of flexibility in case-sensitive queries (e.g., searching for a specific case).
Unique Index on lower(email)

#### Pros
Efficient for both uniqueness and fast lookups since the index is on lower(email).
Allows flexible querying without worrying about case sensitivity.

#### Cons
Requires manual management of the normalization, which can lead to potential inconsistencies if not properly handled.
Can be more complex when dealing with partial queries or sorting.
Generated Normalized Column

#### Pros
Provides a clean separation between normalized and original data.
Allows for efficient uniqueness checks and case-insensitive lookups via an index on the generated column.

#### Cons
Additional storage overhead due to maintaining a second column.
Requires careful management of updates, ensuring both columns stay in sync.

#### Closing Summary

Given these trade-offs, we would typically choose between citext for simplicity and performance benefits or a unique index on lower(email) for flexibility. The generated normalized column approach is more complex but offers the best balance if you need to enforce uniqueness while allowing fast case-insensitive lookups in various parts of your application.

#### Common Mistakes/Follow-Ups
Mistake: Overlooking the performance implications of citext and not considering its overhead.

#### Follow-Up
Discuss edge cases such as handling emails with leading/trailing spaces or special characters, ensuring that all solutions account for these scenarios.

**Comparison:**

```
tableUsers = [
    "| id | email                     | email_lower        |",
    "| 1  | John.Doe@example.com      | john.doer@example.com|",
    "| 2  | jane.doe@EXAMPLE.com      | jane.doe@example.com|"
]
```

**Code:**

```sql
-- Example: Creating a table with citext column
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email CITEXT UNIQUE
);

-- Example: Creating a unique index on lower(email)
CREATE INDEX idx_email_lower ON users (lower(email));

-- Example: Generating a normalized column for case-insensitive lookups
ALTER TABLE users ADD COLUMN email_lower TEXT GENERATED ALWAYS AS (LOWER(email)) STORED;

CREATE UNIQUE INDEX idx_email_lower_unique ON users (email_lower);
```

**Example input / output:**

```text
- **Input**: `INSERT INTO users (email) VALUES ('john.doe@EXAMPLE.COM');`
- **Output**: Query succeeds, and the email is automatically converted to lowercase.
```

**Data:**

```json
{
  "users": [
    { "id": 1, "email": "John.Doe@example.com" },
    { "id": 2, "email": "jane.doe@EXAMPLE.com" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User Inserts Email
6. Is Email Unique?
7. Insert into Table
8. Email Exists with DifferentCase?
9. Return Error
10. Convert to Lowercase andInsert

