---
type: "practice"
difficulty: "Intermediate"
---

# What does SELECT 1 FROM a table return, and why do people sometimes write it inside an EXISTS check?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Scalar Subqueries](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Scalar_Subqueries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL, SELECT 1 FROM a_table is a query that retrieves no data but returns a constant value (in this case, the integer 1). People often use such queries in EXISTS checks to efficiently determine if any row exists in a table. This approach can be more performant than using other methods like COUNT(*).

#### Triaging the Question

The question is essentially asking two parts:

What does SELECT 1 FROM a_table do?
Why do people use this query inside an EXISTS check?

#### First-Principles Solution
Part 1: Understanding SELECT 1 FROM a_table

When you run the SQL statement SELECT 1 FROM a_table;, it does not return any rows of data but rather returns a single value (1). This is useful when you need to perform operations that require a boolean outcome, such as checking for the existence of data.

Part 2: Using EXISTS Check

The EXISTS clause in SQL checks if there is at least one row in a subquery. When used with SELECT 1, it effectively translates into "does this table have any rows?" The key benefit here is that once the database engine finds the first row, it can stop evaluating further and return quickly.

Here's how you might use it:

SQL
-- Example: Check if the 'employees' table has at least one record.
EXISTS (SELECT 1 FROM employees)

This query will return TRUE as soon as the database engine finds a single row in the employees table, making it efficient and quick.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Using EXISTS (SELECT 1) is an elegant way to check for the existence of rows without fetching unnecessary data. However, there are some common mistakes:

Misusing IN instead of EXISTS: Sometimes people might use IN incorrectly, which can lead to full table scans.
Overusing EXISTS: While efficient in most cases, it's important not to overuse EXISTS where simpler conditions or aggregate functions could suffice.

#### Example Scenario

Imagine you have a large database and want to ensure that the 'orders' table has at least one order before running some processing. Using EXISTS (SELECT 1 FROM orders) is much more efficient than doing:

SQL
-- Inefficient way
(SELECT COUNT(*) > 0 FROM orders)

The first approach stops as soon as it finds a row, whereas the second fetches all rows to count them.

#### Common Mistakes/Follow-Ups
Not understanding when EXISTS is more efficient than other methods.
Overlooking performance implications in complex queries.

**Code:**

```sql
-- Example query using EXISTS with SELECT 1
EXISTS (SELECT 1 FROM employees)
```

