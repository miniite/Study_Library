---
type: "practice"
difficulty: "Basic"
---

# What SQL techniques would you use in the transform step to aggregate daily transactions by customer?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ ETL vs ELT Design](<../../../../02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/ETL_vs_ELT_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To aggregate daily transactions by customer, we will focus on using SQL's powerful aggregation functions and date handling capabilities. This involves grouping data by customer ID and transaction date, then applying aggregation functions like SUM or COUNT to calculate totals or counts.

#### Triaging the Question

The question asks about SQL techniques for aggregating daily transactions by customers during an ETL/ELT process. The key aspects being probed are:

Understanding of SQL aggregation functions.
Knowledge of date handling in SQL.
Ability to structure queries logically to meet business requirements.

#### Assumptions
include that we have a customer ID and transaction timestamp in our dataset, and we aim to summarize the data on a daily basis per customer.

Step-by-Step Solution
Identify Required Columns: We need columns such as customer_id, transaction_timestamp, and possibly transaction_amount.
Date Handling: Convert the transaction_timestamp into a date format suitable for aggregation.
Group by Customer and Date: Use the GROUP BY clause to aggregate data.
Apply Aggregation Functions: Summarize transaction amounts or count transactions.
SQL Query Example
SQL
SELECT
    customer_id,
    DATE(transaction_timestamp) AS transaction_date,
    COUNT(*) AS transaction_count,
    SUM(transaction_amount) AS total_transaction_amount
FROM
    raw_transactions
GROUP BY
    customer_id,
    DATE(transaction_timestamp)
ORDER BY
    customer_id,
    transaction_date;

#### Closing Summary
and Common Mistakes/Follow-Ups

This query effectively groups daily transactions by each customer and provides both the count of transactions and their total amounts. Key points to remember:

Use DATE() function for proper date aggregation.
Ensure that all necessary columns are included in the GROUP BY clause to avoid errors.

#### Cons
ider adding an ORDER BY clause if you need a specific order for reporting.

#### Common mistakes
include:

Forgetting to use the DATE() function, which can lead to incorrect grouping by timestamps instead of dates.
Not including required columns in the GROUP BY clause, causing aggregation issues.
Overlooking the ORDER BY clause, leading to unordered results that may be harder to interpret.
Triaging Common Follow-Ups
How would you handle situations where transactions have different currency values?
What if there are missing customer IDs or inconsistent timestamps?
Discuss performance optimizations for large datasets.

**Comparison:**

| customer_id | transaction_date  | transaction_count | total_transaction_amount |
|-------------|------------------|------------------|-------------------------|
| 101         | 2023-09-15        | 1                | 50.5                    |
| 101         | 2023-09-16        | 1                | 75.0                    |
| 102         | 2023-09-14        | 1                | 120.0                   |
| 102         | 2023-09-15        | 1                | 80.0                    |

**Code:**

```sql
-- Example SQL Query
SELECT
    customer_id,
    DATE(transaction_timestamp) AS transaction_date,
    COUNT(*) AS transaction_count,
    SUM(transaction_amount) AS total_transaction_amount
FROM
    raw_transactions
GROUP BY
    customer_id,
    DATE(transaction_timestamp)
ORDER BY
    customer_id,
    transaction_date;
```

**Example input / output:**

```text
-- Input Data
INSERT INTO raw_transactions (customer_id, transaction_timestamp, transaction_amount) VALUES
(101, '2023-09-15T14:30:00Z', 50.5),
(101, '2023-09-16T08:00:00Z', 75.0),
(102, '2023-09-14T10:30:00Z', 120.0),
(102, '2023-09-15T16:00:00Z', 80.0);

-- Expected Output
SELECT
    customer_id,
    DATE(transaction_timestamp) AS transaction_date,
    COUNT(*) AS transaction_count,
    SUM(transaction_amount) AS total_transaction_amount
FROM
    raw_transactions
GROUP BY
    customer_id,
    DATE(transaction_timestamp)
ORDER BY
    customer_id,
    transaction_date;

-- Output
+------------+-------------------+--------------------+--------------------------------------+
| customer_id | transaction_date  | transaction_count  | total_transaction_amount             |
+------------+-------------------+--------------------+--------------------------------------+
| 101        | 2023-09-15        | 1                  | 50.5                                 |
| 101        | 2023-09-16        | 1                  | 75.0                                 |
| 102        | 2023-09-14        | 1                  | 120.0                                |
| 102        | 2023-09-15        | 1                  | 80.0                                 |
+------------+-------------------+--------------------+--------------------------------------+
```

**Data:**

```json
{
  "raw_transactions": [
    {"customer_id": 101, "transaction_timestamp": "2023-09-15T14:30:00Z", "transaction_amount": 50.5},
    {"customer_id": 101, "transaction_timestamp": "2023-09-16T08:00:00Z", "transaction_amount": 75.0},
    {"customer_id": 102, "transaction_timestamp": "2023-09-14T10:30:00Z", "transaction_amount": 120.0},
    {"customer_id": 102, "transaction_timestamp": "2023-09-15T16:00:00Z", "transaction_amount": 80.0}
  ]
}
```

**Flow:**

1. Yes
2. Start
3. Is customer_id andtransaction_timestamppresent?
4. Extract date from timestamp
5. Group by customer_id,DATE(transaction_timestamp)
6. Apply aggregation functions(COUNT, SUM)
7. Order by customer_id,transaction_date
8. End

