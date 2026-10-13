---
type: "practice"
difficulty: "Deep"
---

# In Spark SQL a window without PARTITION BY forces all data into a single partition; how do you diagnose the resulting single-task skew from the Spark UI and what redesign avoids it?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Window Function Fundamentals and Performance](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Window_Function_Fundamentals_and_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose a single-partition skew in Spark SQL when using window functions without a PARTITION BY, we need to understand that all rows are placed into a single partition, causing the job to be executed by a single task. This can lead to performance bottlenecks and inefficient use of resources. The key is to identify this issue through Spark UI metrics and then redesign the query to properly partition the data.

#### Triage

The question asks us to diagnose a single-task skew in Spark SQL when using window functions without PARTITION BY and how to avoid it. This involves understanding the implications of such an approach, identifying the specific symptoms on the Spark UI, and restructuring the query for better performance.

Diagnosing Single-Task Skew
Spark UI Metrics: In the Spark Web UI, navigate to the "Stages" tab. Look for stages with only one active task. A single-task stage is a clear indicator of skew.
Stage Details: Click on the problematic stage to see more details. Pay attention to metrics like Shuffle Write Size and Total shuffle read bytes. These often indicate data imbalance.
Redesigning the Query

To avoid this single-partition skew, we need to properly partition our data using a PARTITION BY clause in the window function or by splitting the data into multiple partitions at the planning stage. Here’s a step-by-step approach:

Identify Suitable Partition Keys: Choose appropriate columns that can naturally split your dataset into smaller, more balanced subsets.
Implement Window Function with PARTITION BY: Modify the query to use PARTITION BY on these keys.

#### Example
Query

Suppose we have a dataset of transactions and want to calculate rolling averages without using any partitioning:

SQL
SELECT
    transaction_id,
    user_id,
    timestamp,
    AVG(amount) OVER (ORDER BY timestamp ROWS BETWEEN 10 PRECEDING AND CURRENT ROW) as rolling_avg
FROM transactions

To avoid the single-task skew, we can modify it to include PARTITION BY:

SQL
SELECT
    transaction_id,
    user_id,
    timestamp,
    AVG(amount) OVER (PARTITION BY user_id ORDER BY timestamp ROWS BETWEEN 10 PRECEDING AND CURRENT ROW) as rolling_avg
FROM transactions

This ensures that the window function is applied separately to each user_id, distributing the workload across multiple tasks.

#### Closing Summary
and Common Mistakes
Diagnosing: Use Spark UI to identify single-task stages.
Redesigning: Properly use PARTITION BY in your queries.

#### Common Mistakes
Overlooking the importance of partitioning, failing to monitor Spark UI metrics, not understanding the implications of data skew.

**Comparison:**

```
tablechart
columns: [Step, Action]
rows:
    Step | Action
    1 | Identify single-task stages in Spark UI
    2 | Check metrics like Shuffle Write Size
    3 | Choose appropriate partition keys
    4 | Apply PARTITION BY in window functions
```

**Code:**

```sql
-- Original query with no PARTITION BY
SELECT
    transaction_id,
    user_id,
    timestamp,
    AVG(amount) OVER (ORDER BY timestamp ROWS BETWEEN 10 PRECEDING AND CURRENT ROW) as rolling_avg
FROM transactions

-- Redesigned query with PARTITION BY
SELECT
    transaction_id,
    user_id,
    timestamp,
    AVG(amount) OVER (PARTITION BY user_id ORDER BY timestamp ROWS BETWEEN 10 PRECEDING AND CURRENT ROW) as rolling_avg
FROM transactions
```

**Example input / output:**

```

Input:

SQL
Copy
SELECT
    transaction_id,
    user_id,
    timestamp,
    AVG(amount) OVER (ORDER BY timestamp ROWS BETWEEN 10 PRECEDING AND CURRENT ROW) as rolling_avg
FROM transactions

Output:

Stage with only one task, high Shuffle Write Size, and Total shuffle read bytes.

Input:

SQL
Copy
SELECT
    transaction_id,
    user_id,
    timestamp,
    AVG(amount) OVER (PARTITION BY user_id ORDER BY timestamp ROWS BETWEEN 10 PRECEDING AND CURRENT ROW) as rolling_avg
FROM transactions

Output:

Multiple tasks, balanced Shuffle Write Size, and lower Total shuffle read bytes.
```

**Data:**

```json
{
  "transactions": [
    {"transaction_id": 1, "user_id": 1001, "timestamp": "2023-10-01", "amount": 50},
    {"transaction_id": 2, "user_id": 1001, "timestamp": "2023-10-02", "amount": 60},
    ...
    {"transaction_id": 1000, "user_id": 1002, "timestamp": "2023-10-31", "amount": 80}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Identify skew
4. Use Spark UI?
5. Check metrics, diagnose skew
6. No action needed
7. Redesign query withPARTITION BY
8. Monitor performanceimprovements

