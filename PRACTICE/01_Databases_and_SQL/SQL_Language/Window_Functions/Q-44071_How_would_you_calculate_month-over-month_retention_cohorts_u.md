---
type: "practice"
difficulty: "Deep"
---

# How would you calculate month-over-month retention cohorts using LAG across a pivoted cohort table, and where do window functions fit versus a self-join?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Time-based Windows](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Time-based_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To calculate month-over-month (MoM) retention cohorts using LAG in SQL, we start by understanding that this involves comparing user activity between consecutive months to identify how many users remain active from one month to another. The core idea is leveraging window functions like LAG to access data from previous rows within the same partition. This approach avoids complex self-joins and simplifies the query logic.

First, we need to structure our table appropriately. For this example, let's assume a simplified schema:

SQL
CREATE TABLE user_activity (
    id INT,
    user_id INT,
    month DATE
);

**Comparison:**

```
user_id	month
1	2023-04-01
2	2023-05-01
3	2023-06-01
4	2023-05-01
```

**Flow:**

1. No
2. Start
3. Is the table pivoted?
4. Create a pivot table withuser_id, month
5. Lag the active users from onemonth to another
6. Calculate MoM retention ratio
7. End

