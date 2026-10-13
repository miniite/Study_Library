---
type: "practice"
difficulty: "Intermediate"
---

# What is a self-referencing foreign key and give a use case?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Polymorphic Associations](<../../../../01_Databases_and_SQL/Database_Design/Schema_Design/Polymorphic_Associations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database design, a self-referencing foreign key is a type of foreign key that references back to its own primary key in a table. This allows you to represent relationships where entities can relate to themselves—such as in hierarchical data structures or when dealing with parent-child relationships within the same entity.

#### Triage

The question is asking for an explanation of self-referencing foreign keys and providing a practical example of their use case. The interviewer may be probing your understanding of relational databases, normalization, and how to model complex relationships.

#### Solution
from First Principles

Let's break down why we need this concept and explore its application in a real-world scenario:

Understanding the Concept: A foreign key is used to establish a relationship between two tables where one table’s primary key is referenced by another table as a foreign key. In self-referencing foreign keys, these relationships are established within the same table.

Use Case Scenario: Imagine designing an organization's hierarchy system. Each employee can have one or more subordinates, creating a parent-child relationship within the "employees" table itself.

Schema Design: We need to define both the primary key and foreign key constraints in such a way that they support this hierarchical structure.

Step-by-Step Solution
Identify the Entity: Let's use the "employees" table as an example.
Primary Key: Assign a unique identifier for each employee, e.g., employee_id.
Foreign Key: Create another column in the same table to reference the primary key of the parent (higher-level) employee.

Here’s how you might set this up:

SQL
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    manager_id INT,  -- This is a self-referencing foreign key
    FOREIGN KEY (manager_id) REFERENCES employees(employee_id)
);

#### Example
Data

Let’s populate the table with some sample data:

SQL
INSERT INTO employees (employee_id, name, manager_id) VALUES
(1, 'Alice', NULL),
(2, 'Bob', 1),
(3, 'Charlie', 1),
(4, 'David', 2);

In this setup:

Alice is the root employee with no manager.
Bob and Charlie report to Alice, hence their manager_id is 1.
David reports directly to Bob, making his manager_id 2.

#### Common Mistakes
and Follow-Ups

#### Common mistakes
might include not properly defining the foreign key constraints or ignoring normalization rules, leading to data inconsistencies. The interviewer might follow up with questions like:

How would you ensure referential integrity in such a schema?
What are the implications of using self-referencing tables for performance and complexity?

#### Closing Summary

Self-referencing foreign keys enable modeling complex hierarchical relationships within the same table. By defining appropriate primary and foreign key constraints, we can accurately represent such structures while ensuring data consistency.

Self-Check

Correctness + completeness confirmed.

**Comparison:**

| employee_id | name   | manager_id |
|-------------|--------|------------|
| 1           | Alice  | NULL       |
| 2           | Bob    | 1          |
| 3           | Charlie| 1          |
| 4           | David  | 2          |

**Code:**

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    manager_id INT,
    FOREIGN KEY (manager_id) REFERENCES employees(employee_id)
);

INSERT INTO employees (employee_id, name, manager_id) VALUES
(1, 'Alice', NULL),
(2, 'Bob', 1),
(3, 'Charlie', 1),
(4, 'David', 2);
```

**Example input / output:**

```text
Input:
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    manager_id INT,
    FOREIGN KEY (manager_id) REFERENCES employees(employee_id)
);

INSERT INTO employees (employee_id, name, manager_id) VALUES
(1, 'Alice', NULL),
(2, 'Bob', 1),
(3, 'Charlie', 1),
(4, 'David', 2);

Output:
-- Data inserted successfully into the employees table with a hierarchical structure.
```

**Data:**

```sql
-- Sample data for the employees table
INSERT INTO employees (employee_id, name, manager_id) VALUES
(1, 'Alice', NULL),
(2, 'Bob', 1),
(3, 'Charlie', 1),
(4, 'David', 2);
```

**Flow:**

1. Employee Table
2. Primary Key (employee_id)
3. Foreign Key (manager_id)

