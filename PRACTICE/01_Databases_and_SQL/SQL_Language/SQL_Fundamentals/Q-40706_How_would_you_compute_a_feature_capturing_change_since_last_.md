---
type: "practice"
difficulty: "Intermediate"
---

# How would you compute a feature capturing 'change since last week' per entity in SQL, handling entities with sparse history so the comparison doesn't silently produce NULL for everyone?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Analytical SQL for ML Features](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Analytical_SQL_for_ML_Features.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of computing a "change since last week" per entity in SQL while handling sparse history gracefully, we need to ensure that if an entity does not have any records from the previous week, it doesn't return NULL but instead returns 0 or some other default value. This involves using window functions and possibly some conditional logic.

#### Triage
The question is essentially asking how to perform a rolling comparison between current values and their corresponding values from one week ago for each entity. We need to handle cases where the entity might not have records from last week, so we should ensure that such entities are appropriately treated.

#### Assumptions

We have a table named entity_data with columns like id, date, and value.
The date column is in a format compatible with SQL date functions (e.g., YYYY-MM-DD).
"Last week" can be determined by subtracting 7 days from the current date.

Step-by-step Solution:

First, identify the relevant dates: today's date and last week’s date.
Use a window function to partition data by entity and order it by date.
Calculate the difference between the current value and the value from one week ago for each entity.

SQL Code:

SQL
WITH latest_value AS (
    SELECT
        id,
        value,
        LAG(value) OVER (PARTITION BY id ORDER BY date) AS last_week_value
    FROM
        entity_data
    WHERE
        date >= DATEADD(day, -7, CURRENT_DATE())
),
with_changes AS (
    SELECT
        id,
        value,
        last_week_value,
        COALESCE(value - last_week_value, 0) AS change_since_last_week
    FROM
        latest_value
)
SELECT * FROM with_changes;

Explanation:

The latest_value CTE uses the LAG function to get the value from one week ago for each entity. It only considers records within the past week.
In the with_changes CTE, we compute the change by subtracting last_week_value from value. Using COALESCE ensures that if there is no last week's value (i.e., NULL), it defaults to 0.

#### Closing Summary
By leveraging window functions and conditional logic in SQL, we can accurately calculate the "change since last week" for each entity while gracefully handling cases where an entity lacks records from last week. This approach ensures consistency and reliability in our feature computation. Common Mistakes/Follow-Ups:

Forgetting to filter the data by a specific date range could lead to incorrect results.
Not using COALESCE can result in NULL values for entities with sparse history.

**Code:**

```sql
WITH latest_value AS (
    SELECT
        id,
        value,
        LAG(value) OVER (PARTITION BY id ORDER BY date) AS last_week_value
    FROM
        entity_data
    WHERE
        date >= DATEADD(day, -7, CURRENT_DATE())
),
with_changes AS (
    SELECT
        id,
        value,
        last_week_value,
        COALESCE(value - last_week_value, 0) AS change_since_last_week
    FROM
        latest_value
)
SELECT * FROM with_changes;
```

