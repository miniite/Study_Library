---
type: "practice"
difficulty: "Intermediate"
---

# How do you use a self-join to compute the day-over-day difference in a daily snapshot table when there's no window function support?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Attribution with Window Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Attribution_with_Window_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how to use a self-join to compute the day-over-day difference in a daily snapshot table without using window functions, let’s first break down what we need and the problem at hand.

#### Triage

The core of this question is about implementing a solution that calculates the differences between two consecutive days' data points. Since window functions are not supported, we’ll need to use a self-join on our table to achieve this. The key assumptions here include having a timestamp column and possibly other relevant columns like value or metric.

#### Solution

Let's assume you have a daily snapshot table named daily_snapshot, with columns such as date (timestamp) and value. Our goal is to compute the difference between today’s value and yesterday’s value.

Step-by-Step Breakdown
Self-Join: Join the table with itself, using the date column. We will join on the current day's data with the previous day's data.
Date Filtering: Use a condition to ensure we are matching each row with its corresponding previous day.
Difference Calculation: Subtract the value of the previous day from the value of the current day.

Here’s how you can write this in SQL:

SQL
SELECT
    t1.date AS today,
    t2.date AS yesterday,
    t1.value AS today_value,
    t2.value AS yesterday_value,
    (t1.value - t2.value) AS diff
FROM
    daily_snapshot t1
JOIN
    daily_snapshot t2 ON DATE_SUB(t1.date, INTERVAL 1 DAY) = t2.date;

#### Closing Summary
and Common Mistakes

This query will give you a table with the date of today, yesterday's date, today’s value, yesterday’s value, and the difference between them. However, there are some common pitfalls:

Handling Missing Data: If data is missing for one or both days, the JOIN might fail to match pairs.
Date Range Considerations: Ensure your table has continuous dates without gaps; otherwise, you need additional logic to handle those gaps.

#### Example
Input and Output

Let’s assume we have the following sample data:

JSON
[
  {"date": "2023-10-01", "value": 100},
  {"date": "2023-10-02", "value": 150},
  {"date": "2023-10-03", "value": 180}
]

Running the query would produce:

MARKDOWN
| today        | yesterday     | today_value | yesterday_value | diff         |
|--------------|---------------|-------------|-----------------|--------------|
| 2023-10-02   | 2023-10-01    | 150         | 100             | 50           |
| 2023-10-03   | 2023-10-02    | 180         | 150             | 30           |
Self-check

This solution is correct and complete for computing day-over-day differences without window functions.

**Code:**

```sql
SELECT
    t1.date AS today,
    t2.date AS yesterday,
    t1.value AS today_value,
    t2.value AS yesterday_value,
    (t1.value - t2.value) AS diff
FROM
    daily_snapshot t1
JOIN
    daily_snapshot t2 ON DATE_SUB(t1.date, INTERVAL 1 DAY) = t2.date;
```

**Data:**

```json
[
  {"date": "2023-10-01", "value": 100},
  {"date": "2023-10-02", "value": 150},
  {"date": "2023-10-03", "value": 180}
]
```

