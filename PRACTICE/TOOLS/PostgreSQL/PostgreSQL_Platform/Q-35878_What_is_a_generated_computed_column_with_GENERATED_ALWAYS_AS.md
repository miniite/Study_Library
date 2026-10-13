---
type: "practice"
difficulty: "Deep"
---

# What is a generated (computed) column with GENERATED ALWAYS AS ... STORED, and give a use case.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Materialized CTEs](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Materialized_CTEs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A generated (computed) column in PostgreSQL is a special kind of column that is not physically stored; instead, its value is computed from other columns whenever it's accessed. The GENERATED ALWAYS AS ... STORED clause specifies that the computed value should be saved to disk for faster future retrieval.

The trivia behind generated columns: They are a performance optimization tool. By storing the result of an expression in a column, you can avoid recomputing it each time it's needed. This is particularly useful when dealing with complex or computationally expensive calculations that are frequently queried.

To illustrate this concept and provide a use case:

#### Triage
What the Question is Asking
Core Concept: Understand what generated columns are and why they're used.
Key Idea: Learn about the GENERATED ALWAYS AS ... STORED syntax in PostgreSQL and its benefits.

#### Assumptions
Familiarity with SQL, PostgreSQL, and basic database concepts.

#### First Principles Solution
Definition: A generated column is a virtual column that is not stored on disk but rather computed based on an expression involving other columns or expressions.
Syntax: To create a generated column in PostgreSQL, you use the GENERATED ALWAYS AS ... STORED clause.
Use Case Example: Suppose we have a table storing user data and frequently need to compute a derived value such as the user's age based on their birthdate.
Step-by-Step Solution

Table Definition:

SQL
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    date_of_birth DATE NOT NULL,
    -- Using GENERATED ALWAYS AS ... STORED to store the computed value
    age INT GENERATED ALWAYS AS (EXTRACT(YEAR FROM AGE(date_of_birth))) STORED
);

Insert Data:

SQL
INSERT INTO users (name, date_of_birth) VALUES ('Alice', '1990-05-15');
INSERT INTO users (name, date_of_birth) VALUES ('Bob', '1985-07-30');

Query the Table:

SQL
SELECT id, name, age FROM users;

This query will return both age and other columns.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Summary: Generated columns are a powerful feature in PostgreSQL that allow for performance optimization by storing computed values directly. They are particularly useful when the same derived value is frequently queried.

#### Common Mistakes
Forgetting to use GENERATED ALWAYS AS ... STORED which would make the column non-stored, thus requiring recomputation each time it's accessed.
Overusing generated columns for large tables that do not have frequent access patterns, potentially increasing storage requirements without significant performance benefits.
Real-World Example

In a social media application where user age is often displayed on profiles, storing an age column can significantly reduce the computational load when querying data. However, if users frequently change their birthdate or if ages are infrequently accessed, this might not be worth the extra storage space and maintenance overhead.

**Comparison:**

| id | name  | date_of_birth | age |
|----|-------|---------------|-----|
| 1  | Alice | 1990-05-15    | 32  |
| 2  | Bob   | 1985-07-30    | 37  |

**Code:**

```sql
-- Example of creating a table with a generated column for age
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    date_of_birth DATE NOT NULL,
    -- Generated column to store the computed value
    age INT GENERATED ALWAYS AS (EXTRACT(YEAR FROM AGE(date_of_birth))) STORED
);

-- Inserting data into the table
INSERT INTO users (name, date_of_birth) VALUES ('Alice', '1990-05-15');
INSERT INTO users (name, date_of_birth) VALUES ('Bob', '1985-07-30');

-- Querying the table
SELECT id, name, age FROM users;
```

**Example input / output:**

```

-- Inserting data and querying the table with generated column INSERT INTO users (name, date_of_birth) VALUES ('Alice', '1990-05-15'); INSERT INTO users (name, date_of_birth) VALUES ('Bob', '1985-07-30');

SELECT id, name, age FROM users;

CODE
Copy
Expected Output:
```sql
+----+--------+-----+
| id | name  | age |
+----+-------+-----+
|  1 | Alice | 32  |
|  2 | Bob   | 37  |
+----+-------+-----+
```

**Data:**

```json
[
    {
        "id": 1,
        "name": "Alice",
        "date_of_birth": "1990-05-15"
    },
    {
        "id": 2,
        "name": "Bob",
        "date_of_birth": "1985-07-30"
    }
]
```

**Flow:**

1. Yes
2. No
3. Create Table with GeneratedColumn
4. Is the Derived Value FrequentlyAccessed?
5. Store Computed Values forPerformance
6. Use Regular Columns forSimplicity
7. Optimize Queries by ReducingComputational Load

