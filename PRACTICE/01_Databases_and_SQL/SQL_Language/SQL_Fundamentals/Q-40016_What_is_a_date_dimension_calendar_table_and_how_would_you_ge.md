---
type: "practice"
difficulty: "Intermediate"
---

# What is a date dimension (calendar table), and how would you generate one in SQL to support time-based reporting?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Expanding Date Ranges](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Expanding_Date_Ranges.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A date dimension, also known as a calendar table, is a database table that contains all dates within a specific range, often used in data warehousing and business intelligence systems to support time-based reporting. It allows you to consistently reference dates across various dimensions and facts, ensuring accurate calculations over periods like days, weeks, months, and years.

To generate a date dimension in SQL, we need to create a table that contains all the dates within our desired range. Here’s how you can do it from first principles:

Define the Date Range: Determine the start and end date for your calendar.
Generate Dates Using Recursive CTE (Common Table Expression): Use recursion to generate sequential dates.
Calculate Derived Columns: Add columns such as weekday, month name, quarter, etc.

Let's break this down step by step:

#### Triaging the Question

The interviewer is likely looking for you to demonstrate your understanding of:

The purpose and importance of a date dimension in data warehousing.
The SQL techniques involved in generating a complete set of dates within a range.
How derived columns can be added to provide additional context.

#### First-Principles Solution
Step 1: Define the Date Range

For example, we might want to cover all dates from January 1, 2023, to December 31, 2025.

SQL
WITH RECURSIVE date_range AS (
    SELECT '2023-01-01'::DATE AS date
    UNION ALL
    SELECT DATEADD(day, 1, date)
    FROM date_range
    WHERE date < '2025-12-31'
)
Step 2: Generate the Date Table

We then select all dates from our recursive CTE and add derived columns.

SQL
SELECT
    date AS day,
    EXTRACT(WEEK FROM date) AS week_number,
    TO_CHAR(date, 'Month') AS month_name,
    EXTRACT(MONTH FROM date) AS month_number,
    EXTRACT(YEAR FROM date) AS year,
    CASE
        WHEN EXTRACT(QUARTER FROM date) = 1 THEN 'Q1'
        WHEN EXTRACT(QUARTER FROM date) = 2 THEN 'Q2'
        WHEN EXTRACT(QUARTER FROM date) = 3 THEN 'Q3'
        ELSE 'Q4'
    END AS quarter,
    CASE
        WHEN EXTRACT(DAYOFWEEK FROM date) = 1 THEN 'Sunday'
        WHEN EXTRACT(DAYOFWEEK FROM date) = 2 THEN 'Monday'
        WHEN EXTRACT(DAYOFWEEK FROM date) = 3 THEN 'Tuesday'
        WHEN EXTRACT(DAYOFWEEK FROM date) = 4 THEN 'Wednesday'
        WHEN EXTRACT(DAYOFWEEK FROM date) = 5 THEN 'Thursday'
        WHEN EXTRACT(DAYOFWEEK FROM date) = 6 THEN 'Friday'
        ELSE 'Saturday'
    END AS day_of_week
FROM
    (SELECT DISTINCT date FROM date_range) dr;
Step 3: Create the Date Dimension Table

Finally, we can create a table from this query.

SQL
CREATE TABLE date_dimension (
    id SERIAL PRIMARY KEY,
    day DATE NOT NULL,
    week_number INT,
    month_name VARCHAR(20),
    month_number INT,
    year INT,
    quarter VARCHAR(10),
    day_of_week VARCHAR(9)
);
INSERT INTO date_dimension (day, week_number, month_name, month_number, year, quarter, day_of_week)
SELECT
    dr.date AS day,
    EXTRACT(WEEK FROM dr.date) AS week_number,
    TO_CHAR(dr.date, 'Month') AS month_name,
    EXTRACT(MONTH FROM dr.date) AS month_number,
    EXTRACT(YEAR FROM dr.date) AS year,
    CASE
        WHEN EXTRACT(QUARTER FROM dr.date) = 1 THEN 'Q1'
        WHEN EXTRACT(QUARTER FROM dr.date) = 2 THEN 'Q2'
        WHEN EXTRACT(QUARTER FROM dr.date) = 3 THEN 'Q3'
        ELSE 'Q4'
    END AS quarter,
    CASE
        WHEN EXTRACT(DAYOFWEEK FROM dr.date) = 1 THEN 'Sunday'
        WHEN EXTRACT(DAYOFWEEK FROM dr.date) = 2 THEN 'Monday'
        WHEN EXTRACT(DAYOFWEEK FROM dr.date) = 3 THEN 'Tuesday'
        WHEN EXTRACT(DAYOFWEEK FROM dr.date) = 4 THEN 'Wednesday'
        WHEN EXTRACT(DAYOFWEEK FROM dr.date) = 5 THEN 'Thursday'
        WHEN EXTRACT(DAYOFWEEK FROM dr.date) = 6 THEN 'Friday'
        ELSE 'Saturday'
    END AS day_of_week
FROM
    (SELECT DISTINCT date FROM date_range) dr;

#### Closing Summary
+ Common Mistakes/Follow-Ups

Generating a date dimension in SQL is crucial for consistent time-based reporting. The key steps involve defining the date range, using recursive CTEs to generate dates, and adding derived columns. Common mistakes include:

Not covering all necessary days (e.g., missing weekends or holidays).
Failing to handle leap years correctly.
Overlooking the importance of indexing on the day column for performance.

For follow-ups, you might be asked about optimizing the date generation process, handling different time zones, or integrating the dimension into a larger data model.

**Comparison:**

| 159 | 2025-12-28 | 52 | December | 12 | 2025 | Q4 | Saturday | | 160 | 2025-12-29 | 52 | December | 12 | 2025 | Q4 | Sunday | | 161 | 2025-12-30 | 52 | December | 12 | 2025 | Q4 | Monday |

**Code:**

```sql
WITH RECURSIVE date_range AS (
    SELECT '2023-01-01'::DATE AS date
    UNION ALL
    SELECT DATEADD(day, 1, date)
    FROM date_range
    WHERE date < '2025-12-31'
),
date_dimension AS (
    SELECT
        dr.date AS day,
        EXTRACT(WEEK FROM dr.date) AS week_number,
        TO_CHAR(dr.date, 'Month') AS month_name,
        EXTRACT(MONTH FROM dr.date) AS month_number,
        EXTRACT(YEAR FROM dr.date) AS year,
        CASE
            WHEN EXTRACT(QUARTER FROM dr.date) = 1 THEN 'Q1'
            WHEN EXTRACT(QUARTER FROM dr.date) = 2 THEN 'Q2'
            WHEN EXTRACT(QUARTER FROM dr.date) = 3 THEN 'Q3'
            ELSE 'Q4'
        END AS quarter,
        CASE
            WHEN EXTRACT(DAYOFWEEK FROM dr.date) = 1 THEN 'Sunday'
            WHEN EXTRACT(DAYOFWEEK FROM dr.date) = 2 THEN 'Monday'
            WHEN EXTRACT(DAYOFWEEK FROM dr.date) = 3 THEN 'Tuesday'
            WHEN EXTRACT(DAYOFWEEK FROM dr.date) = 4 THEN 'Wednesday'
            WHEN EXTRACT(DAYOFWEEK FROM dr.date) = 5 THEN 'Thursday'
            WHEN EXTRACT(DAYOFWEEK FROM dr.date) = 6 THEN 'Friday'
            ELSE 'Saturday'
        END AS day_of_week
    FROM
        (SELECT DISTINCT date FROM date_range) dr
)
INSERT INTO date_dimension (day, week_number, month_name, month_number, year, quarter, day_of_week)
SELECT * FROM date_dimension;
```

**Example input / output:**

```

Input:

SQL
Copy
WITH RECURSIVE date_range AS (
    SELECT '2023-01-01'::DATE AS date
    UNION ALL
    SELECT DATEADD(day, 1, date)
    FROM date_range
    WHERE date < '2025-12-31'
),
date_dimension AS (
    SELECT
        dr.date AS day,
        EXTRACT(WEEK FROM dr.date) AS week_number,
        TO_CHAR(dr.date, 'Month') AS month_name,
        EXTRACT(MONTH FROM dr.date) AS month_number,
        EXTRACT(YEAR FROM dr.date) AS year,
        CASE
            WHEN EXTRACT(QUARTER FROM dr.date) = 1 THEN 'Q1'
            WHEN EXTRACT(QUARTER FROM dr.date) = 2 THEN 'Q2'
            WHEN EXTRACT(QUARTER FROM dr.date) = 3 THEN 'Q3'
            ELSE 'Q4'
        END AS quarter,
        CASE
            WHEN EXTRACT(DAYOFWEEK FROM dr.date) = 1 THEN 'Sunday'
            WHEN EXTRACT(DAYOFWEEK FROM dr.date) = 2 THEN 'Monday'
            WHEN EXTRACT(DAYOFWEEK FROM dr.date) = 3 THEN 'Tuesday'
            WHEN EXTRACT(DAYOFWEEK FROM dr.date) = 4 THEN 'Wednesday'
            WHEN EXTRACT(DAYOFWEEK FROM dr.date) = 5 THEN 'Thursday'
            WHEN EXTRACT(DAYOFWEEK FROM dr.date) = 6 THEN 'Friday'
            ELSE 'Saturday'
        END AS day_of_week
    FROM
        (SELECT DISTINCT date FROM date_range) dr
)
INSERT INTO date_dimension (day, week_number, month_name, month_number, year, quarter, day_of_week)
SELECT * FROM date_dimension;

Output:

CODE
Copy
id |   day   | week_number | month_name  | month_number | year | quarter |    day_of_week
----+---------+-------------+-------------+--------------+-------+---------+------------------
(160 rows)
```

**Data:**

```json
{
  "startDate": "2023-01-01",
  "endDate": "2025-12-31"
}
```

**Flow:**

1. Yes
2. Define Date Range
3. Use Recursive CTE?
4. Generate Dates
5. Add Derived Columns
6. Create Table
7. Insert Data into Table

