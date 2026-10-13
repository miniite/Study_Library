---
type: "practice"
difficulty: "Basic"
---

# Write the SQL to add a foreign key on dept_id in employees referencing departments.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Foreign Keys](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Foreign_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of adding a foreign key constraint on dept_id in the employees table that references the departments table, we need to understand the basic concepts of primary keys (PK) and foreign keys (FK). A primary key uniquely identifies each record in a database table, while a foreign key is used to establish relationships between two tables. Let's break down the steps needed to achieve this:

Identify Existing Keys: We first need to identify the primary key of the departments table.
Define the Foreign Key Constraint: Using SQL, we will add a constraint on the employees table that ensures dept_id values match those in the departments table’s primary key.

#### Triaging the Question

The question is asking for an SQL command to create a foreign key relationship between two tables: employees and departments. The interviewer might be probing your understanding of database normalization, data integrity, and relational database design. We assume that the dept_id in the employees table should reference the primary key (let's denote it as dept_id) in the departments table.

#### First-Principles Solution
Identify Primary Key: Check if dept_id is a primary key or not in both tables.
Create Foreign Key Constraint: Use the ALTER TABLE statement to add a foreign key constraint on the employees table referencing the departments table.
Sample SQL Command

Here’s how you can write the SQL command:

SQL
ALTER TABLE employees
ADD CONSTRAINT fk_dept_id FOREIGN KEY (dept_id)
REFERENCES departments(dept_id);

This command will ensure that any value in the dept_id column of the employees table must exist as a value in the dept_id column of the departments table.

#### Closing Summary
and Common Mistakes

In this scenario, it’s important to correctly identify the primary key in both tables. If there is no primary key defined on dept_id, you might need to define one first before creating the foreign key constraint. Another common mistake could be using a column name that does not exist or having mismatched data types between the two columns.

#### Common Mistakes
Incorrect Column Names: Using an incorrect column name for either table.
Data Type Mismatch: Ensuring both columns have compatible data types (usually integer).
Missing Primary Key on Departments Table: If dept_id is not a primary key, you might need to define it first.

**Code:**

```sql
ALTER TABLE employees
ADD CONSTRAINT fk_dept_id FOREIGN KEY (dept_id)
REFERENCES departments(dept_id);
```

