---
type: "practice"
difficulty: "Basic"
---

# What is a CHECK constraint and give an example of enforcing valid data with it.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Adding Constraints Online](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Adding_Constraints_Online.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In PostgreSQL, a CHECK constraint is a database integrity rule that ensures only valid data is inserted into or updated in a table. It allows you to define a condition that must be true for all rows in a table.

To illustrate how a CHECK constraint can enforce valid data, let's consider an example where we have a table for managing employees. We want to ensure that no employee has a negative age. Here’s the step-by-step reasoning and solution:

Summary: A CHECK constraint is a database rule that enforces conditions on column values. It prevents invalid data from being inserted or updated.

#### Triage
The question asks for an explanation of what a CHECK constraint is and to provide an example. The core idea is to ensure only valid data (e.g., non-negative age) is stored in the database.

#### First-Principles Solution

Define the Constraint: First, we define a column or columns that should be subject to the CHECK constraint.
Create the Table: Next, we create a table with a CHECK constraint applied to one of its columns.
Insert Valid and Invalid Data: Finally, we insert both valid and invalid data into the table to see how the constraint behaves.

Let's use an example where we have an employees table with a column for age:

SQL
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    age INTEGER CHECK (age >= 0)
);

Here, the CHECK (age >= 0) constraint ensures that any value inserted into the age column must be greater than or equal to zero.

Insert Valid Data:
SQL
INSERT INTO employees (name, age) VALUES ('John Doe', 30);

This would succeed because the age is non-negative.

Insert Invalid Data:
SQL
INSERT INTO employees (name, age) VALUES ('Jane Smith', -5);

This would fail with an error message indicating that the CHECK constraint was violated.

#### Closing Summary
and Common Mistakes/Follow-Ups: In this example, we saw how a CHECK constraint can enforce valid data in PostgreSQL. It's important to note that not all constraints need to be enforced at the database level; sometimes business logic might require more complex validation, which could be handled through application code.

#### Common mistakes
include:

Overusing CHECK constraints for complex logic (better suited for application-level validation).
Failing to test constraints with both valid and invalid data.
Not considering performance implications of overly restrictive or complex constraints.

When asked about this in an interview, it's crucial to demonstrate a deep understanding of the constraint's purpose and how to use it effectively. Always be prepared to discuss trade-offs between database-level validation and application logic.

