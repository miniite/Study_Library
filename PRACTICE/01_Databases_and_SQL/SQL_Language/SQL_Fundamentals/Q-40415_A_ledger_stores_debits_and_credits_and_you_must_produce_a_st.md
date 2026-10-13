---
type: "practice"
difficulty: "Deep"
---

# A ledger stores debits and credits, and you must produce a statement with a running balance that resets each account. How do you partition and order the window, and why must you break ties deterministically to keep balances stable across reruns?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Large Batch Updates](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Large_Batch_Updates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To generate a ledger statement with a running balance that resets each account, we need to ensure the SQL query is designed to correctly maintain stability in the balance calculations even when there are ties or multiple transactions at the same time. The key steps involve partitioning by account and ordering the transactions appropriately while handling potential tie-breaking conditions deterministically.

#### Triaging the Question

The core of this question revolves around ensuring a stable and deterministic calculation of running balances across different runs of the query. We need to partition the data by each account and then order the transactions within those partitions. Since ties can occur (e.g., multiple transactions at the same timestamp), we must break these ties deterministically to avoid any ambiguity in the balance calculations.

#### First-Principles Solution
Partition by Account: Each transaction belongs to a specific account, so partitioning by account ensures that all transactions for an account are processed together.
Order Transactions: Within each partition (account), order transactions based on their timestamp or any other relevant field. If timestamps are identical, use another deterministic field like transaction ID or insertion order.
Running Balance Calculation: Use a window function to compute the running balance starting from zero at the beginning of each account.
Detailed Solution
Step 1: Define Partitions and Order

We partition by account_id and order by transaction_date, then within the same date, order by transaction_id.

SQL
SELECT
    account_id,
    transaction_date,
    transaction_id,
    amount,
    SUM(amount) OVER (
        PARTITION BY account_id
        ORDER BY transaction_date ASC, transaction_id ASC
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_balance
FROM transactions;
Step 2: Ensure Deterministic Order for Ties

To handle ties in transaction_date, we use transaction_id to break them deterministically. If both fields are the same (rare but possible), ensure a unique identifier is used, such as an auto-incrementing primary key.

#### Closing Summary
and Common Mistakes/Follow-Ups

By partitioning by account and ordering transactions within each partition using a deterministic approach, we can maintain stable and consistent running balances. If there are ties in the order, they need to be broken deterministically to ensure that results remain stable across multiple executions of the query.

#### Common Mistakes
Ignoring Ties: Not handling ties correctly can lead to non-deterministic results.
Lack of Determinism: Failing to specify a deterministic method for breaking ties can cause the same input data to produce different outputs in repeated runs.
Trialing Follow-Up Questions
How would you optimize this query for large datasets?
What if there are multiple accounts with transactions at exactly the same time? How would you handle that?

**Comparison:**

| account_id | transaction_date | transaction_id | amount | running_balance |
|------------|------------------|----------------|--------|-----------------|
| 100        | 2023-10-01       | 1              | 50     | 50              |
| 100        | 2023-10-01       | 2              | -20    | 30              |
| 101        | 2023-10-01       | 3              | 30     | 30              |

**Code:**

```sql
SELECT
    account_id,
    transaction_date,
    transaction_id,
    amount,
    SUM(amount) OVER (
        PARTITION BY account_id
        ORDER BY transaction_date ASC, transaction_id ASC
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_balance
FROM transactions;
```

**Example input / output:**

```text
Input:
transactions = [
    {account_id: 100, transaction_date: '2023-10-01', transaction_id: 1, amount: 50},
    {account_id: 100, transaction_date: '2023-10-01', transaction_id: 2, amount: -20},
    {account_id: 101, transaction_date: '2023-10-01', transaction_id: 3, amount: 30}
]

Output:
[
    {"account_id": 100, "transaction_date": "2023-10-01", "transaction_id": 1, "amount": 50, "running_balance": 50},
    {"account_id": 100, "transaction_date": "2023-10-01", "transaction_id": 2, "amount": -20, "running_balance": 30},
    {"account_id": 101, "transaction_date": "2023-10-01", "transaction_id": 3, "amount": 30, "running_balance": 30}
]
```

**Data:**

```json
{
  "transactions": [
    {"account_id": 100, "transaction_date": "2023-10-01", "transaction_id": 1, "amount": 50},
    {"account_id": 100, "transaction_date": "2023-10-01", "transaction_id": 2, "amount": -20},
    {"account_id": 101, "transaction_date": "2023-10-01", "transaction_id": 3, "amount": 30}
  ]
}
```

**Flow:**

1. SQL Query
2. SELECT account_id,transaction_date,transaction_id, amount, ...
3. SUM(amount) OVER(PARTITION BY account_idORDER BY transaction_dateASC, transaction_id ASC ROWSBETWEEN UNBOUNDEDPRECEDING AND CURRENTROW)
4. Start
5. Partition by Account
6. Order by Date and TransactionID
7. Calculate Running Balance
8. End

