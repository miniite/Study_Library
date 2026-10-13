---
type: "practice"
difficulty: "Deep"
---

# Your lake stores slowly changing dimensions (like a user's plan tier). How do you pick the right feature value for a training example from a given date?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tiered Lake Storage](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of picking the right feature value for a training example from a given date, we need to consider how slowly changing dimensions are handled in data architecture. Specifically, in a lake environment like Delta Lake or Iceberg Table, data is stored with time-evolving attributes. The key is to determine which version of the dimension (user's plan tier) should be used for feature engineering based on the training example’s timestamp.

Firstly, let's clarify what we are probing:

We need to understand how to extract historical data from a lake storage system.
We must consider different strategies for handling time-series data in machine learning.
The approach should balance between accuracy and computational efficiency.

Now, let's reason through the problem step by step:

Data Model: In our lake, we store user plan tiers as slowly changing dimensions (SCDs). These might be represented with a timestamp indicating when the change occurred.

Querying Historical Data: To get the most recent plan tier for a given date, we can use SQL or a query language that supports window functions and joins. For example, in SQL, you could use LAG or LEAD to find the latest value before a certain date.

#### Approach
es:

Snapshot Join: We join the user data with the latest plan tier snapshot as of the training example's timestamp.
Temporal Join: This involves using window functions to dynamically fetch the most recent plan tier within a time range around the training example’s timestamp.

#### Implementation
Example (SQL):

SQL
SELECT
    user_id,
    plan_tier,
    -- other features...
FROM (
    SELECT
        u.user_id,
        p.plan_tier,
        LAG(p.end_date) OVER (PARTITION BY u.user_id ORDER BY p.start_date DESC) AS prev_end_date,
        ROW_NUMBER() OVER (PARTITION BY u.user_id ORDER BY p.start_date DESC) as rn
    FROM users u
    JOIN plans p ON u.user_id = p.user_id AND p.start_date <= '2023-10-05' AND (p.end_date IS NULL OR p.end_date > '2023-10-05')
) subquery
WHERE rn = 1;

#### Cons
iderations:

Performance: Ensure the query is optimized, especially with large datasets.
Data Consistency: Handle edge cases like users who might not have any historical plan tier data.
Schema Evolution: Plan for potential changes in how SCDs are stored over time.

In conclusion, picking the right feature value involves querying the lake storage to retrieve the most recent plan tier as of a given date. This requires a robust understanding of temporal joins and window functions. The approach balances between accuracy (using the latest relevant data) and efficiency (optimizing query performance).

**Comparison:**

| Training Date: 2023-10-05

**Code:**

```sql
-- Example SQL query
SELECT
    user_id,
    plan_tier,
FROM (
    SELECT
        u.user_id,
        p.plan_tier,
        LAG(p.end_date) OVER (PARTITION BY u.user_id ORDER BY p.start_date DESC) AS prev_end_date,
        ROW_NUMBER() OVER (PARTITION BY u.user_id ORDER BY p.start_date DESC) as rn
    FROM users u
    JOIN plans p ON u.user_id = p.user_id AND p.start_date <= '2023-10-05' AND (p.end_date IS NULL OR p.end_date > '2023-10-05')
) subquery
WHERE rn = 1;
```

**Example input / output:**

```text
Input:
- Training date: '2023-10-05'
Output:
- user_id: 1, plan_tier: 'Silver' (latest tier before training date)
- user_id: 2, plan_tier: 'Bronze' (last valid tier on or before training date)
```

**Data:**

```json
{
    "users": [
        {"user_id": 1, "start_date": "2022-01-01", "end_date": null},
        {"user_id": 2, "start_date": "2023-01-01", "end_date": "2023-06-30"}
    ],
    "plans": [
        {"plan_tier": "Bronze", "start_date": "2022-01-01", "end_date": null, "user_id": 1},
        {"plan_tier": "Silver", "start_date": "2023-07-01", "end_date": null, "user_id": 1},
        {"plan_tier": "Gold", "start_date": "2023-06-01", "end_date": "2023-06-30", "user_id": 2}
    ]
}
```

**Flow:**

1. Yes
2. No
3. User Data
4. Join Users and Plans
5. Find Latest Plan Tier
6. (Optional) Handle Edge Cases
7. Is Training Date within PlanTier?
8. Use Current Plan Tier
9. Use Previous Plan Tier

