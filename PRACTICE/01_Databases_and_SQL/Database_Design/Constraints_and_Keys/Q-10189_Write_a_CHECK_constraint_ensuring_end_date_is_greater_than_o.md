---
type: "practice"
difficulty: "Intermediate"
---

# Write a CHECK constraint ensuring end_date is greater than or equal to start_date in a bookings table.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Check and NOT NULL Constraints](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Check_and_NOT_NULL_Constraints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that end_date is always greater than or equal to start_date in a bookings table, we need to write a CHECK constraint. This type of constraint ensures data integrity by restricting values to meet certain conditions. In this case, the condition is that the booking end date must be on or after the start date.

#### Triaging the Question

The question asks for a CHECK constraint implementation in SQL. The interviewer is probing your understanding of SQL syntax and constraints. We need to assume that we are working with a standard relational database system that supports CHECK constraints, such as PostgreSQL, MySQL, or SQLite.

#### First-Principles Solution

A CHECK constraint ensures the condition end_date >= start_date always holds true for each row in the bookings table. Here’s how we can implement it:

Understand the Condition: The condition is simple: end_date should be greater than or equal to start_date.
SQL Syntax: Use the CHECK keyword followed by a boolean expression.

#### Cons
traint Name: Provide a meaningful name for the constraint, like valid_booking_dates.
Full Solution

Here’s the SQL statement to add the CHECK constraint:

SQL
ALTER TABLE bookings
ADD CONSTRAINT valid_booking_dates
CHECK (end_date >= start_date);

#### Closing Summary
and Common Mistakes/Follow-Ups

This CHECK constraint ensures that any attempt to insert or update a row in the bookings table where end_date < start_date will result in an error. It’s important to note that this constraint does not enforce the dates themselves (e.g., start_date must be a date), but only their relative order.

#### Common mistakes
include:

Forgetting to use double quotes around column names, especially if they contain spaces or special characters.
Not defining a meaningful name for the constraint, which can make debugging issues harder later on.
Assuming that CHECK constraints are automatically enforced in all database systems without checking your specific system’s documentation.

If asked about potential edge cases, you might discuss scenarios where dates could be NULL, and decide whether to allow these or enforce non-null values. Additionally, if the interviewer probes further, consider discussing how this constraint interacts with other parts of the application, such as booking validation logic in a front-end interface.

**Code:**

```sql
ALTER TABLE bookings
ADD CONSTRAINT valid_booking_dates
CHECK (end_date >= start_date);
```

