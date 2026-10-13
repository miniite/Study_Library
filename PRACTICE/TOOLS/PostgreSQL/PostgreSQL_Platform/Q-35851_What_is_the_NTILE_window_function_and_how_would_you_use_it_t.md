---
type: "practice"
difficulty: "Deep"
---

# What is the NTILE window function and how would you use it to split users into quartiles by spend?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ FIRST_VALUE and LAST_VALUE](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/FIRST_VALUE_and_LAST_VALUE.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To split users into quartiles based on their spending using PostgreSQL, we need to understand the NTILE window function. The NTILE function divides a result set into a specified number of groups (quartiles in this case) and assigns each row to one of these groups. This is particularly useful for partitioning data or creating buckets.

The question is asking us to use NTILE to categorize users based on their spending levels, effectively dividing them into four equal parts: Q1 (lowest spenders), Q2, Q3, and Q4 (highest spenders).

Let’s dive into the first-principles solution:

#### Triage
Question Analysis: We need to understand how NTILE works and apply it in a practical scenario.

#### Assumptions
The table is named users.
There is a column called spending_amount that contains each user's total spending.

#### First Principles Solution

To use the NTILE window function, we first need to understand its syntax and functionality. The basic structure of the query would look like this:

SQL
SELECT
    user_id,
    spending_amount,
    NTILE(4) OVER (ORDER BY spending_amount DESC) as quartile
FROM
    users;

This query will divide the spending_amount values into four groups, ranking them from highest to lowest. The resulting quartile column will assign each user a number from 1 to 4.

Detailed Steps
Identify the Column: We need a column that captures spending amounts.
Order by Spending Amount: Use ORDER BY spending_amount DESC to sort users based on their total spend in descending order (most to least).
Apply NTILE Function: The NTILE(4) function will divide these sorted values into four equal groups.

#### Example
Query

Here is a full example query:

SQL
SELECT
    user_id,
    spending_amount,
    NTILE(4) OVER (ORDER BY spending_amount DESC) as quartile
FROM
    users;

This would result in something like this:

user_id	spending_amount	quartile
123	5000	1
456	4000	1
789	3000	1
...	...	...
999	2000	2
888	1500	2
...	...	...
666	1000	3
777	900	4

#### Closing Summary
and Common Mistakes

#### Common Mistakes

Forgetting to order the rows before applying NTILE.
Misunderstanding how NTILE works (e.g., assuming it splits based on equal intervals rather than ordered quantiles).

#### Follow-Ups

Discussing performance implications, such as ensuring that the query is optimized for large datasets.
Exploring alternative approaches if NTILE does not meet specific requirements.

**Code:**

```sql
SELECT
    user_id,
    spending_amount,
    NTILE(4) OVER (ORDER BY spending_amount DESC) as quartile
FROM
    users;
```

