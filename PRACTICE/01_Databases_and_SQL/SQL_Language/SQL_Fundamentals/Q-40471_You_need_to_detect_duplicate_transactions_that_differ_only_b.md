---
type: "practice"
difficulty: "Deep"
---

# You need to detect duplicate transactions that differ only by a retry-generated id but share (user, amount, merchant, minute). How would you write the dedup query and what unique constraint would prevent them at write time?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Unique Constraints for Dedup](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Unique_Constraints_for_Dedup.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of detecting duplicate transactions that differ only by a retry-generated ID but share common attributes like user, amount, merchant, and minute, we need to craft an SQL query that identifies such duplicates efficiently. Additionally, implementing unique constraints at write time can prevent these duplicates from being created in the first place.

At its core, this problem involves detecting identical transactions with different IDs. The solution will leverage SQL's powerful set operations and pattern matching capabilities. Let’s break it down step by step.

First, let's restate what the question is asking:

Identify Duplicate Transactions: Find transactions that share user, amount, merchant, and minute but have different retry-generated IDs.
Prevent Duplicates at Write Time: Implement a unique constraint to ensure such duplicates are not inserted into the database.
Step-by-Step Solution
1. Identify Duplicate Transactions

To detect these duplicate transactions, we can use a combination of GROUP BY and aggregate functions like COUNT() or DISTINCT. Here’s how you might write the query:

SQL
SELECT user_id, merchant_id, amount, minute,
       STRING_AGG(transaction_id, ', ') AS transaction_ids
FROM (
    SELECT user_id, merchant_id, amount, minute, transaction_id
    FROM transactions
    GROUP BY user_id, merchant_id, amount, minute
    HAVING COUNT(DISTINCT transaction_id) > 1
) AS duplicates
GROUP BY user_id, merchant_id, amount, minute;

This query groups transactions by user, merchant, amount, and minute and filters out those that have more than one unique transaction_id. The nested subquery ensures we only consider groups with multiple transaction IDs.

2. Prevent Duplicates at Write Time

To prevent these duplicates from being inserted, you can add a unique constraint on the combination of columns: user, amount, merchant, and minute. This is done using a partial index in SQL.

Here’s how to define such an index:

SQL
CREATE UNIQUE INDEX idx_user_amount_merchant_minute ON transactions (user_id, amount, merchant_id, minute);

This index ensures that no two rows with the same combination of these columns can exist in the transactions table. If someone tries to insert a row that violates this constraint, the database will throw an error.

#### Closing Summary

In summary, we've identified duplicate transactions by grouping and aggregating data, and prevented them at write time using a unique index on specific columns. This approach ensures data integrity while minimizing redundant storage and processing costs.

#### Common Mistakes/Follow-Ups
Mistake 1: Not considering the performance implications of large GROUP BY clauses.
Mistake 2: Failing to define proper indexes, leading to potential bottlenecks during write operations.

#### Follow-Up
Discussing how this solution scales with a high number of concurrent transactions and what additional optimizations might be necessary.

**Comparison:**

```
00
00
Query Definition
Index Creation
Deduplication and Unique Constraint Implementation
```

**Code:**

```sql
-- Query for identifying duplicates
SELECT user_id, merchant_id, amount, minute,
       STRING_AGG(transaction_id, ', ') AS transaction_ids
FROM (
    SELECT user_id, merchant_id, amount, minute, transaction_id
    FROM transactions
    GROUP BY user_id, merchant_id, amount, minute
    HAVING COUNT(DISTINCT transaction_id) > 1
) AS duplicates
GROUP BY user_id, merchant_id, amount, minute;

-- Unique constraint definition
CREATE UNIQUE INDEX idx_user_amount_merchant_minute ON transactions (user_id, amount, merchant_id, minute);
```

**Example input / output:**

```text
Input:
INSERT INTO transactions (user_id, merchant_id, amount, minute, transaction_id) VALUES (1, 201, 5.99, 1638475200, 4);

Output:
ERROR: duplicate key value violates unique constraint "idx_user_amount_merchant_minute"
DETAIL: Key (user_id, merchant_id, amount, minute)=(1, 201, 5.99, 1638475200) already exists.
```

**Data:**

```json
{
    "transactions": [
        {"user_id": 1, "merchant_id": 201, "amount": 5.99, "minute": 1638475200, "transaction_id": 1},
        {"user_id": 1, "merchant_id": 201, "amount": 5.99, "minute": 1638475200, "transaction_id": 2},
        {"user_id": 2, "merchant_id": 202, "amount": 10.99, "minute": 1638475205, "transaction_id": 3}
    ]
}
```

**Flow:**

1. Transaction Insertion
2. Check Unique Constraint
3. Insert Success

