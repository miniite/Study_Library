---
type: "practice"
difficulty: "Intermediate"
---

# In a simple pipeline what steps would you follow to turn one denormalized export into 1NF then 2NF then 3NF?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Denormalization in Lakes](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Denormalization_in_Lakes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To transform a single denormalized export into its normalized forms (1NF, 2NF, and 3NF), we need to follow systematic steps that address data redundancy, dependency issues, and atomicity constraints progressively. Here’s an overview of the process:

First, let's triage what this question is asking: The goal is to understand how to systematically normalize a dataset from its initial denormalized state to achieve various levels of normalization (1NF, 2NF, and 3NF). This involves identifying and resolving redundancy issues at each step.

Step-by-Step Solution
Initial Denormalized Export

Assume we have a single CSV file containing employee data with repeated columns like employee_name, department_name, and manager_id. Each row represents an employee, but the data is redundant since department names and manager IDs are listed for each employee.

1NF: Ensuring Atomicity

Identify Non-Atomic Columns: The first step in normalization is to ensure that all data elements are atomic (indivisible). For example, department_name and manager_id might be repeated multiple times across the dataset.

Eliminate Repeated Data: Create separate tables for departments and managers. This ensures each piece of data is stored only once.

MARKDOWN
# Employees Table
| employee_id | name    |
+-------------+---------+
| 1           | Alice   |
| 2           | Bob     |
| ...         | ...     |

# Departments Table
| department_id | name       |
+---------------+------------+
| 100           | Engineering|
| 101           | Sales      |
| ...           | ...        |

# Managers Table
| manager_id    | name      |
+---------------+-----------+
| 201           | Charlie   |
| 202           | David     |
| ...           | ...       |
2NF: Eliminating Partial Dependencies

Identify Transitive Dependencies: After ensuring atomicity, we need to check for partial dependencies where a non-key attribute depends on only part of the primary key.

Create Additional Tables if Necessary: If a department's name is dependent on an employee but not a manager, it might make sense to keep department_id in both tables.

MARKDOWN
# Employees Table (revised)
| employee_id | name    | department_id |
+-------------+---------+---------------+
| 1           | Alice   | 100           |
| 2           | Bob     | 100           |
| ...         | ...     | ...           |

# Departments Table
| department_id | name       |
+---------------+------------+
| 100           | Engineering|
| 101           | Sales      |
| ...           | ...        |

# Managers Table (revised)
| manager_id    | name      | department_id |
+---------------+-----------+---------------+
| 201           | Charlie   | 100           |
| 202           | David     | 101           |
| ...           | ...       | ...           |
3NF: Eliminating Transitive Dependencies

Identify Transitive Dependencies: Now, we need to ensure that every non-key attribute in a table is fully dependent on the primary key and not another non-key.

Remove Redundant Columns: If manager_id is no longer needed as an independent column but still belongs in the relationship between employees and managers, it should be moved appropriately.

MARKDOWN
# Employees Table (final)
| employee_id | name    | department_id |
+-------------+---------+---------------+
| 1           | Alice   | 100           |
| 2           | Bob     | 100           |
| ...         | ...     | ...           |

# Departments Table (final)
| department_id | name       |
+---------------+------------+
| 100           | Engineering|
| 101           | Sales      |
| ...           | ...        |

# Managers Table (final)
| manager_id    | name      | department_id |
+---------------+-----------+---------------+
| 201           | Charlie   | 100           |
| 202           | David     | 101           |
| ...           | ...       | ...           |

# EmployeeManagers Table
| employee_id | manager_id |
+-------------+-----------+
| 1           | 201        |
| 2           | 201        |
| ...         | ...        |

#### Closing Summary
and Common Mistakes

By following these steps, we have transformed our initial denormalized export into a fully normalized form (3NF). The key takeaways are:

Atomicity: Ensuring that data elements are indivisible.
Partial Dependencies: Addressing dependencies on part of the primary key.
Transitive Dependencies: Removing redundant columns to ensure all non-key attributes depend only on the primary key.

#### Common mistakes
include over-normalizing (introducing too many tables with minimal relationships), under-normalizing (leaving partial and transitive dependencies unresolved), or failing to fully decompose relationships leading to anomalies in the database.

**Comparison:**

```
Table Name	Columns
Employees	employee_id, name, department_id
Departments	department_id, name
Managers	manager_id, name, department_id
EmployeeManagers	employee_id, manager_id
```

**Code:**

```sql
-- SQL for creating the normalized schema

CREATE TABLE Employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(255),
    department_id INT
);

CREATE TABLE Departments (
    department_id INT PRIMARY KEY,
    name VARCHAR(255)
);

CREATE TABLE Managers (
    manager_id INT PRIMARY KEY,
    name VARCHAR(255),
    department_id INT
);

CREATE TABLE EmployeeManagers (
    employee_id INT,
    manager_id INT,
    PRIMARY KEY (employee_id, manager_id),
    FOREIGN KEY (employee_id) REFERENCES Employees(employee_id),
    FOREIGN KEY (manager_id) REFERENCES Managers(manager_id)
);
```

**Example input / output:**

```text
-- Example input data

INSERT INTO Employees (employee_id, name, department_id) VALUES (1, 'Alice', 100);
INSERT INTO Employees (employee_id, name, department_id) VALUES (2, 'Bob', 100);

INSERT INTO Departments (department_id, name) VALUES (100, 'Engineering');

INSERT INTO Managers (manager_id, name, department_id) VALUES (201, 'Charlie', 100);
```

**Data:**

```json
[
  {
    "employee_id": 1,
    "name": "Alice",
    "department_id": 100
  },
  {
    "employee_id": 2,
    "name": "Bob",
    "department_id": 100
  }
]

{
  "department_id": 100,
  "name": "Engineering"
}

{
  "manager_id": 201,
  "name": "Charlie",
  "department_id": 100
}
```

**Flow:**

1. Denormalized Export
2. Identify Non-Atomic Columns
3. Eliminate Repeated Data
4. Create Separate Tables
5. Ensure Atomicity
6. Identify Partial Dependencies
7. Remove Redundant Columns
8. Create Additional Tables ifNecessary
9. Eliminate TransitiveDependencies
10. Final Normalized Schema

