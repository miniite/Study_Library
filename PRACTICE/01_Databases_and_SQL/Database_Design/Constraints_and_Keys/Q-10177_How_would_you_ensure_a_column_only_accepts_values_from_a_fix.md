---
type: "practice"
difficulty: "Intermediate"
---

# How would you ensure a column only accepts values from a fixed set such as 'ACTIVE', 'INACTIVE', 'PENDING'?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Check and NOT NULL Constraints](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Check_and_NOT_NULL_Constraints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a column in your database only accepts specific, predefined values like 'ACTIVE', 'INACTIVE', and 'PENDING', we can use a combination of constraints and keys at the database level. This ensures data integrity and consistency across our tables. Here's a step-by-step approach:

Understand What We're Probing: The question is about ensuring data validation at the column level in a database to prevent the insertion or update of invalid values.

#### Assumptions
We are working with a relational database system (like MySQL, PostgreSQL, etc.) and we have a table named Users with a column status.

#### First Principles Solution
Use a CHECK constraint to enforce that only certain values can be inserted or updated in the status column.
Create an enumerated type if supported by your database system (e.g., PostgreSQL).

Let's dive into the details:

Step 1: Define the Enumerated Type

If you are using PostgreSQL, which supports enumerated types, we can define a custom data type for statuses.

SQL
-- PostgreSQL Example
CREATE TYPE status_type AS ENUM ('ACTIVE', 'INACTIVE', 'PENDING');
Step 2: Create the Table with the CHECK Constraint

Next, we create our Users table using this custom type or by defining a CHECK constraint to enforce valid values.

Using Enumerated Type:
SQL
CREATE TABLE Users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    status status_type -- Use the ENUM defined earlier
);
Using CHECK Constraint (if not supported by your DBMS):

If your database system does not support enumerated types, you can use a CHECK constraint instead:

SQL
CREATE TABLE Users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    status VARCHAR CHECK (status IN ('ACTIVE', 'INACTIVE', 'PENDING'))
);
Step 3: Handling Inheritance and Other Tables

If the status column is used across multiple tables, you can define the type or constraint in a common base table and use it as needed.

SQL
-- Base Table with Status Type
CREATE TABLE Users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    status status_type -- Assuming ENUM is defined earlier
);

CREATE TABLE Employees (
    id SERIAL PRIMARY KEY,
    employee_id INT REFERENCES Users(id) ON DELETE CASCADE,
    department VARCHAR(50),
    status status_type -- Assuming ENUM is defined earlier
);

#### Closing Summary
and Common Mistakes/Follow-Ups:
Mistakes: Forgetting to define the type or constraint, allowing invalid data to enter.

#### Follow-Up
How would you handle changes in the set of valid statuses? Would it be better to use a lookup table for more flexibility?

This approach ensures that only predefined values can be stored in the status column, maintaining data integrity and consistency. Using enumerated types or CHECK constraints is crucial for this validation.

**Code:**

```sql
-- PostgreSQL Enum Example
CREATE TYPE status_type AS ENUM ('ACTIVE', 'INACTIVE', 'PENDING');

-- Table Definition with ENUM
CREATE TABLE Users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    status status_type -- Use the ENUM defined earlier
);

-- Table Definition with CHECK Constraint (if no ENUM support)
CREATE TABLE Users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    status VARCHAR CHECK (status IN ('ACTIVE', 'INACTIVE', 'PENDING'))
);
```

**Example input / output:**

```text
-- Valid Insertion
INSERT INTO Users (name, status) VALUES ('Dave', 'ACTIVE'); -- Succeeds

-- Invalid Insertion
INSERT INTO Users (name, status) VALUES ('Eve', 'RETIRED'); -- Fails with error
```

**Data:**

```sql
-- Sample Data for Users Table
INSERT INTO Users (name, status) VALUES ('Alice', 'ACTIVE');
INSERT INTO Users (name, status) VALUES ('Bob', 'INACTIVE');
INSERT INTO Users (name, status) VALUES ('Charlie', 'PENDING');
```

