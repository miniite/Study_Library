---
type: "practice"
difficulty: "Intermediate"
---

# Write a query to find the number of new users who signed up each month this year.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Date Functions and Grouping](<../../../../TOOLS/MySQL/MySQL_Platform/Date_Functions_and_Grouping.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, you need to write a SQL query that counts the number of new user sign-ups by month within the current year. The key here is to filter and group the data correctly based on the date of signup.

#### Triaging the Question

The question asks for the count of new users per month in the current year. We assume:

You have a table named users with at least two columns: id (user ID) and signup_date (the date when the user signed up).
The current year is determined from the system's current date.

#### Solution
Extract Current Year: Determine the start of this year by extracting the current year.
Filter Data for This Year: Use a WHERE clause to filter users who signed up in the current year.
Group By Month: Group the results by month using SQL's DATE_FORMAT or equivalent function.
Count Users: Count the number of unique users per month.
Step-by-Step Solution
SQL
SELECT
    DATE_FORMAT(signup_date, '%Y-%m') AS signup_month,
    COUNT(DISTINCT id) AS new_users_count
FROM
    users
WHERE
    YEAR(signup_date) = YEAR(CURDATE())
GROUP BY
    signup_month;

#### Closing Summary
and Common Mistakes/Follow-Ups

The query above will return a list of months (in the format YYYY-MM) along with the count of new users for each month. Here are some common pitfalls to avoid:

Ensure that your date column is correctly formatted.

#### Cons
ider edge cases where users might sign up at exactly midnight, which can be tricky depending on time zones and database configurations.

#### Follow-up
questions could involve discussing performance optimizations if this query needs to run frequently or scaling considerations if dealing with a large user base.

**Code:**

```sql
SELECT
    DATE_FORMAT(signup_date, '%Y-%m') AS signup_month,
    COUNT(DISTINCT id) AS new_users_count
FROM
    users
WHERE
    YEAR(signup_date) = YEAR(CURDATE())
GROUP BY
    signup_month;
```

