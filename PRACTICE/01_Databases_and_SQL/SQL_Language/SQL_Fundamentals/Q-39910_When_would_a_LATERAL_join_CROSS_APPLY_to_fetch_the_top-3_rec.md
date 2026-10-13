---
type: "practice"
difficulty: "Deep"
---

# When would a LATERAL join (CROSS APPLY) to fetch the top-3 recent orders per customer beat a window-function approach, and what indexing does the lateral subquery require?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Attribution with Window Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Attribution_with_Window_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL, a LATERAL JOIN or CROSS APPLY is particularly useful when you need to perform complex row-by-row operations that cannot be easily achieved with window functions alone. For instance, fetching the top-3 recent orders per customer could benefit from using a LATERAL JOIN, especially in scenarios where your query logic involves intricate calculations or filtering conditions applied to each row of the outer table.

To understand when a lateral join beats a window function approach, we need to consider several factors: query complexity, performance, and data structure. A key advantage of a lateral join is its ability to handle more complex correlated subqueries efficiently. Let's break down the problem step by step from first principles.

**Comparison:**

| CustomerID | OrderDate       | Amount |
|------------|-----------------|--------|
| 1          | 2023-08-10      | 60.00  |
| 1          | 2023-07-05      | 45.00  |
| 1          | 2023-06-01      | 50.00  |
| 2          | 2023-07-10      | 90.00  |
| 2          | 2023-06-20      | 80.00  |
| 2          | 2023-05-15      | 70.00  |

**Code:**

```sql
-- Sample schema for orders table
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT,
    OrderDate DATETIME,
    Amount DECIMAL(10,2)
);
```

**Example input / output:**

```text
-- Window function approach to fetch top-3 recent orders per customer
WITH RankedOrders AS (
    SELECT
        o.CustomerID,
        o.OrderDate,
        o.Amount,
        ROW_NUMBER() OVER (PARTITION BY o.CustomerID ORDER BY o.OrderDate DESC) as RowNum
    FROM Orders o
)
SELECT
    CustomerID,
    OrderDate,
    Amount
FROM RankedOrders
WHERE RowNum <= 3;

-- Lateral join approach to fetch top-3 recent orders per customer
SELECT
    t1.CustomerID,
    t2.OrderDate,
    t2.Amount
FROM Orders t1
CROSS APPLY (
    SELECT TOP 3 OrderDate, Amount FROM Orders WHERE CustomerID = t1.CustomerID ORDER BY OrderDate DESC
) t2;
```

**Data:**

```json
{
  "orders": [
    { "OrderID": 1, "CustomerID": 1, "OrderDate": "2023-06-01", "Amount": 50.00 },
    { "OrderID": 2, "CustomerID": 1, "OrderDate": "2023-07-05", "Amount": 45.00 },
    { "OrderID": 3, "CustomerID": 1, "OrderDate": "2023-08-10", "Amount": 60.00 },
    { "OrderID": 4, "CustomerID": 2, "OrderDate": "2023-05-15", "Amount": 70.00 },
    { "OrderID": 5, "CustomerID": 2, "OrderDate": "2023-06-20", "Amount": 80.00 },
    { "OrderID": 6, "CustomerID": 2, "OrderDate": "2023-07-10", "Amount": 90.00 }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Orders Table
4. Cross Apply Subquery
5. CustomerID
6. Select Top 3 Recent Orders
7. Top-3 Recent Orders
8. No Recent Orders

