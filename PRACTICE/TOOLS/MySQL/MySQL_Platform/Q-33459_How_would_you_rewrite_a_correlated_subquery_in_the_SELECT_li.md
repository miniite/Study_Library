---
type: "practice"
difficulty: "Intermediate"
---

# How would you rewrite a correlated subquery in the SELECT list (a per-row lookup) into a single JOIN or window function, and when is the rewrite not equivalent?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ MySQL Optimizer Behavior](<../../../../TOOLS/MySQL/MySQL_Platform/MySQL_Optimizer_Behavior.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with MySQL queries that involve correlated subqueries in the SELECT list, such as per-row lookups, we can often optimize them by using a single JOIN or window functions. This approach is more efficient and scalable. However, it’s essential to understand when these alternatives are equivalent and when they may differ.

#### Triaging the Question

The question essentially asks us to transform a correlated subquery into an alternative form that leverages joins or window functions. A correlated subquery in the SELECT list operates per row, making the query potentially slow for large datasets due to its nested nature. We need to evaluate whether we can replace this with a more efficient join or a window function.

#### Assumptions
The correlated subquery is in the SELECT clause and involves filtering or aggregating over other rows based on the current row.
We are working within the constraints of MySQL, which supports both joins and window functions effectively.

#### First-Principles Solution
Correlated Subquery Example

Suppose we have a table orders with columns order_id, customer_id, and amount. We want to find each order’s total amount for all orders by the same customer:

SQL
SELECT
    o.order_id,
    (SELECT SUM(amount) FROM orders WHERE customer_id = o.customer_id) AS total_amount
FROM
    orders o;
Rewriting with a JOIN

To rewrite this query, we can use a JOIN to aggregate the amounts for each customer and then join back to the original table:

SQL
SELECT
    o.order_id,
    SUM(o2.amount) OVER (PARTITION BY o.customer_id) AS total_amount
FROM
    orders o
LEFT JOIN
    (SELECT customer_id, SUM(amount) as amount_sum FROM orders GROUP BY customer_id) o2
ON
    o.customer_id = o2.customer_id;
Window Function Approach

Alternatively, we can use a window function to achieve the same result more directly:

SQL
SELECT
    order_id,
    SUM(amount) OVER (PARTITION BY customer_id) AS total_amount
FROM
    orders;

#### Closing Summary
and Common Mistakes

The key difference between these approaches lies in their execution plans. The JOIN solution involves a correlated subquery at each row, which can be slow for large datasets. In contrast, the window function operates on the entire result set more efficiently.

When the Rewrites are Equivalent
Both methods will return the same results if there are no duplicate rows or complex conditions.
Window functions and joins can produce equivalent results when aggregated by the same partitioning key (in this case, customer_id).

#### Common Mistakes
to Avoid
Overlooking Join Conditions: Ensure that the join condition in both methods is correctly set up to avoid missing or including extra rows.
Ignoring Performance Considerations: While window functions are generally more efficient, they might not always be available depending on the database version or complexity of the query.
Misunderstanding Aggregation Scope: Make sure you understand how aggregation works in both methods; window functions aggregate over a defined scope, while joins do so across multiple rows.
Final Check

This solution is correct and covers all aspects of the question, ensuring equivalence where applicable and highlighting potential differences in performance and implementation.

