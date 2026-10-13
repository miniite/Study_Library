---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between the COUNT function returning zero versus returning NULL?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ NULL vs Zero in Aggregates](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/NULL_vs_Zero_in_Aggregates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with SQL, it's essential to understand the nuances of different aggregation functions and how they handle various scenarios, such as when there are no matching rows. The COUNT function is a common example used in database queries, but its behavior can sometimes be confusing, especially for those new to SQL.

The question at hand is: What is the difference between the COUNT function returning zero versus returning NULL? To understand this fully, we need to break down what these two outcomes mean and when they occur.

#### Triaging the Question
Understanding the Core of the Question: The core of the question revolves around understanding how SQL's COUNT function behaves in cases where there are no matching rows.
What the Interviewer is Probing:
Your knowledge of SQL fundamentals, particularly the COUNT function.
Your ability to distinguish between different outcomes and their implications.

#### First-Principles Solution

The COUNT function is an aggregate function in SQL used to count the number of rows that match a specified condition. Here’s how it behaves:

Return Value When Rows Exist: If there are matching rows, the COUNT function will return a non-zero value, typically an integer representing the count.
Return Value When No Rows Match:
Zero (0): In some SQL dialects (like MySQL), if no rows match the condition specified in the WHERE clause, COUNT returns 0.
NULL: In other SQL dialects (like PostgreSQL and SQL Server), if there are no matching rows, COUNT returns NULL.

To illustrate this, consider a simple table named orders:

SQL
CREATE TABLE orders (
    order_id INT,
    customer_name VARCHAR(100),
    order_date DATE
);

#### Example
Queries

Return 0 in MySQL:

SQL
SELECT COUNT(*) FROM orders WHERE order_id = 999;
If there is no order_id with the value 999, this query will return 0.

Return NULL in PostgreSQL and SQL Server:

SQL
SELECT COUNT(*) FROM orders WHERE order_id = 999;
Again, if there is no order_id with the value 999, this query will return NULL.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Key Differences: The primary difference lies in how different SQL dialects handle the absence of rows. MySQL returns a zero, while PostgreSQL and SQL Server return NULL.
Practical Implications:
When using these functions in queries, understanding this behavior can influence your logic and error handling.
For instance, if you're expecting a non-zero value, receiving a NULL might indicate an unexpected condition that needs to be handled differently.

#### Common Mistakes
Assuming Consistency Across Dialects: Assuming that all SQL databases behave the same way regarding COUNT can lead to errors in your logic.
Overlooking NULL Values: Overlooking that a function returning NULL might require different handling compared to 0.

#### Follow-Ups
How would you handle this difference if you were writing a cross-database application?
Can you provide an example of when the behavior might affect your query logic?

**Code:**

```sql
-- Example in MySQL
SELECT COUNT(*) FROM orders WHERE order_id = 999;

-- Example in PostgreSQL and SQL Server
SELECT COUNT(*) FROM orders WHERE order_id = 999;
```

