---
type: "practice"
difficulty: "Basic"
---

# How would you pull a training dataset of the last 90 days of user transactions from a SQL table?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Retention Queries](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Retention_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To extract a training dataset of the last 90 days of user transactions from an SQL table, we need to filter out records based on their transaction date being within the last 90 days. This involves using SQL's WHERE clause with date functions and understanding how to handle date ranges in your database.

#### Triage

The question is asking for a SQL query that retrieves data from a specific time range (the last 90 days) from a user transactions table. The interviewer is probing whether you can effectively use SQL date functions, understand date arithmetic, and write efficient queries. Key assumptions include the existence of a transaction_date column in the table.

#### First-Principles Solution
Identify the Date Range: We need to determine the start and end dates for the last 90 days.
Filter Transactions: Use SQL's date functions to filter transactions that fall within this range.
Select Relevant Data: Ensure you select all necessary columns from the table.
Step-by-Step Solution

Identify Today’s Date:

Assume today's date is 2023-10-15.

Calculate the Start Date of the Last 90 Days:

Subtract 90 days from today’s date: 2023-10-15 - 90 = 2023-07-16.

Formulate the SQL Query:

SQL
SELECT *
FROM user_transactions
WHERE transaction_date >= '2023-07-16'
  AND transaction_date < '2023-10-15';

#### Closing Summary
and Common Mistakes/Follow-Ups

The query above is a straightforward way to retrieve the required data. However, there are a few common pitfalls:

Incorrect Date Handling: Ensure you use the correct date format for your database (e.g., 'YYYY-MM-DD').
Inclusive vs Exclusive Boundaries: The upper boundary uses AND transaction_date < '2023-10-15' to exclude today's transactions, as we are looking at data up to but not including the current date.

#### Performance Considerations
If this table is very large, ensure that the column used in filtering (transaction_date) has an index to optimize query performance.

In a follow-up, you might be asked about handling time zones or dealing with transactions spanning midnight across different days. For now, our simple solution covers most scenarios effectively.

**Code:**

```sql
SELECT *
FROM user_transactions
WHERE transaction_date >= '2023-07-16'
  AND transaction_date < '2023-10-15';
```

**Example input / output:**

```

None provided directly, but here’s a conceptual example:

SQL
Copy
-- Example Input
user_transactions_table

| user_id | product_id | quantity | transaction_date |
|---------|------------|----------|------------------|
| 1       | A          | 5        | 2023-07-16       |
| 2       | B          | 4        | 2023-08-25       |
| 3       | C          | 6        | 2023-09-15       |
| 4       | D          | 3        | 2023-10-10       |
| 5       | E          | 7        | 2023-10-15       |

-- Example Output
result_table

| user_id | product_id | quantity | transaction_date |
|---------|------------|----------|------------------|
| 1       | A          | 5        | 2023-07-16       |
| 2       | B          | 4        | 2023-08-25       |
| 3       | C          | 6        | 2023-09-15       |
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Today's Date Known?
5. Calculate Start and End Dates
6. Formulate SQL Query
7. Run the Query
8. Retrieve Data
9. End
10. Input Today's Date Manually

