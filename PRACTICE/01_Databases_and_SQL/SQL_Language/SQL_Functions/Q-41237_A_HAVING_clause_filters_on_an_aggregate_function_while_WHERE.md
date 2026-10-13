---
type: "practice"
difficulty: "Deep"
---

# A HAVING clause filters on an aggregate function while WHERE could filter earlier. Explain the logical processing order and why moving a predicate from HAVING to WHERE changed the result AND performance.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ UDF Performance](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/UDF_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL, the HAVING clause is used in conjunction with GROUP BY to apply conditions after data has been aggregated. Conversely, the WHERE clause filters rows before any grouping occurs. Moving a predicate from HAVING to WHERE can significantly change both the result set and performance due to differences in logical processing order.

TL;DR
Logical Processing Order: WHERE -> GROUP BY (with HAVING) -> SELECT.
Impact of Moving Predicates: Changing where a condition is applied can alter which rows are grouped, leading to different results. It can also impact performance by affecting the order in which data is processed.

#### Triage

The question asks you to explain the logical processing order of SQL statements and how moving a predicate from HAVING to WHERE affects both the result set and performance. The key points include understanding the difference between WHERE and HAVING, recognizing the impact on grouping, and analyzing performance implications.

#### First-Principles Solution

Logical Processing Order:

WHERE: Filters rows based on conditions before any data is grouped.
GROUP BY: Groups rows by specified columns (or expressions).
HAVING: Filters groups after aggregation operations have been performed using the GROUP BY clause.

#### Example Scenario
Let's consider a database table orders with columns customer_id, order_date, and amount. We want to find customers who ordered more than 10 times in a given month.

Initial Query with WHERE and HAVING:

SQL
SELECT customer_id, COUNT(*) as order_count
FROM orders
WHERE order_date >= '2023-10-01' AND order_date < '2023-11-01'
GROUP BY customer_id
HAVING COUNT(*) > 10;

Explanation:

First, WHERE filters the rows to include only those where order_date is within the specified month.
Then, GROUP BY groups by customer_id.
Finally, HAVING ensures that we only see customers who have more than 10 orders in the filtered period.

Moving Predicate from HAVING to WHERE:

SQL
SELECT customer_id, COUNT(*) as order_count
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 10 AND order_date >= '2023-10-01' AND order_date < '2023-11-01';

Impact on Results:

By moving the WHERE condition to HAVING, we are changing which customers are considered for grouping and filtering.
The first query groups all customers who ordered in October, then filters those with more than 10 orders.
The second query only considers customers who ordered during the specified period before any aggregation.

Impact on Performance:

In the WHERE -> GROUP BY -> HAVING scenario, you filter a smaller set of data first, potentially reducing the number of groups to process.
In the HAVING only scenario, more rows are processed initially before filtering by date and then by count.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Common Mistake: Thinking that moving conditions between WHERE and HAVING doesn't affect performance because SQL optimizers handle such changes. However, the order can significantly impact which data is processed first.

#### Follow-Up
Questions: Interviewers might ask about additional optimization techniques or how to handle different aggregate functions with similar logic.
Self-Check

Correctness + completeness confirmed.

