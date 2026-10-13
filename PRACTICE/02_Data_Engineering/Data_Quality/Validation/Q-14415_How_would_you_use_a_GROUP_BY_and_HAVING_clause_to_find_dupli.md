---
type: "practice"
difficulty: "Basic"
---

# How would you use a GROUP BY and HAVING clause to find duplicate values in a column?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Uniqueness Checks](<../../../../02_Data_Engineering/Data_Quality/Validation/Uniqueness_Checks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To identify duplicate values in a column using SQL, we can leverage the GROUP BY and HAVING clauses effectively. The core idea is to group rows by the specific column you suspect might have duplicates and then filter those groups where the count of rows exceeds one—thus identifying duplicates.

#### Triage

The question is essentially asking how to use SQL to detect duplicate values in a specific column within a dataset. The GROUP BY clause will help us aggregate data based on the column, while the HAVING clause allows us to set conditions that filter groups based on their size (i.e., number of rows).

Step-by-Step Solution
Identify the Column: Determine which column you want to check for duplicates.
Group By: Use the GROUP BY clause with the identified column.
Count Rows: Add a count function (COUNT()) in the select statement to get the number of rows per group.
Filter Groups: Apply the HAVING clause to filter groups where the count is greater than 1.

#### Example

Suppose we have an orders table with columns order_id, customer_id, and amount. We want to find duplicate customer_id.

SQL
SELECT customer_id, COUNT(*) AS order_count
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 1;

This query will return all customer_ids that have more than one corresponding row in the orders table.

#### Closing Summary

In summary, using a combination of GROUP BY and HAVING clauses allows us to efficiently identify duplicate values in a column. The key steps involve grouping by the target column, counting rows within each group, and then filtering for groups with counts greater than one.

#### Common Mistakes/Follow-Ups
Mistake: Forgetting to use GROUP BY. Without it, you won't be able to aggregate data.

#### Follow-Up
In more complex scenarios, you might want to list the duplicate values explicitly by adding customer_id to the select statement. For example:
SQL
SELECT customer_id, COUNT(*) AS order_count
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 1;

This can be extended to include additional columns for a more detailed analysis.

**Code:**

```sql
-- Example SQL query
SELECT customer_id, COUNT(*) AS order_count
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

