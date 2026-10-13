---
type: "practice"
difficulty: "Basic"
---

# What does the RETURNING * clause give you back after a DELETE and why is it handy for confirming what was removed?

**Difficulty:** Basic · **Levels:** Fresher · [↑ RETURNING Clause](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/RETURNING_Clause.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you use RETURNING * in a PostgreSQL DELETE statement, it returns all the rows that were deleted as a result of the operation. This can be very handy because it provides a quick and easy way to confirm what data was removed without needing to query the database separately.

#### Triage
What’s Being Probed

The question is essentially asking you to explain how RETURNING * works in PostgreSQL, why it's useful for post-operation verification, and any potential limitations or considerations. The interviewer might be probing your understanding of SQL operations, particularly delete statements, and your familiarity with PostgreSQL features.

Key Assumptions:
You are familiar with basic SQL commands.
You have some knowledge of the DELETE command in PostgreSQL.
Understanding the concept of transactions and data manipulation is helpful.

#### First Principles Solution

When you issue a DELETE statement followed by RETURNING *, PostgreSQL returns a copy of each row that was deleted. The * in this context means "all columns". This can be incredibly useful because it allows you to inspect the exact rows that were affected without making additional queries.

For example, if you have a table like:

SQL
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50),
    age INT
);

And you execute a DELETE statement:

SQL
DELETE FROM employees WHERE age < 30 RETURNING *;

PostgreSQL will return all the rows that had an age less than 30 and were deleted.

#### Example
Use Case:

Suppose you have this table with some sample data:

SQL
INSERT INTO employees (name, age) VALUES ('Alice', 25), ('Bob', 40), ('Charlie', 29);

Running the delete query would return something like:

PLAINTEXT
 id | name  | age
----+-------+-----
 1  | Alice |  25
 3  | Charlie | 29
Full Solution

The RETURNING * clause can be very handy for several reasons:

Confirmation: It quickly confirms which rows were deleted, helping to catch any unintended deletions.
Error Handling: If a deletion was accidental or based on incorrect criteria, having the exact data helps in troubleshooting.
Audit Logging: In systems that require strict control over data modifications, RETURNING * can serve as an audit log.

#### Closing Summary
and Common Mistakes

Using RETURNING * with DELETE is a simple yet powerful feature of PostgreSQL. It’s especially useful for debugging and ensuring data integrity. However, it's important to note that the returned rows are not deleted from the original table; they are just returned as a result set.

#### Common mistakes
might include:

Forgetting to use RETURNING *, leading to missed confirmation.
Expecting the RETURNING clause to modify the original rows in any way (it doesn't).

#### Example
Code Block

Here’s an example of how you might use this feature in practice:

SQL
-- Sample delete with RETURNING *
DELETE FROM employees WHERE age < 30 RETURNING *;
Self-Check

This answer covers the core functionality, benefits, and potential pitfalls of using RETURNING * in PostgreSQL DELETE statements.

**Comparison:**

| id | name    | age |
|---:|:--------|----:|
|   1| Alice   |   25|
|   3| Charlie |   29|

**Code:**

```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50),
    age INT
);

INSERT INTO employees (name, age) VALUES ('Alice', 25), ('Bob', 40), ('Charlie', 29);

DELETE FROM employees WHERE age < 30 RETURNING *;
```

**Example input / output:**

```text
Input:
DELETE FROM employees WHERE age < 30 RETURNING *;

Output:
 id | name  | age
----+-------+-----
 1  | Alice |  25
 3  | Charlie | 29
```

**Data:**

```plaintext
id | name  | age
----+-------+-----
 1  | Alice |  25
 3  | Charlie | 29
```

**Flow:**

1. Yes
2. No
3. Start Delete
4. Condition Met?
5. Delete Rows and Return
6. End
7. Log Error or Retry
8. End

