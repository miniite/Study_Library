---
type: "practice"
difficulty: "Deep"
---

# Write a query using SUM with a CASE expression to count paid versus unpaid invoices.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Counting with Joins](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/Counting_with_Joins.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to write a SQL query that uses SUM along with a CASE statement to differentiate between paid and unpaid invoices. The core idea is to sum up the counts of invoices based on their payment status.

#### Triaging the Question

The question asks for a query using SUM and a CASE expression to count paid versus unpaid invoices. This involves understanding how to use aggregate functions and conditional logic in SQL. We'll assume that we have an invoices table with at least two columns: invoice_id (to uniquely identify each invoice) and status (indicating whether the invoice is 'paid' or 'unpaid').

#### First-Principles Solution
Identify the Table and Columns: We need to select from an invoices table.
Use SUM with CASE Expression: The SUM function will be used in conjunction with a CASE statement to count the number of paid and unpaid invoices.
Group by Status (Optional): While grouping isn't necessary for this simple aggregation, it could be useful if we wanted to break down the counts by some other column.

Here's how you can construct the query step-by-step:

Select all rows from the invoices table.
Use a CASE statement within an aggregate function (SUM) to count paid and unpaid invoices.
Return the results in a readable format.
Full Solution:
SQL
SELECT
    SUM(CASE WHEN status = 'paid' THEN 1 ELSE 0 END) AS paid_invoices,
    SUM(CASE WHEN status = 'unpaid' THEN 1 ELSE 0 END) AS unpaid_invoices;

#### Closing Summary
and Common Mistakes/Follow-Ups:

This query effectively uses SUM and a CASE expression to count the number of paid and unpaid invoices. The key point is ensuring that the CASE statement correctly differentiates between 'paid' and 'unpaid' statuses.

#### Common Mistakes

Forgetting to use an ELSE 0 in the CASE statement.
Not using a SUM function around the CASE expression, leading to incorrect results or syntax errors.

#### Example

If you run the above query on a sample dataset with the following data:

invoice_id	status
1	paid
2	unpaid
3	paid

The result would be:

paid_invoices	unpaid_invoices
2	1

This indicates that there are two paid invoices and one unpaid invoice.

**Code:**

```sql
SELECT
    SUM(CASE WHEN status = 'paid' THEN 1 ELSE 0 END) AS paid_invoices,
    SUM(CASE WHEN status = 'unpaid' THEN 1 ELSE 0 END) AS unpaid_invoices;
```

**Example input / output:**

```text
+--------------+-----------------+
| paid_invoices| unpaid_invoices |
+--------------+-----------------+
| 2            | 1               |
+--------------+-----------------+
```

**Data:**

| invoice_id | status    |
|------------|-----------|
| 1          | paid      |
| 2          | unpaid    |
| 3          | paid      |

