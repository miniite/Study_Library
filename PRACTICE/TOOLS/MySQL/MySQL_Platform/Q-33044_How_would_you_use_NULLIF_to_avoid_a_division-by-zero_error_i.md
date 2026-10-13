---
type: "practice"
difficulty: "Intermediate"
---

# How would you use NULLIF to avoid a division-by-zero error in a computed ratio column?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Charset and Collation Migration](<../../../../TOOLS/MySQL/MySQL_Platform/Charset_and_Collation_Migration.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To avoid a division-by-zero error when computing a ratio column in MySQL, we can use the NULLIF function effectively. The core idea is to replace any denominator that would be zero with NULL, thereby ensuring no division by zero occurs and the result of such calculations defaults to NULL.

#### Triaging the Question

The question asks how to avoid a division-by-zero error using NULLIF. Division by zero in SQL can lead to errors or incorrect results. The interviewer is probing your understanding of basic arithmetic operations, error handling, and MySQL functions like NULLIF.

Step-by-Step Solution from First Principles
Understanding NULLIF: NULLIF takes two arguments: if the first argument equals the second, it returns NULL; otherwise, it returns the first argument.
Identifying the Zero Denominator: In a ratio calculation ( \frac{A}{B} ), we need to ensure that ( B \neq 0 ). If ( B = 0 ), then NULLIF(B, 0) will return NULL.
Handling the Result: We use conditional logic or arithmetic functions in SQL to handle this result appropriately.

#### Example Scenario

Suppose you have a table with sales data where you want to compute a ratio of units sold per customer:

SQL
CREATE TABLE Sales (
    id INT,
    customer_id INT,
    units_sold INT
);

You might write a query like the following to calculate the average units sold per customer, while handling division by zero properly:

SQL
SELECT
    customer_id,
    AVG(units_sold) / NULLIF(COUNT(*) OVER (PARTITION BY customer_id), 0) AS avg_units_per_customer
FROM Sales
GROUP BY customer_id;

#### Closing Summary
and Common Mistakes/Follow-Ups

In this example, COUNT(*) OVER (PARTITION BY customer_id) calculates the number of sales records for each customer. Using NULLIF ensures that if there are no sales records for a customer, the division operation does not fail; it simply returns NULL.

#### Common Mistakes

Forgetting to use NULLIF or using incorrect conditions.
Misusing aggregate functions without partitioning by necessary columns.
Ignoring edge cases where both numerator and denominator could be zero, although NULLIF handles this well in most scenarios.

#### Conclusion

Using NULLIF effectively in SQL queries helps you avoid division-by-zero errors and ensures your computations remain robust. Always consider the possibility of null or zero values that might cause such errors.

**Code:**

```sql
SELECT
    customer_id,
    AVG(units_sold) / NULLIF(COUNT(*) OVER (PARTITION BY customer_id), 0) AS avg_units_per_customer
FROM Sales
GROUP BY customer_id;
```

