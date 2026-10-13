---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you would use a LEFT JOIN plus IS NULL to find users who have never made a purchase for a negative training class.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ NULL Handling](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/NULL_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to identify users who do not have any corresponding entries in the purchases table. We can achieve this by using a LEFT JOIN between the users and purchases tables, followed by filtering out users whose purchase records are NULL. This approach ensures that we capture all users even if they don't have any associated purchases.

#### Triaging the Question

The question is essentially asking how to find users who haven't made a single purchase. In SQL terms, this means finding users in the left table (users) that do not have matching records in the right table (purchases).

#### Assumptions
We have two tables: users and purchases.
The users table contains user information.
The purchases table contains purchase details, including a foreign key linking to users.

#### First-Principles Solution
Perform a LEFT JOIN between the users and purchases tables on the appropriate columns (e.g., user ID).
Filter the result set to include only those rows where the purchase record is NULL.

This can be achieved with the following SQL query:

SQL
SELECT u.*
FROM users u
LEFT JOIN purchases p ON u.user_id = p.user_id
WHERE p.purchase_id IS NULL;

#### Closing Summary
and Common Mistakes/Follow-Ups

The SQL query above correctly identifies all users who have never made a purchase. However, there are some common pitfalls to be aware of:

Mistake: Using JOIN instead of LEFT JOIN. If you use an inner join or simple join, it will only return users with at least one purchase, which is not what we want.

#### Follow-Up
Discuss potential indexing on the foreign keys to optimize performance if the tables are large.

#### Example
SQL Execution

Assume the following sample data:

users:

user_id	name
1	Alice
2	Bob
3	Charlie

purchases:

purchase_id	user_id	product_name
1	1	Book
2	1	Pen

Running the query:

SQL
SELECT u.*
FROM users u
LEFT JOIN purchases p ON u.user_id = p.user_id
WHERE p.purchase_id IS NULL;

The result would be:

user_id	name
2	Bob
3	Charlie
Self-Check

This solution is both correct and complete, covering the core concept of LEFT JOIN with IS NULL to find non-matching records.

**Code:**

```sql
SELECT u.*
FROM users u
LEFT JOIN purchases p ON u.user_id = p.user_id
WHERE p.purchase_id IS NULL;
```

**Example input / output:**

```text
**Input:**
- `users` table:
| user_id | name       |
|---------|------------|
| 1       | Alice      |
| 2       | Bob        |
| 3       | Charlie    |

- `purchases` table:
| purchase_id | user_id | product_name |
|-------------|---------|--------------|
| 1           | 1       | Book         |
| 2           | 1       | Pen          |

**Output:**
| user_id | name      |
|---------|-----------|
| 2       | Bob       |
| 3       | Charlie   |
```

