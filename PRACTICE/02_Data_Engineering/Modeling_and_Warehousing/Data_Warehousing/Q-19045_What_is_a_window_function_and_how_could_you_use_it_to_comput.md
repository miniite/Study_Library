---
type: "practice"
difficulty: "Intermediate"
---

# What is a window function and how could you use it to compute a running total in warehouse reporting?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Fact Dimension Integrity](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Fact_Dimension_Integrity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Window functions are powerful tools in SQL that allow us to perform calculations across a set of rows related to the current row. In the context of data warehousing, they are particularly useful for creating derived metrics such as running totals, which provide insights into cumulative values over time.

To compute a running total using window functions, we need to understand how these functions work and then apply them in a specific scenario. Let's break down the steps involved from first principles.

#### Triaging the Question

The interviewer is probing your understanding of SQL window functions and their practical application for generating derived metrics like running totals. This involves:

Understanding Window Functions: How they partition data, define an ordered set (or "window") of rows, and then perform calculations on this set.
Identifying the Problem: Computing a running total requires summing up values over a sequence of rows, which can be done using the SUM window function with appropriate frame specifications.

#### Assumptions
We have a fact table in our data warehouse that records transactions or events over time.
The table has columns such as transaction_id, timestamp, and amount.
Step-by-Step Solution
1. Define the Problem Domain

Let's consider a simple example where we want to compute the running total of transaction amounts per user, ordered by timestamp.

SQL
SELECT
    user_id,
    timestamp,
    amount,
    SUM(amount) OVER (
        PARTITION BY user_id
        ORDER BY timestamp
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM transactions;
2. Breakdown of the Query
PARTITION BY user_id: This clause divides the data into partitions based on user_id. Each partition contains all rows with the same user_id.
ORDER BY timestamp: The rows within each partition are ordered by timestamp in ascending order.
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW: This frame specification defines the window for the running total. It includes all rows from the start of the partition up to and including the current row.
3. Example Scenario

Suppose we have the following data:

user_id	timestamp	amount
u1	2023-01-01T10:00	100
u1	2023-01-02T11:00	200
u1	2023-01-03T12:00	150

The query would produce:

user_id	timestamp	amount	running_total
u1	2023-01-01T10:00	100	100
u1	2023-01-02T11:00	200	300
u1	2023-01-03T12:00	150	450
4. Common Mistakes and Follow-Ups
Incorrect Partitioning: Failing to partition the data correctly can lead to incorrect running totals.
Ordering Issues: If not ordering the rows properly, the running total might be computed incorrectly or may not make sense.
Frame Specification Errors: Misusing frame specifications (e.g., using ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING instead of ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) can result in incorrect results.

#### Closing Summary

Window functions are essential for generating derived metrics like running totals, which provide valuable insights into cumulative values over time. By correctly defining partitions and ordering the rows, we can compute accurate running totals that help in warehouse reporting. Always double-check your partitioning and frame specifications to ensure correct behavior.

**Code:**

```sql
SELECT
    user_id,
    timestamp,
    amount,
    SUM(amount) OVER (
        PARTITION BY user_id
        ORDER BY timestamp
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM transactions;
```

**Data:**

```json
[
  { "user_id": "u1", "timestamp": "2023-01-01T10:00", "amount": 100 },
  { "user_id": "u1", "timestamp": "2023-01-02T11:00", "amount": 200 },
  { "user_id": "u1", "timestamp": "2023-01-03T12:00", "amount": 150 }
]
```

