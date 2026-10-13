---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd compute business-day-based moving windows (skipping weekends/holidays) using a dense business-day sequence number rather than raw dates.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Time-based Windows](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Time-based_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute business-day-based moving windows, we need to account for non-working days like weekends and holidays. The key idea is to use a dense business-day sequence number instead of raw dates. This approach ensures that our window calculations are based on the actual business day count rather than the calendar date.

#### Triaging the Question
The question is asking how to create moving windows in SQL, where each window spans a fixed number of business days, skipping weekends and holidays. The interviewer might be probing your understanding of window functions, calendar manipulation, and handling non-standard periods like business days.

#### Assumptions
and Context:

We have a table with data points indexed by date.
Business days are typically Monday to Friday, but some organizations may have additional rules for holidays.
Our goal is to create a moving window that skips weekends and holidays.

Step-by-Step Solution:

Generate the Dense Business-Day Sequence: First, we need to generate a sequence of business-day numbers. This can be done by creating a calendar table that includes all dates with their corresponding day-of-the-week and holiday status. Then, we filter out non-business days (weekends and holidays) and map each date to its dense sequence number.

Window Function Setup: Using the generated dense business-day sequence numbers, we can create our moving windows based on this sequence rather than the raw dates. This ensures that the window calculations are aligned with actual business day counts.

Define the Window Specification: The window function will be defined to include a specified number of preceding and following business days. For example, if we want to compute a 5-day moving average, we'll define our window as including the current date plus 4 previous dates in terms of dense sequence numbers.

Here is an illustrative example for a SQL solution:

SQL
WITH BusinessDays AS (
    -- Generate calendar table with day-of-week and holiday flags
    SELECT
        DATEADD(day, ROW_NUMBER() OVER (ORDER BY DATE) - 1, '2023-01-01') AS date,
        DATENAME(WEEKDAY, DATEADD(day, ROW_NUMBER() OVER (ORDER BY DATE) - 1, '2023-01-01')) AS weekday,
        CASE
            WHEN DATEADD(day, ROW_NUMBER() OVER (ORDER BY DATE) - 1, '2023-01-01') IN ('2023-10-09', '2023-12-25')
            THEN 1
            ELSE 0
        END AS is_holiday,
    ROW_NUMBER() OVER (ORDER BY DATE) - 1 AS dense_sequence_number
),
FilteredBusinessDays AS (
    -- Filter out non-business days and generate dense sequence number for business days only
    SELECT
        date, weekday, is_holiday, dense_sequence_number
    FROM
        BusinessDays
    WHERE
        weekday NOT IN ('Saturday', 'Sunday') AND is_holiday = 0
)
SELECT
    fbd1.dense_sequence_number,
    SUM(data_value) OVER (ORDER BY fbd1.dense_sequence_number ROWS BETWEEN 4 PRECEDING AND CURRENT ROW) AS moving_sum_5_days
FROM
    (SELECT * FROM FilteredBusinessDays WHERE date = '2023-10-10') fbd1

#### Closing Summary
By using a dense business-day sequence number, we can create moving windows that respect the actual flow of business days. This approach ensures accurate calculations even when weekends and holidays are involved.

#### Common Mistakes/Follow-Ups

Forgetting to filter out non-business days before generating the dense sequence.
Incorrectly defining the window specification in terms of raw dates instead of dense sequence numbers.
Not considering edge cases where the start or end of a month might include holidays, affecting the sequence count.

**Comparison:**

```
Generate Business Days
Filter Non-Business Days
Assign Dense SequenceNumbers
Compute Moving Sum
```

**Code:**

```sql
WITH BusinessDays AS (
    SELECT
        DATEADD(day, ROW_NUMBER() OVER (ORDER BY DATE) - 1, '2023-01-01') AS date,
        DATENAME(WEEKDAY, DATEADD(day, ROW_NUMBER() OVER (ORDER BY DATE) - 1, '2023-01-01')) AS weekday,
        CASE
            WHEN DATEADD(day, ROW_NUMBER() OVER (ORDER BY DATE) - 1, '2023-01-01') IN ('2023-10-09', '2023-12-25')
            THEN 1
            ELSE 0
        END AS is_holiday,
    ROW_NUMBER() OVER (ORDER BY DATE) - 1 AS dense_sequence_number
),
FilteredBusinessDays AS (
    SELECT
        date, weekday, is_holiday, dense_sequence_number
    FROM
        BusinessDays
    WHERE
        weekday NOT IN ('Saturday', 'Sunday') AND is_holiday = 0
)
SELECT
    fbd1.dense_sequence_number,
    SUM(data_value) OVER (ORDER BY fbd1.dense_sequence_number ROWS BETWEEN 4 PRECEDING AND CURRENT ROW) AS moving_sum_5_days
FROM
    (SELECT * FROM FilteredBusinessDays WHERE date = '2023-10-10') fbd1
```

**Example input / output:**

```

WITH BusinessDays AS ( SELECT DATEADD(day, ROW_NUMBER() OVER (ORDER BY DATE) - 1, '2023-09-25') AS date, DATENAME(WEEKDAY, DATEADD(day, ROW_NUMBER() OVER (ORDER BY DATE) - 1, '2023-09-25')) AS weekday, CASE WHEN DATEADD(day, ROW_NUMBER() OVER (ORDER BY DATE) - 1, '2023-09-25') IN ('2023-10-09', '2023-12-25') THEN 1 ELSE 0 END AS is_holiday, ROW_NUMBER() OVER (ORDER BY DATE) - 1 AS dense_sequence_number ), FilteredBusinessDays AS ( SELECT date, weekday, is_holiday, dense_sequence_number FROM BusinessDays WHERE weekday NOT IN ('Saturday', 'Sunday') AND is_holiday = 0 ) SELECT fbd1.dense_sequence_number, SUM(data_value) OVER (ORDER BY fbd1.dense_sequence_number ROWS BETWEEN 4 PRECEDING AND CURRENT ROW) AS moving_sum_5_days FROM (SELECT * FROM FilteredBusinessDays WHERE date = '2023-10-10') fbd1;

CODE
Copy
Result:
```sql
| dense_sequence_number | moving_sum_5_days |
|-----------------------|------------------|
| 7                     | 14.0             |
```

**Data:**

```json
{
  "date": ["2023-09-25", "2023-09-26", "2023-09-27", "2023-09-28", "2023-09-29"],
  "weekday": ["Tuesday", "Wednesday", "Thursday", "Friday", "Monday"]
}
```

**Flow:**

1. Input Data
2. Generate Business DayCalendar
3. Filter Out Weekends andHolidays
4. Assign Dense SequenceNumbers to Business DaysOnly
5. Define Window Specification(5-day Moving Sum)
6. Compute 5-Day Moving SumUsing Generated Sequence

