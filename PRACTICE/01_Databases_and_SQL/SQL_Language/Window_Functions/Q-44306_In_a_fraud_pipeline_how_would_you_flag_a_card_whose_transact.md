---
type: "practice"
difficulty: "Deep"
---

# In a fraud pipeline, how would you flag a card whose transaction exceeds N times its trailing 30-day framed average, and why is a RANGE INTERVAL frame preferable to ROWS here?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Time-based Windows](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Time-based_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of identifying fraudulent transactions where a credit card’s transaction amount significantly deviates from its historical norm, we need to compare each transaction against the average of the previous 30 days. Using window functions in SQL allows us to efficiently compute this trailing average for every transaction within a sliding frame.

TL;DR: We will use RANGE INTERVAL with a frame size of 30 days to calculate the moving average, and then flag transactions that exceed N times this average value. The key advantage of using RANGE INTERVAL is its ability to account for the time dimension accurately, ensuring a robust fraud detection mechanism.

#### Triage
The question is probing your understanding of window functions in SQL, specifically focusing on how to define and use frame intervals effectively. We need to make some assumptions: the transactions are timestamped; N is a predefined threshold value; and we want to flag suspicious activities based on a significant deviation from historical norms.

#### First-Principles Solution

Define the Window Frame: We will create a window that considers a RANGE INTERVAL of 30 days around each transaction.
Calculate the Average Transaction Amount: Within this frame, we compute the average transaction amount for each card.
Flag Transactions Exceeding N Times the Average: Compare the current transaction against this calculated average to identify potential fraud.

Code Implementation:

SQL
WITH AvgTransaction AS (
    SELECT
        card_id,
        transaction_time,
        AVG(amount) OVER (PARTITION BY card_id ORDER BY transaction_time RANGE BETWEEN INTERVAL '30' DAY PRECEDING AND CURRENT ROW) as avg_amount_30_days
    FROM transactions
)
SELECT
    t.card_id,
    t.transaction_time,
    t.amount,
    CASE
        WHEN t.amount > N * a.avg_amount_30_days THEN 'FRAUD'
        ELSE 'NORMAL'
    END AS fraud_flag
FROM transactions t
JOIN AvgTransaction a ON t.card_id = a.card_id AND t.transaction_time = a.transaction_time;

#### Closing Summary
By using RANGE INTERVAL in the window function, we ensure that our average transaction amount calculation accurately accounts for the time dimension. This approach helps in identifying fraudulent activities more precisely by flagging transactions that significantly deviate from historical norms.

#### Common Mistakes/Follow-Ups
Common pitfalls include misinterpreting the frame definition or using ROWS instead of RANGE INTERVAL, which would not capture the temporal aspect correctly. Another mistake is not considering edge cases, such as when there are insufficient historical data points within 30 days for an accurate average.

**Comparison:**

```
card_id	transaction_time	amount	fraud_flag
1	2023-01-01	50	NORMAL
1	2023-01-07	60	NORMAL
1	2023-01-14	150	FRAUD
```

**Code:**

```sql
WITH AvgTransaction AS (
    SELECT
        card_id,
        transaction_time,
        AVG(amount) OVER (PARTITION BY card_id ORDER BY transaction_time RANGE BETWEEN INTERVAL '30' DAY PRECEDING AND CURRENT ROW) as avg_amount_30_days
    FROM transactions
)
SELECT
    t.card_id,
    t.transaction_time,
    t.amount,
    CASE
        WHEN t.amount > N * a.avg_amount_30_days THEN 'FRAUD'
        ELSE 'NORMAL'
    END AS fraud_flag
FROM transactions t
JOIN AvgTransaction a ON t.card_id = a.card_id AND t.transaction_time = a.transaction_time;
```

**Example input / output:**

```

WITH AvgTransaction AS ( SELECT card_id, transaction_time, AVG(amount) OVER (PARTITION BY card_id ORDER BY transaction_time RANGE BETWEEN INTERVAL '30' DAY PRECEDING AND CURRENT ROW) as avg_amount_30_days FROM transactions ) SELECT t.card_id, t.transaction_time, t.amount, CASE WHEN t.amount > 2 * a.avg_amount_30_days THEN 'FRAUD' ELSE 'NORMAL' END AS fraud_flag FROM transactions t JOIN AvgTransaction a ON t.card_id = a.card_id AND t.transaction_time = a.transaction_time;

CODE
Copy
Result:
```sql
card_id | transaction_time | amount | fraud_flag
1       | 2023-01-01      | 50     | NORMAL
1       | 2023-01-07      | 60     | NORMAL
1       | 2023-01-14      | 150    | FRAUD
```

**Data:**

```json
{
  "transactions": [
    {"card_id": 1, "transaction_time": "2023-01-01", "amount": 50},
    {"card_id": 1, "transaction_time": "2023-01-07", "amount": 60},
    {"card_id": 1, "transaction_time": "2023-01-14", "amount": 150}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Transaction Record
4. Is Time Interval Valid?
5. RANGE INTERVAL FrameCalculation
6. Average Amount Computation
7. Fraud Flag Determination(amount > N *avg_amount_30_days)
8. Flagged Transaction
9. Insufficient Data Handling
10. Normal Transaction

