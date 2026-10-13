---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd exclude entities that were still in an observation window (right-censored) from a churn training set in SQL so you don't mislabel not-yet-churned users as retained.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Analytical SQL for ML Features](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Analytical_SQL_for_ML_Features.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to ensure that entities which are still within their observation window (right-censored) are excluded from our churn training set in SQL. This is crucial because including such entities would result in mislabeling them as retained users, leading to biased and inaccurate models.

In essence, the question is asking us to understand how to handle right-censored data by excluding records where the churn event has not yet occurred within the observation window. The core idea involves identifying those users whose churn event hasn’t happened yet and filtering them out before training our model.

Let’s break down the solution step-by-step:

Identify Right-Censored Users: These are users who have been observed but haven't yet churned by the end of their observation period.
Define the Observation Window: The observation window is typically defined as the time frame during which we can observe user behavior before making a prediction about churn.
Filter Out Right-Censored Users: We need to exclude these users from our training dataset so that they are not mislabeled.

#### Triaging the Question

The interviewer is probing your understanding of SQL and how you handle right-censored data in a time-series context. The main assumptions include:

You have access to user behavior data, including timestamps of churn events.
Each user has a defined observation period during which their churn event can be observed.
Step-by-Step Solution

Identify Churn Events and Observation Windows:

We need to understand the structure of our dataset. Typically, it might look like this:

PLAINTEXT
User_ID | Churn_Date | Observation_Window_End
001     | NULL       | 2023-10-15
002     | 2023-09-10 | 2023-10-30
003     | 2023-10-10 | NULL

Here, Churn_Date is the date when the user churned and Observation_Window_End is the end of our observation period.

Identify Right-Censored Users:

A user is right-censored if their Churn_Date is NULL, meaning they haven’t churned by the end of their observation window.
We can identify these users with a simple SQL query:
SQL
SELECT User_ID, Churn_Date, Observation_Window_End
FROM churn_data
WHERE Churn_Date IS NULL;

Filter Out Right-Censored Users:

To exclude right-censored users from our training set, we need to filter out rows where the Churn_Date is NULL. This can be done in a variety of ways depending on how you want to handle the remaining data:
SQL
SELECT *
FROM churn_data
WHERE Churn_Date IS NOT NULL;

Advanced Considerations:

If your observation period is dynamic and spans multiple time intervals, you may need to adjust your filtering logic accordingly.
You might also want to consider the scenario where Observation_Window_End itself could be NULL, meaning there's no fixed end date for monitoring churn.

#### Closing Summary

In summary, to exclude right-censored users from a churn training set in SQL, we filter out records where the user hasn’t churned by the end of their observation window. This ensures that our model isn't trained on mislabeled data and maintains its accuracy.

#### Common Mistakes/Follow-Ups
Mislabeling Right-Censored Users: Forgetting to exclude users whose churn event has not yet occurred.
Handling Dynamic Observation Periods: Not considering the case where observation periods are not fixed.
Ignoring NULL Values: Failing to properly handle scenarios where Churn_Date or Observation_Window_End might be NULL.

**Comparison:**

| User_ID | Churn_Date   | Observation_Window_End |
|---------|--------------|------------------------|
| 001     | 2023-09-05   | 2023-10-30             |
| 002     | NULL         | 2023-10-15             |
| 003     | 2023-10-20   | NULL                   |
| User_ID | Churn_Date   | Observation_Window_End |
|---------|--------------|------------------------|
| 001     | 2023-09-05   | 2023-10-30             |
| 003     | 2023-10-20   | NULL                   |

**Code:**

```sql
-- Filter out right-censored users
SELECT *
FROM churn_data
WHERE Churn_Date IS NOT NULL;
```

**Example input / output:**

```text
Input:
User_ID | Churn_Date | Observation_Window_End
001     | 2023-09-05 | 2023-10-30
002     | NULL       | 2023-10-15
003     | 2023-10-20 | NULL

Output:
User_ID | Churn_Date | Observation_Window_End
001     | 2023-09-05 | 2023-10-30
003     | 2023-10-20 | NULL
```

**Data:**

```plaintext
User_ID | Churn_Date | Observation_Window_End
001     | 2023-09-05 | 2023-10-30
002     | NULL       | 2023-10-15
003     | 2023-10-20 | NULL
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Churn_Date NULL?
5. Exclude from training
6. Include in training
7. End

