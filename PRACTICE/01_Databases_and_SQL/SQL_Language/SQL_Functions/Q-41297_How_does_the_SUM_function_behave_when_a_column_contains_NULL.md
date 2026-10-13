---
type: "practice"
difficulty: "Basic"
---

# How does the SUM function behave when a column contains NULL values?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ NULL vs Zero in Aggregates](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/NULL_vs_Zero_in_Aggregates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL, the SUM function is used to calculate the total sum of all non-NULL numeric values in a specified column. However, it behaves specifically with respect to NULL values. When a column contains NULLs, they are treated as zeros for the purpose of the summation. This behavior might seem intuitive at first glance, but understanding how this impacts your data and query results is crucial.

**Code:**

```sql
-- Example table creation
CREATE TABLE sales (
    id INT,
    amount DECIMAL(10, 2)
);

-- Insert some data with NULLs included
INSERT INTO sales (id, amount) VALUES
(1, 100.00),
(2, 200.00),
(NULL, 300.00), -- This entry will be treated as zero in SUM calculations
(4, 400.00);
```

**Example input / output:**

```text
-- Example query to use the SUM function
SELECT SUM(amount) AS total_sales FROM sales;
-- Output:
-- total_sales
-- 700.00

-- If we modify one of the entries to NULL explicitly
UPDATE sales SET amount = NULL WHERE id = 4;

-- Re-run the query after modification
SELECT SUM(amount) AS total_sales FROM sales;
-- New output:
-- total_sales
-- 600.00
```

**Data:**

```plaintext
+----+--------+
| id | amount |
+----+--------+
| 1  | 100.00 |
| 2  | 200.00 |
| NULL | NULL    |
| 4  | 400.00 |
+----+--------+
```

