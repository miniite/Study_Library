---
type: "practice"
difficulty: "Intermediate"
---

# What does the DATEDIFF function calculate and what arguments does it take?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Date Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Date_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

DATEDIFF is a SQL function that calculates the difference between two dates or datetimes in terms of a specified datepart (unit of time). It's commonly used to determine intervals such as days, weeks, months, years, and so on. This function is essential for various applications like calculating age, time elapsed between events, and more.

The DATEDIFF function takes three arguments:

@datepart - The datepart value that determines the granularity of the difference calculation.
start_date - The starting point or base date from which to calculate the datepart.
end_date - The end point or target date against which to compare.

Let's break down the function and see how it works in practice:

**Comparison:**

```
Datepart	Start Date	End Date	Result
day	'2023-01-01'	'2023-05-15'	124
month	'2023-01-01'	'2023-06-15'	5
year	'2023-01-01'	'2024-01-15'	1
```

**Code:**

```sql
-- Example usage of DATEDIFF with different dateparts
SELECT
    DATEDIFF(day, '2023-01-01', '2023-05-15') AS DayDifference,
    DATEDIFF(month, '2023-01-01', '2023-06-15') AS MonthDifference,
    DATEDIFF(year, '2023-01-01', '2024-01-15') AS YearDifference;

In this example:

DATEDIFF(day, '2023-01-01', '2023-05-15') calculates the difference in days between January 1, 2023, and May 15, 2023.
DATEDIFF(month, '2023-01-01', '2023-06-15') calculates the difference in months between January 1, 2023, and June 15, 2023.
DATEDIFF(year, '2023-01-01', '2024-01-15') calculates the difference in years between January 1, 2023, and January 15, 2024.
```

**Flow:**

1. yes
2. no
3. DATEDIFF Function
4. Is datepart valid?
5. Calculate difference based ondatepart
6. Throw error

