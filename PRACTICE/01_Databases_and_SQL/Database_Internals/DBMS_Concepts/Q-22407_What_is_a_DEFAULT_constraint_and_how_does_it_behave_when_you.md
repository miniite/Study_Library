---
type: "practice"
difficulty: "Basic"
---

# What is a DEFAULT constraint and how does it behave when you omit the column in an INSERT?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Normal Forms](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Normal_Forms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A default constraint in a database management system (DBMS) is used to provide a default value for a column if no explicit value is provided during an INSERT operation. It ensures that every row inserted into the table will have a valid value, even if you forget or choose not to specify one.

#### Triaging the Question

The question asks about how a default constraint behaves when a column is omitted in an INSERT. The core of the question revolves around understanding the behavior of the DBMS with respect to default values and their application during data insertion. We need to consider how the database handles this scenario without causing errors.

#### First-Principles Solution
Default Constraint Definition: A default constraint is defined when a developer wants certain columns to have standard, pre-set values. This definition typically includes specifying the column name and the default value.
INSERT Statement Without Column Value: When you insert data into a table with a default constraint but omit the value for that specific column, the DBMS will automatically use the default value defined for that column.

#### Example Scenario
SQL
-- Assuming a table `employees` with columns `id`, `name`, and `salary`
ALTER TABLE employees ADD CONSTRAINT def_salary DEFAULT 5000 FOR salary;

-- An insert statement without specifying the `salary` value
INSERT INTO employees (id, name) VALUES (1, 'Alice');
Behavior: The above SQL will result in a row being inserted into the employees table with the values id = 1, name = 'Alice', and salary = 5000 because of the default constraint on the salary column.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, when you define a default constraint for a column, the DBMS automatically assigns that value to the column whenever no explicit value is provided during an insert. Omitting the column in an INSERT will result in the default value being used.

#### Common Mistakes

Failing to understand that omitting a column with a defined default does not throw an error but simply uses the default value.
Misinterpreting the behavior of default constraints, thinking they apply only during table creation and not dynamically during insertions.

#### Example
Follow-Up Questions
How would you handle situations where multiple columns have default constraints?
Can we enforce different types of default values for different rows?

**Code:**

```sql
-- Default constraint example
ALTER TABLE employees ADD CONSTRAINT def_salary DEFAULT 5000 FOR salary;

-- Insert statement with omission of column value
INSERT INTO employees (id, name) VALUES (1, 'Alice');
```

**Example input / output:**

```text
-- Input
ALTER TABLE employees ADD CONSTRAINT def_salary DEFAULT 5000 FOR salary;
INSERT INTO employees (id, name) VALUES (1, 'Alice');

-- Output
-- Row inserted with id = 1, name = 'Alice', and salary = 5000
```

**Data:**

```json
{
  "table": "employees",
  "columns": [
    {"name": "id", "type": "INT"},
    {"name": "name", "type": "VARCHAR(50)"},
    {"name": "salary", "type": "DECIMAL"}
  ],
  "constraints": [
    {
      "constraint_name": "def_salary",
      "column": "salary",
      "default_value": 5000
    }
  ]
}
```

