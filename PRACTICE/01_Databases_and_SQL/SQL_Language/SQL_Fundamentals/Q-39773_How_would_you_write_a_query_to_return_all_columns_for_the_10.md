---
type: "practice"
difficulty: "Basic"
---

# How would you write a query to return all columns for the 10 most recently created users?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Aggregation Query Practice](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Aggregation_Query_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To retrieve all columns of the 10 most recently created users, we need to select from a users table and order by the creation timestamp in descending order. Here’s how you would write such an SQL query step-by-step.

First, let's break down what the question is asking:

We need to return all columns for each user.
The selection should be based on the 10 most recently created users.
All relevant columns from the users table are needed in the output.

We'll assume that there’s a users table with at least the following columns: id, name, email, and created_at. The created_at column is used to determine the order of creation dates.

#### Solution
SQL
SELECT *
FROM users
ORDER BY created_at DESC
LIMIT 10;

This query selects all columns from the users table, orders them by the created_at timestamp in descending order, and limits the result to only the top 10 records. This ensures we get the most recent user data first.

#### Closing Summary

The SQL query above is straightforward and efficient for retrieving the required data. Here are some common mistakes and follow-ups:

Mistakes: Forgetting to use DESC in the ORDER BY clause, or not including a LIMIT statement.

#### Follow-Ups
What if we need pagination?
How would you handle users with the same creation timestamp?
If performance is an issue due to large tables, how can we optimize this query?

**Comparison:**

| id | name | email               | created_at       |
|----|------|---------------------|------------------|
| 5  | Eve  | eve@example.com     | 2023-04-30      |
| 9  | Frank| frank@example.com   | 2023-04-29      |
| 1  | Alice| alice@example.com   | 2023-05-01      |
| 7  | Grace| grace@example.com   | 2023-04-28      |
| 6  | David| david@example.com   | 2023-04-27      |
| 4  | Charlie| charlie@example.com| 2023-04-25      |
| 3  | Betty | betty@example.com   | 2023-04-24      |
| 8  | Adam  | adam@example.com    | 2023-04-22      |
| 2  | Bob   | bob@example.com     | 2023-04-19      |
| 10 | Helen | helen@example.com   | 2023-04-17      |

**Code:**

```sql
SELECT *
FROM users
ORDER BY created_at DESC
LIMIT 10;
```

**Example input / output:**

```text
Input:
SELECT *
FROM users
ORDER BY created_at DESC
LIMIT 10;

Output:
+----+----------+---------------------+------------+
| id | name     | email               | created_at |
+----+----------+---------------------+------------+
| 5  | Eve      | eve@example.com     | 2023-04-30 |
| 9  | Frank    | frank@example.com   | 2023-04-29 |
| 1  | Alice    | alice@example.com   | 2023-05-01 |
| 7  | Grace    | grace@example.com   | 2023-04-28 |
| 6  | David    | david@example.com   | 2023-04-27 |
| 4  | Charlie  | charlie@example.com | 2023-04-25 |
| 3  | Betty    | betty@example.com   | 2023-04-24 |
| 8  | Adam     | adam@example.com    | 2023-04-22 |
| 2  | Bob      | bob@example.com     | 2023-04-19 |
| 10 | Helen    | helen@example.com   | 2023-04-17 |
+----+----------+---------------------+------------+
```

**Data:**

```json
{
  "users": [
    {
      "id": 1,
      "name": "Alice",
      "email": "alice@example.com",
      "created_at": "2023-05-01"
    },
    {
      "id": 2,
      "name": "Bob",
      "email": "bob@example.com",
      "created_at": "2023-04-28"
    },
    // ... more user data
  ]
}
```

**Flow:**

1. Yes
2. No
3. SELECT * FROM users
4. Order by created_at DESC?
5. LIMIT 10
6. Error
7. Return top 10 rows

