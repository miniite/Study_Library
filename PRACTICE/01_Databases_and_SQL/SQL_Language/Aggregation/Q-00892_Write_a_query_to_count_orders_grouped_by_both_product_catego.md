---
type: "practice"
difficulty: "Intermediate"
---

# Write a query to count orders grouped by both product category and order status.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Counting with Joins](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/Counting_with_Joins.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To write a SQL query that counts orders grouped by both product category and order status, we need to leverage aggregate functions such as COUNT() along with the GROUP BY clause. The key is to ensure that our query accurately groups the data by the specified columns and then performs the count.

#### Triaging the Question

The question is asking for a SQL query to group orders by both product category (category_id) and order status (status). We need to use aggregate functions, specifically COUNT(), to determine how many orders belong to each combination of these two categories. The interviewer might be probing our understanding of SQL syntax, the usage of aggregate functions, and grouping logic.

#### First-Principles Solution
Identify the Relevant Columns: We need columns for product category (category_id) and order status (status).
Use COUNT() for Aggregation: This function will count the number of rows (orders) that match each combination of categories and statuses.
Apply GROUP BY Clause: Group the results by both category_id and status.
Step-by-Step Solution
SQL
SELECT
    category_id,
    status,
    COUNT(*) AS order_count
FROM
    orders
GROUP BY
    category_id,
    status;

#### Closing Summary
and Common Mistakes/Follow-Ups

The query correctly groups the data by category_id and status and counts the number of orders in each group. Some common mistakes include:

Forgetting to use COUNT(*), which could result in incorrect or missing count.
Incorrectly using aggregate functions without grouping, leading to a syntax error or wrong results.

If asked for more details, you might be probed about performance implications or how different databases handle such queries. Understanding the query's execution plan can help optimize and scale it better, especially for large datasets.

**Code:**

```sql
SELECT
    category_id,
    status,
    COUNT(*) AS order_count
FROM
    orders
GROUP BY
    category_id,
    status;
```

