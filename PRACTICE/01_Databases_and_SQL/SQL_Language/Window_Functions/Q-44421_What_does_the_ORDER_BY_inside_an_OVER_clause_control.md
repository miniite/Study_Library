---
type: "practice"
difficulty: "Basic"
---

# What does the ORDER BY inside an OVER clause control?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Ordering Inside and Outside OVER](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Ordering_Inside_and_Outside_OVER.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand what ORDER BY inside an OVER clause controls, let's break down the core concept and explore its implications.

#### Triaging the Question

The question is probing your understanding of SQL window functions, specifically how the ORDER BY within the OVER clause influences the execution and results. The key idea here is to grasp that the ORDER BY inside an OVER clause determines the order in which rows are processed for each partition of a query.

#### First-Principles Solution
Understanding Window Functions: Window functions operate on a set of rows that is related to, but potentially different from, the result set returned by the query. The OVER clause defines this window.
The Role of ORDER BY: Within the OVER clause, an ORDER BY statement specifies the order in which rows are processed for each partition. This ordering affects how calculations and aggregations are performed within that window.
Detailed Explanation

Basic Query Structure:

SQL
SELECT column1,
       LAG(column2) OVER (PARTITION BY group_column ORDER BY sort_column)
FROM table_name;

Partitions: The PARTITION BY clause divides the result set into partitions. For example, if you partition by a customer ID, each customer's rows are treated as a separate window.

Ordering within Partitions: The ORDER BY inside the OVER clause defines the order in which rows are processed for each partition. This is crucial because it affects how row functions like LAG, LEAD, and others operate. For instance, LAG(column2) OVER (PARTITION BY customer_id ORDER BY transaction_date) processes rows by the transaction_date.

#### Example

#### Cons
ider a sales table with the following data:

customer_id	sale_date	amount
1	2023-01-01	50
1	2023-01-03	70
1	2023-01-05	60
2	2023-01-02	80
2	2023-01-04	90

For the query:

SQL
SELECT customer_id, sale_date,
       LAG(amount) OVER (PARTITION BY customer_id ORDER BY sale_date) AS prev_amount
FROM sales;

The result would be:

customer_id	sale_date	prev_amount
1	2023-01-01	NULL
1	2023-01-03	50
1	2023-01-05	70
2	2023-01-02	NULL
2	2023-01-04	80

Here, the ORDER BY sale_date within the OVER clause ensures that the previous amount is calculated based on the order of sales dates for each customer.

#### Common Mistakes
and Follow-Ups
Misunderstanding the Order: Not recognizing that the ORDER BY inside OVER is crucial for defining the sequence in which rows are processed.
Incorrect Partitioning: Assuming that window functions operate without partitioning, leading to incorrect results when multiple partitions are involved.

#### Closing Summary
and Common Mistakes

In summary, the ORDER BY inside an OVER clause controls the order of row processing within each partition for window functions. Understanding this is crucial for leveraging window functions effectively in SQL queries. Be mindful that without proper ordering or partitioning, results can be incorrect or misleading.

#### Common mistakes
include not specifying a meaningful order, leading to incorrect aggregation or calculation outcomes.

**Comparison:**

```
customer_id	sale_date	amount
1	2023-01-01	50
1	2023-01-03	70
1	2023-01-05	60
2	2023-01-02	80
2	2023-01-04	90
```

**Code:**

```sql
SELECT customer_id, sale_date,
       LAG(amount) OVER (PARTITION BY customer_id ORDER BY sale_date) AS prev_amount
FROM sales;
```

**Example input / output:**

```sql
-- Input
SELECT customer_id, sale_date,
       LAG(amount) OVER (PARTITION BY customer_id ORDER BY sale_date) AS prev_amount
FROM sales;

-- Output
+-------------+--------------+----------------+
| customer_id | sale_date    | prev_amount    |
+-------------+--------------+----------------+
| 1           | 2023-01-01   | NULL           |
| 1           | 2023-01-03   | 50             |
| 1           | 2023-01-05   | 70             |
| 2           | 2023-01-02   | NULL           |
| 2           | 2023-01-04   | 80             |
+-------------+--------------+----------------+
```

**Data:**

| customer_id | sale_date    | amount |
|-------------|--------------|--------|
| 1           | 2023-01-01   | 50     |
| 1           | 2023-01-03   | 70     |
| 1           | 2023-01-05   | 60     |
| 2           | 2023-01-02   | 80     |
| 2           | 2023-01-04   | 90     |

**Flow:**

1. Input
2. Window Function with ORDERBY
3. Partitioning by customer_id,ordering by sale_date
4. LAG(amount) is calculatedbased on this order
5. Output with prev_amountvalues

