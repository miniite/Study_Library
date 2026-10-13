---
type: "practice"
difficulty: "Basic"
---

# Write a query to count the total number of orders placed in a given year.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Counting with Joins](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/Counting_with_Joins.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To answer this question, we need to write a SQL query that uses an aggregate function to count the total number of orders placed in a specified year. The core idea is to filter the orders by the desired year and then use the COUNT function to get the total count.

#### Triage

The interviewer is probing your understanding of basic SQL syntax, filtering data using dates, and applying aggregate functions.
We will assume you have access to a table named orders, which contains at least two columns: order_id (to uniquely identify each order) and order_date (to store the date when the order was placed).

#### First-principles solution

Filter Orders by Year: Use a WHERE clause to filter orders based on the year part of the order_date.
Apply Aggregate Function: Use the COUNT function to count the filtered records.

Here’s the step-by-step SQL query:

SQL
SELECT
    COUNT(order_id) AS total_orders
FROM
    orders
WHERE
    YEAR(order_date) = '2023';

Explanation:

The YEAR() function is used to extract the year part from the order_date.
We are filtering for orders placed in the year 2023.
The COUNT(order_id) counts all rows where the condition is met, effectively giving us the total number of orders for that year.

#### Closing summary
This query correctly returns the total count of orders placed in a given year. A common mistake to avoid is forgetting to use the aggregate function or incorrectly filtering the data, which could lead to incorrect results.

#### Common mistakes/follow-ups

Not using the YEAR() function properly.
Filtering on the full date instead of just the year part.
Misunderstanding what COUNT does and trying to count other columns unintentionally.

**Code:**

```sql
SELECT
    COUNT(order_id) AS total_orders
FROM
    orders
WHERE
    YEAR(order_date) = 2023;
```

