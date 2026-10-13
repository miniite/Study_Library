---
type: "practice"
difficulty: "Intermediate"
---

# What does ON UPDATE CASCADE do and when is it useful?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Referential Actions](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Referential_Actions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

ON UPDATE CASCADE in database constraints refers to an action that automatically propagates changes from a parent table to related child tables. This means if you update a primary key (PK) or foreign key (FK), the related records will be updated accordingly. It's particularly useful when maintaining referential integrity across multiple tables.

#### Triage

The question is asking about ON UPDATE CASCADE and its utility. The interviewer wants to understand your knowledge of database constraints, specifically how they interact with relationships between tables. They are probing whether you can explain the mechanism and provide examples where such a feature might be beneficial.

#### First-Principles Solution

ON UPDATE CASCADE works in conjunction with foreign keys (FKs). When you update a primary key (PK) in one table that is referenced by another table via a foreign key, ON UPDATE CASCADE ensures that all related records in the child table are updated to reflect the new value. Here’s a step-by-step explanation:

Define Foreign Key and Primary Key Constraints: First, ensure both tables have appropriate primary keys (PKs) and foreign keys (FKs).
Enable Cascading Updates: Use the ON UPDATE CASCADE clause in your foreign key definition to specify that updates should be propagated.

#### Example Scenario
Table A (parent_table) has a PK column id.
Table B (child_table) has an FK column parent_id referencing id from parent_table.

If you update the value in id in parent_table, ON UPDATE CASCADE will automatically update all records in child_table where parent_id matches the new id value.

Detailed Example

Suppose we have two tables: employees and departments. The employees table has a foreign key department_id that references the primary key id in the departments table. We want to ensure that if a department is renamed or moved, all related employees are updated.

SQL
CREATE TABLE departments (
    id INT PRIMARY KEY,
    name VARCHAR(255)
);

CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(id) ON UPDATE CASCADE
);

Now, if we update the name of a department in the departments table:

SQL
UPDATE departments SET name = 'New Name' WHERE id = 1;

This query will automatically update all employees in the employees table whose department_id is 1.

#### Closing Summary
+ Common Mistakes/Follow-Ups

ON UPDATE CASCADE is useful for maintaining referential integrity, especially when dealing with large datasets or complex relationships. However, it should be used judiciously because it can lead to unintended data changes if not carefully managed.

#### Common mistakes
include:

Forgetting to define the foreign key constraint correctly.
Overusing ON UPDATE CASCADE, which can introduce unnecessary complexity and potential data loss issues.
Not considering performance implications; cascading updates can be slow on large tables.

#### Follow-ups
might involve discussing trade-offs between referential integrity and performance, or scenarios where other constraints like ON DELETE SET NULL would be more appropriate.

**Code:**

```sql
CREATE TABLE departments (
    id INT PRIMARY KEY,
    name VARCHAR(255)
);

CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(id) ON UPDATE CASCADE
);
```

**Example input / output:**

```text
INSERT INTO departments (id, name) VALUES (1, 'Engineering');
INSERT INTO employees (id, name, department_id) VALUES (1, 'Alice', 1);

-- Update the department name and observe the effect on employees.
UPDATE departments SET name = 'New Engineering' WHERE id = 1;
```

