---
type: "practice"
difficulty: "Intermediate"
---

# How would you find the gap in days between consecutive login dates for each user?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Time-based Windows](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Time-based_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to identify the gaps in days between consecutive login dates for each user. The key concept here is using window functions, which allow us to perform calculations across a set of rows that are related to the current row.

The core idea is to use LAG and DATEDIFF (or similar date difference function) within a window defined by user IDs and ordered by login dates. This will help us calculate the difference in days between each user's consecutive logins.

#### Triage

The question requires finding gaps in days between consecutive login dates for users. The interviewer is probing your understanding of SQL, specifically window functions and date arithmetic. We'll assume that the data contains a users table with columns such as user_id and login_date.

#### First-Principles Solution
Identify the Data Structure: Assume we have a table named logins with columns user_id, login_date.
Use Window Functions:
Use LAG to access the previous row's login date for each user.
Compute the difference in days between consecutive logins using DATEDIFF.

Here’s how you can implement this:

SQL
SELECT
    user_id,
    login_date,
    DATEDIFF(day, LAG(login_date) OVER (PARTITION BY user_id ORDER BY login_date), login_date) AS gap_in_days
FROM
    logins
ORDER BY
    user_id, login_date;

#### Closing Summary
and Common Mistakes/Follow-Ups

The query above will return the gap in days between consecutive login dates for each user. It’s important to ensure that login_date is correctly formatted as a date or timestamp.

#### Common Mistakes

Forgetting to partition by user_id, which would result in incorrect differences across different users.
Using an incorrect window function (e.g., LAG instead of LEAD for backward-looking gaps).
Not ordering the data by login_date within each partition, leading to incorrect sequences.

#### Follow-Ups

How do you handle cases where a user has no consecutive logins?
What if there are many users and logins, would performance be an issue? Can we optimize it further?

**Comparison:**

| user_id | login_date |
|---------|------------|
| 1       | 2023-01-01 |
| 1       | 2023-01-05 |
| 1       | 2023-01-10 |
| 2       | 2023-02-01 |

**Code:**

```sql
SELECT
    user_id,
    login_date,
    DATEDIFF(day, LAG(login_date) OVER (PARTITION BY user_id ORDER BY login_date), login_date) AS gap_in_days
FROM
    logins
ORDER BY
    user_id, login_date;
```

**Example input / output:**

```text
user_id | login_date   | gap_in_days
--------|--------------|-------------
1       | 2023-01-05   | 4
1       | 2023-01-10   | 5
2       | 2023-02-01   | NULL (no consecutive logs for user 2)
```

**Data:**

```json
{
  "logins": [
    {"user_id": 1, "login_date": "2023-01-01"},
    {"user_id": 1, "login_date": "2023-01-05"},
    {"user_id": 1, "login_date": "2023-01-10"},
    {"user_id": 2, "login_date": "2023-02-01"}
  ]
}
```

**Flow:**

1. Logins Table
2. User ID?
3. Partition By User ID
4. Order by Login Date
5. Use LAG Function
6. Calculate Gap in Days
7. Return Results

