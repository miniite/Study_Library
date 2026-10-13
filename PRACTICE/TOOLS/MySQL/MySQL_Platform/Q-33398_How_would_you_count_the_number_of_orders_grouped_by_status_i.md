---
type: "practice"
difficulty: "Basic"
---

# How would you count the number of orders grouped by status in an orders table?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Ordering and Limits](<../../../../TOOLS/MySQL/MySQL_Platform/Ordering_and_Limits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To count the number of orders grouped by their status in a MySQL database, we need to use the GROUP BY clause along with the COUNT() function. This is a common operation in SQL that allows us to aggregate data based on certain criteria.

#### Triage

The question asks for counting orders by status. In terms of what the interviewer might be probing, they are likely interested in understanding your familiarity with basic SQL queries and grouping operations. They may also want to see if you can handle simple aggregation functions like COUNT() effectively.

#### First-Principles Solution
Identify the Table: The table we need is orders.
Determine the Column for Grouping: We will use a column named status in this case.
Use COUNT() to Count Rows: For each unique value in the status column, we want to count how many orders exist.

The SQL query would look like:

SQL
SELECT status, COUNT(*) AS order_count
FROM orders
GROUP BY status;

#### Closing Summary
and Common Mistakes/Follow-Ups:

This query will return a list of all unique statuses in the orders table along with their corresponding count. A common mistake is forgetting to use COUNT(*), which is necessary to perform the aggregation. Another pitfall could be using an incorrect column name or misspelling it, leading to syntax errors.

Self-Check:

Correctness + completeness confirmed.

