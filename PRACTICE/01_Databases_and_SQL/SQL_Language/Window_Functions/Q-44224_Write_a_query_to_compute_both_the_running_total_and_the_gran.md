---
type: "practice"
difficulty: "Intermediate"
---

# Write a query to compute both the running total and the grand total in the same result set.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ First Last per Group and Running Totals](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/First_Last_per_Group_and_Running_Totals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to write an SQL query that computes two metrics: the running total (cumulative sum) of a specific column and the overall grand total of the same column. The key is using window functions in SQL, specifically the SUM function with a frame clause.

#### Triaging the Question

The question asks us to generate a result set containing both the running total and the grand total for a given dataset. This involves understanding how window functions work and applying them correctly. We will assume you have a table called sales_data with columns like sale_id, amount, and possibly other details.

#### First Principles Solution
Understand the Data: Let’s assume we have a table named sales_data with the following schema:
PLAINTEXT
sales_data (sale_id, amount)
Running Total: We need to compute the cumulative sum of amount as each row is processed.
Grand Total: This would be the total sum of all amounts in the table.
Writing the Query

Here’s how you can write this query step-by-step:

SQL
SELECT
    sale_id,
    amount,
    SUM(amount) OVER (ORDER BY sale_id ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total,
    SUM(amount) OVER () AS grand_total
FROM sales_data;
Explanation of the Code
Running Total: The SUM(amount) OVER (ORDER BY sale_id ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) part calculates the cumulative sum as rows are processed. It orders by sale_id and includes all previous rows up to the current row.
Grand Total: The SUM(amount) OVER () clause computes the total sum of amount without any ordering or partitioning.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The query correctly returns each record along with its running total and grand total. Common mistakes might include not understanding how window functions work, incorrectly framing the OVER() clause, or misunderstanding the use of frame boundaries (ROWS BETWEEN ...).

#### Example
Dry-Run

Assume we have the following sample data:

SQL
CREATE TABLE sales_data (sale_id INT, amount DECIMAL);
INSERT INTO sales_data VALUES
(1, 100),
(2, 200),
(3, 300),
(4, 400);

The expected output would be:

sale_id	amount	running_total	grand_total
1	100	100	1000
2	200	300	1000
3	300	600	1000
4	400	1000	1000
Self-Check

This solution is correct and complete. It handles the requirements of both running total and grand total effectively.

**Comparison:**

  | 2 | 200
  | 3 | 300
  | 4 | 400

**Code:**

```sql
SELECT
    sale_id,
    amount,
    SUM(amount) OVER (ORDER BY sale_id ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total,
    SUM(amount) OVER () AS grand_total
FROM sales_data;
```

**Example input / output:**

| sale_id | amount | running_total | grand_total |
|---------|--------|---------------|-------------|
| 1       | 100    | 100           | 1000        |
| 2       | 200    | 300           | 1000        |
| 3       | 300    | 600           | 1000        |
| 4       | 400    | 1000          | 1000        |

**Data:**

```plaintext
CREATE TABLE sales_data (sale_id INT, amount DECIMAL);
INSERT INTO sales_data VALUES
(1, 100),
(2, 200),
(3, 300),
(4, 400);
```

**Flow:**

1. Yes
2. Sales Data Table
3. Is there a sale_id?
4. Calculate Running Total
5. Calculate Grand Total
6. Combine Results
7. Return Result Set

