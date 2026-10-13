---
type: "practice"
difficulty: "Intermediate"
---

# Write a query to find users who hit their token quota by summing tokens per user and comparing against a quota column.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Aggregation Query Practice](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Aggregation_Query_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to write an SQL query that sums up the tokens consumed by each user and compares it with a predefined quota. This requires us to first group the data by user IDs, sum the token values, and then check if any of these sums exceed the quota.

#### Triaging the Question

The core of this question is to filter users based on their total token usage exceeding a specific limit. We need to use SQL's GROUP BY clause for aggregation and HAVING clause for filtering groups.

#### Assumptions
The table has columns like user_id, token_usage.
There is a column named quota representing the allowed token usage per user.

#### First Principles Solution

Step 1: Group Data by User ID:

We need to group all records by user_id to aggregate their token usage.

Step 2: Sum Token Usage Per User:

For each grouped user, sum up the token_usage.

Step 3: Filter Users Who Exceed Quota:

Use a HAVING clause to filter out users whose total token_usage is greater than their quota.
Full Solution

Here's how we can write this SQL query:

SQL
SELECT user_id, SUM(token_usage) as total_tokens_used
FROM token_usage_table
GROUP BY user_id
HAVING SUM(token_usage) > quota;

This query will return a list of users who have exceeded their token usage limit.

#### Closing Summary
and Common Mistakes/Follow-Ups
Common Mistake: Forgetting to use the SUM() function for aggregation.

#### Follow-Up
Discuss how you might optimize this query if performance is a concern (e.g., using indexes, partitioning).

If we have additional data or need more details, we can further refine our approach.

**Code:**

```sql
SELECT user_id, SUM(token_usage) as total_tokens_used
FROM token_usage_table
GROUP BY user_id
HAVING SUM(token_usage) > quota;
```

