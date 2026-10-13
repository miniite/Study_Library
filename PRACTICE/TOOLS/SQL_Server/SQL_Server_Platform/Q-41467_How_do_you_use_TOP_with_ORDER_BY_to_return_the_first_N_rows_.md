---
type: "practice"
difficulty: "Basic"
---

# How do you use TOP with ORDER BY to return the first N rows in SQL Server?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ OR Conditions and Index Usage](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/OR_Conditions_and_Index_Usage.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To retrieve the first N rows from a table or result set in SQL Server, we can use the TOP clause combined with an ORDER BY clause. The key idea is to order the rows based on a specific column and then limit the number of rows returned using the TOP keyword.

#### Triage

The question asks about using TOP with ORDER BY in SQL Server to return the first N rows from a table or query result. Here, we need to understand that:

TOP is used to restrict the number of rows returned.
ORDER BY helps us define the order of the rows before applying the TOP.

#### First-Principles Solution

Let's break down the process step by step:

Define the Query: Start with a basic SELECT statement that selects columns from a table or views.

Add ORDER BY Clause: Define the column(s) you want to order the results by, and specify whether the ordering should be in ascending (ASC) or descending (DESC) order.

Use TOP Clause: Finally, use the TOP keyword followed by the number of rows you wish to return.

Here’s an example:

SQL
-- Example: Retrieve the top 5 customers based on their total purchase amount

SELECT CustomerID, TotalPurchaseAmount
FROM (
    SELECT CustomerID, SUM(OrderTotal) AS TotalPurchaseAmount
    FROM Orders
    GROUP BY CustomerID
) AS Subquery
ORDER BY TotalPurchaseAmount DESC
TOP 5;

#### Closing Summary
+ Common Mistakes/Follow-Ups

In this example, we first perform a subquery to calculate the total purchase amount for each customer. Then, we use ORDER BY with DESC to order these customers by their total purchase amount in descending order. Finally, TOP 5 restricts the result set to only the top 5 rows.

#### Common Mistakes

Forgetting to include an ORDER BY clause when using TOP.
Using TOP without a limit value (which would return all rows).
Confusing ASC with DESC for sorting direction.

#### Follow-Up
If you were asked about performance considerations, you might discuss indexing on the columns used in ORDER BY and how they can impact query execution time.

**Comparison:**

| CustomerID | TotalPurchaseAmount |
|------------|---------------------|
| 106        | 700                 |
| 105        | 600                 |
| 104        | 500                 |
| 103        | 400                 |
| 102        | 300                 |

**Code:**

```sql
-- Example SQL code to get top 5 customers by purchase amount
SELECT CustomerID, TotalPurchaseAmount
FROM (
    SELECT CustomerID, SUM(OrderTotal) AS TotalPurchaseAmount
    FROM Orders
    GROUP BY CustomerID
) AS Subquery
ORDER BY TotalPurchaseAmount DESC
TOP 5;
```

**Example input / output:**

```text
Input:
SELECT CustomerID, TotalPurchaseAmount
FROM (
    SELECT CustomerID, SUM(OrderTotal) AS TotalPurchaseAmount
    FROM Orders
    GROUP BY CustomerID
) AS Subquery
ORDER BY TotalPurchaseAmount DESC
TOP 5;

Output:
CustomerID | TotalPurchaseAmount
----------|-------------------
106       | 700
105       | 600
104       | 500
103       | 400
102       | 300
```

**Data:**

```json
{
  "Orders": [
    { "CustomerID": 101, "OrderTotal": 200 },
    { "CustomerID": 102, "OrderTotal": 300 },
    { "CustomerID": 103, "OrderTotal": 400 },
    { "CustomerID": 104, "OrderTotal": 500 },
    { "CustomerID": 105, "OrderTotal": 600 },
    { "CustomerID": 106, "OrderTotal": 700 }
  ]
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. Start Query
5. Include ORDER BY?
6. Specify Columns?
7. SELECT * FROM (query) ORDERBY column ASC/DESC TOP N
8. Return All Rows
9. Get Top N Rows

