---
type: "practice"
difficulty: "Deep"
---

# What is the difference between COUNT of a foreign key column and COUNT(*) in a joined query?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ COUNT Semantics](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/COUNT_Semantics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL, both COUNT(column_name) and COUNT(*) are aggregate functions used to count rows, but they differ based on how they handle NULL values. The COUNT(column_name) function counts all non-NULL values in the specified column, while COUNT(*) returns the total number of rows, including those with NULL values.

To understand this better, let's break down the core concepts and then delve into a practical example.

**Code:**

```sql
-- Example Schema
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT,
    ProductID INT
);

INSERT INTO Orders (OrderID, CustomerID, ProductID) VALUES
(1, 101, 201),
(2, 101, NULL),
(3, 102, 202);
```

**Example input / output:**

```text
-- Query with COUNT(column_name)
SELECT COUNT(ProductID) AS ProductCount FROM Orders;

-- Output:
+------------+
| ProductCount |
+------------+
| 2           |
+------------+

-- Query with COUNT(*)
SELECT COUNT(*) AS TotalRows FROM Orders;

-- Output:
+-----------+
| TotalRows |
+-----------+
| 3         |
+-----------+
```

**Flow:**

1. Yes
2. No
3. COUNT(ProductID)
4. Is ProductID NULL?
5. Return 0
6. Return 1
7. COUNT(*)
8. Return 1 for every row

