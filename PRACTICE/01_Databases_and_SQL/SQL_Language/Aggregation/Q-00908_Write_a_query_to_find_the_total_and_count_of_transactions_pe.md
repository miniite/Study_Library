---
type: "practice"
difficulty: "Intermediate"
---

# Write a query to find the total and count of transactions per payment method in a single query.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Counting with Joins](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/Counting_with_Joins.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to write a SQL query that aggregates transaction data by payment method, calculating both the total amount for each method and the count of transactions associated with it. This involves using aggregate functions SUM() and COUNT(). Here’s how we can approach this step-by-step.

#### Triage

The question is asking us to retrieve two metrics: the total transaction amounts and the number of transactions for each payment method in a single query. The interviewer might be probing our understanding of SQL syntax, especially aggregate functions, as well as our ability to use grouping clauses effectively.

#### First-Principles Solution
Identify the Tables and Columns: Assume we have a table named transactions with columns like transaction_id, payment_method, and amount.
Aggregate Functions: We will use SUM() to aggregate the transaction amounts for each payment method, and COUNT() to count the number of transactions.
Group By Clause: To group by payment methods, we need a GROUP BY clause.
SQL Query
SQL
SELECT
    payment_method,
    COUNT(*) AS transaction_count,
    SUM(amount) AS total_amount
FROM
    transactions
GROUP BY
    payment_method;

#### Closing Summary
and Common Mistakes/Follow-Ups

This query provides the required information in a single pass. The COUNT(*) function counts the number of rows, which corresponds to the number of transactions for each payment method. The SUM(amount) calculates the total transaction amount for each group.

#### Common Mistakes
Forgetting to use the GROUP BY clause, which would result in an error.
Misusing the aggregate functions; for example, using COUNT() instead of SUM(amount) or vice versa.

#### Example
Data

Assume our transactions table has the following data:

transaction_id	payment_method	amount
1	credit_card	50.00
2	debit_card	75.00
3	cash	40.00
4	credit_card	60.00

#### Example
Output

The query would produce the following output:

PLAINTEXT
payment_method  | transaction_count | total_amount
----------------|------------------|--------------
credit_card     | 2                | 110.00
debit_card      | 1                | 75.00
cash            | 1                | 40.00
Self-Check

This query is both correct and complete, as it uses the appropriate aggregate functions and a GROUP BY clause to produce the desired output.

