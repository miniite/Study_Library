---
type: "practice"
difficulty: "Deep"
---

# Write a query using a subquery in FROM to first compute per-customer totals then filter those above 10000.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Subquery Practice Problems](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Subquery_Practice_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we'll use a subquery within the FROM clause of our main query to calculate the total spending by each customer. Then, in the outer query, we'll filter out customers whose total spending exceeds 10,000. Let's break down the solution step-by-step.

#### Triage

The question is asking for a SQL query that computes per-customer totals using a subquery and then filters those results to include only those with a total greater than 10,000. The key here is understanding how to use a FROM clause subquery effectively.

#### Assumptions
We have a table named orders which contains customer information.
Each row in the orders table represents an order and includes fields like customer_id, amount, etc.
The goal is to group orders by customer_id to calculate the total amount spent per customer.

#### Solution

We'll start with a subquery that calculates the total spending for each customer, then filter the results in the main query. Here’s how we can achieve this:

Subquery: Compute the total spending per customer.
Main Query: Filter out customers whose total spending is greater than 10,000.
Step-by-Step First Principles Solution

Let's write a SQL query that first computes the total spending for each customer using a subquery in the FROM clause and then filters these totals to get only those above 10,000.

SQL
SELECT
    t.customer_id,
    t.total_spending
FROM (
    SELECT
        customer_id,
        SUM(amount) AS total_spending
    FROM
        orders
    GROUP BY
        customer_id
) AS t
WHERE
    t.total_spending > 10000;

#### Closing Summary
and Common Mistakes/Follow-Ups

In this query, the subquery computes the total_spending for each customer_id, which is then used in the outer query to filter out only those customers with a spending greater than 10,000. A common mistake here might be not properly understanding how to use the subquery within the FROM clause.

#### Common Mistakes
Incorrectly writing the subquery or main query.
Not grouping by customer_id in the subquery.
Forgetting to filter the results with a condition like total_spending > 10000.

If you need further assistance, let's discuss potential performance optimizations or consider more complex scenarios where we might want to join additional tables.

**Comparison:**

| id | customer_id | amount |
|----|-------------|--------|
| 1  | 1           | 500.00 |
| 2  | 1           | 600.00 |
| 3  | 2           | 1500.00|
| 4  | 3           | 9000.00|
| customer_id | total_spending |
|-------------|----------------|
| 3           | 9000.00        |

**Code:**

```sql
-- Corrected query for clarity and completeness
SELECT
    t.customer_id,
    t.total_spending
FROM (
    SELECT
        customer_id,
        SUM(amount) AS total_spending
    FROM
        orders
    GROUP BY
        customer_id
) AS t
WHERE
    t.total_spending > 10000;
```

**Example input / output:**

```text
Input:
| id | customer_id | amount |
|----|-------------|--------|
| 1  | 1           | 500.00 |
| 2  | 1           | 600.00 |
| 3  | 2           | 1500.00|
| 4  | 3           | 9000.00|

Output:
| customer_id | total_spending |
|-------------|----------------|
| 3           | 9000.00        |
```

**Data:**

```sql
-- Sample data schema
CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT,
    amount DECIMAL(10,2)
);
```

**Flow:**

1. Yes
2. Yes
3. Yes
4. Yes
5. No
6. No
7. Yes
8. No
9. No
10. Yes
11. Yes
12. No
13. START
14. Is the data schema correct?
15. Create table orders
16. Do you have sample inputdata?
17. Load sample data into orders
18. Does the subquery computetotal spending correctly foreach customer_id?
19. Does the main query filter outtotals greater than 10,000?
20. The solution is correct.
21. Do you need to create a newschema or data?
22. Does the query have properGROUP BY clause in subquery?
23. Is the WHERE condition forfiltering correct?
24. Fix the WHERE condition.

