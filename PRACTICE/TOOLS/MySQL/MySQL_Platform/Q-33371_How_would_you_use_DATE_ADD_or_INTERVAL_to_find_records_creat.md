---
type: "practice"
difficulty: "Intermediate"
---

# How would you use DATE_ADD or INTERVAL to find records created in the last 7 days?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Date Arithmetic](<../../../../TOOLS/MySQL/MySQL_Platform/Date_Arithmetic.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To find records created in the last 7 days using MySQL, we can leverage the DATE_ADD function along with the INTERVAL keyword. This approach allows us to filter out only those rows that were created within a specific time frame, which is crucial for data retrieval tasks such as generating recent activity reports or analyzing new user sign-ups.

Firstly, let's understand what the question is really asking and what we need to do:

Question Triage: The interviewer wants to see how you can use date manipulation functions in MySQL to filter records based on a specific time period.

#### Assumptions
We assume there is a table that contains a timestamp or date column for record creation dates, typically named something like created_at.

Now, let's break down the solution from first principles:

Identify the Date Column: The first step is to identify which column in your database stores the creation date of each record.
Use DATE_ADD and INTERVAL: We can use DATE_ADD along with INTERVAL to subtract 7 days from the current date, thereby generating a range of dates that represent the last 7 days.
Apply the Condition in SQL Query: Finally, we apply this generated range as a condition in our SELECT query.

Here is an example using a table named users with a created_at column:

SQL
SELECT *
FROM users
WHERE created_at >= DATE_SUB(CURDATE(), INTERVAL 7 DAY);

Let's walk through the key parts of this solution:

CURDATE(): This function returns the current date.
DATE_SUB: This is used to subtract a specified number of days from the current date. In our case, we are subtracting 7 days.
INTERVAL 7 DAY: Specifies that 7 days should be subtracted.

This query will return all records from the users table where the created_at timestamp is within the last 7 days.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, using DATE_SUB with an INTERVAL of 7 days from the current date (CURDATE()) is a straightforward way to filter records based on a specific time period. A common mistake would be forgetting to use CURDATE() and instead using CURRENT_DATE, which returns the current date as a string, not as a date type.

For follow-ups, you might be asked about performance considerations if the table is very large or how this query could be optimized for better performance.

