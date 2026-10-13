---
type: "practice"
difficulty: "Basic"
---

# How do you update multiple columns in a single UPDATE statement?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Upserts](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Upserts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with databases, updating multiple columns in a single UPDATE statement is a common task. This involves specifying which table to modify and what new values should be assigned to one or more columns for the rows that meet certain conditions.

#### Triage

The question asks how to perform an update operation on multiple columns using a single SQL UPDATE statement. The interviewer is probing your understanding of DML (Data Manipulation Language) commands specifically, focusing on the syntax and structure of the UPDATE statement.

#### First-Principles Solution

Let's break down the process step by step:

Identify the Table: Determine which table you need to modify.
Specify the Columns and New Values: Indicate the columns that will be updated along with their new values.
Apply Conditions (Optional): If there are specific rows to update, use a WHERE clause.

#### Example

Suppose we have a employees table, and we want to update the salary and job_title of employees who joined in 2019. The SQL query would look something like this:

SQL
UPDATE employees
SET salary = new_salary_value,
    job_title = 'new_job_title'
WHERE join_date >= '2019-01-01' AND join_date < '2020-01-01';

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the UPDATE statement allows you to change data in a table by specifying the columns to update along with their new values. Make sure to include a WHERE clause if you need to specify certain rows.

#### Common mistakes

Forgetting to include a SET clause.
Neglecting to add conditions in the WHERE clause, resulting in unintended updates.

#### Follow-up
questions might involve discussing performance implications or how different database management systems handle concurrent updates.

**Code:**

```sql
UPDATE employees
SET salary = new_salary_value,
    job_title = 'new_job_title'
WHERE join_date >= '2019-01-01' AND join_date < '2020-01-01';
```

