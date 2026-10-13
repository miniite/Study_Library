---
type: "practice"
difficulty: "Intermediate"
---

# How do you handle NULLs in aggregate and join logic so a report does not silently drop or double-count rows?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ CROSS APPLY and OUTER APPLY](<../../../../TOOLS/SQL_Server/T-SQL/CROSS_APPLY_and_OUTER_APPLY.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with T-SQL, handling NULL values can be tricky, especially when dealing with aggregates and joins. The key is to ensure that NULL values are correctly treated in both aggregate functions and join conditions to avoid silently dropping or double-counting rows. This answer will walk through the core principles of managing NULL values using T-SQL.

**Comparison:**

| SaleID | ProductID | QuantitySold | UnitPrice | OrderDate  |
|--------|----------|-------------|----------|------------|
| 1      | NULL     | 5           | 19.99    | 2023-01-01 |
| 2      | 101      | 3           | 49.99    | 2023-01-01 |
| 3      | 102      | 7           | NULL     | 2023-01-02 |

**Code:**

```sql
-- Example of handling NULLs in a query

CREATE TABLE Sales (
    SaleID INT,
    ProductID INT,
    QuantitySold INT,
    UnitPrice DECIMAL(10, 2),
    OrderDate DATETIME
);

INSERT INTO Sales (SaleID, ProductID, QuantitySold, UnitPrice, OrderDate)
VALUES
    (1, NULL, 5, 19.99, '2023-01-01'),
    (2, 101, 3, 49.99, '2023-01-01'),
    (3, 102, 7, NULL, '2023-01-02');
```

**Example input / output:**

```

-- Example query to calculate total sales

SELECT SUM(QuantitySold * UnitPrice) AS TotalSales FROM Sales;

CODE
Copy

```sql
-- Incorrect handling of NULLs in the join condition
SELECT s.SaleID, p.ProductName
FROM Sales s
JOIN Products p ON s.ProductID = p.ProductID;
```

**Data:**

```json
{
    "Sales": [
        { "SaleID": 1, "ProductID": null, "QuantitySold": 5, "UnitPrice": 19.99, "OrderDate": "2023-01-01" },
        { "SaleID": 2, "ProductID": 101, "QuantitySold": 3, "UnitPrice": 49.99, "OrderDate": "2023-01-01" },
        { "SaleID": 3, "ProductID": 102, "QuantitySold": 7, "UnitPrice": null, "OrderDate": "2023-01-02" }
    ]
}
```

**Flow:**

1. Yes
2. No
3. Sales Table
4. ProductID IS NULL?
5. Use COALESCE to replaceNULL
6. Proceed Normally
7. Handle NULLs in Aggregatesand Joins
8. Total Sales Calculation

