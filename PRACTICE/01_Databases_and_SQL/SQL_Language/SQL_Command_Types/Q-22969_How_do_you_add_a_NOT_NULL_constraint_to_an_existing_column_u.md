---
type: "practice"
difficulty: "Intermediate"
---

# How do you add a NOT NULL constraint to an existing column using ALTER?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Online Schema Changes](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Online_Schema_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To add a NOT NULL constraint to an existing column in a database, we use the ALTER TABLE statement combined with the ADD CONSTRAINT clause. This operation involves understanding DDL (Data Definition Language), specifically how to modify table structure, and ensuring that any data already present does not violate this new constraint.

#### Triaging the Question

The question is asking about modifying an existing database schema by adding a NOT NULL constraint to a specific column. The interviewer is probing your knowledge of SQL syntax, understanding of constraints, and potential edge cases or pitfalls when altering tables.

#### First-Principles Solution

To add a NOT NULL constraint to an existing column, you need to write the following SQL statement:

SQL
ALTER TABLE table_name
ADD CONSTRAINT constraint_name
CHECK (column_name IS NOT NULL);

However, this approach may not be straightforward if there is already data in the column that violates the new constraint. In such cases, you might first update all existing rows to ensure they meet the NOT NULL requirement.

Step-by-Step Reasoning
Identify the Column: Determine which column needs a NOT NULL constraint.
Check Existing Data: Verify if there are any null values in that column.
Update Existing Rows: If there are null values, update them to non-null values before adding the constraint.
Add the Constraint: Use the ALTER TABLE statement with the appropriate syntax.

#### Example

Let’s say we have a table named employees and want to add a NOT NULL constraint to the salary column:

Check Existing Data:

SQL
SELECT COUNT(*) FROM employees WHERE salary IS NULL;

This query returns the count of rows where salary is null.

Update Null Values (if necessary): If there are any null values, you might use a statement like this to update them:

SQL
UPDATE employees SET salary = 0 WHERE salary IS NULL; -- or some other default value

Add the Constraint: After ensuring all rows have non-null values in salary, add the constraint:

SQL
ALTER TABLE employees ADD CONSTRAINT emp_salary_not_null CHECK (salary IS NOT NULL);

#### Closing Summary
and Common Mistakes

In summary, adding a NOT NULL constraint to an existing column involves first checking for nulls, updating those rows if necessary, and then applying the constraint. A common mistake is attempting to add the constraint without verifying or handling existing null values, which can result in errors or data integrity issues.

Common Follow-Ups
Handling Default Values: Discuss how you might handle default values if a NOT NULL column should not allow zero or other specific values.
Transaction Management: Explain how transactions could be used to ensure that the constraint addition and data update are atomic operations.
Rollback Strategy: Consider scenarios where an error occurs during the process; discuss rollback strategies.

