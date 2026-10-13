---
type: "practice"
difficulty: "Basic"
---

# How do you round a numeric column to two decimal places, and how is ROUND different from TRUNCATE of the decimals?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Storing Money](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Storing_Money.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To round a numeric column to two decimal places in SQL, we typically use the ROUND function. The ROUND function rounds the number to the specified number of decimal places and can handle rounding up or down based on the rules of arithmetic. On the other hand, the TRUNCATE function simply removes the digits beyond the specified place value without rounding.

Let's dive into how these functions work with an example.

**Code:**

```

-- Example table creation CREATE TABLE Sales ( id INT PRIMARY KEY, amount DECIMAL(10, 4) );

-- Insert some sample data INSERT INTO Sales (id, amount) VALUES (1, 123.4567), (2, 89.1234);

CODE
Copy

```sql
-- Using ROUND to round the 'amount' column to two decimal places
SELECT id, ROUND(amount, 2) AS rounded_amount FROM Sales;

-- Output:
+----+----------------+
| id | rounded_amount |
+----+----------------+
| 1  |          123.46|
| 2  |            89.12|
+----+----------------+

-- Using TRUNCATE to truncate the 'amount' column to two decimal places
SELECT id, TRUNCATE(amount, 2) AS truncated_amount FROM Sales;

-- Output:
+----+-----------------+
| id | truncated_amount |
+----+-----------------+
| 1  |             123.45|
| 2  |               89.12|
+----+-----------------+
SQL
Copy
-- Example of a more complex query with rounding and truncation
SELECT
    id,
    ROUND(amount, 2) AS rounded_amount,
    TRUNCATE(amount, 2) AS truncated_amount
FROM Sales;
```

