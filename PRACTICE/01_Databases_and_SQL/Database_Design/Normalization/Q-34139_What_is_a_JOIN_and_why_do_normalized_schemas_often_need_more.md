---
type: "practice"
difficulty: "Basic"
---

# What is a JOIN and why do normalized schemas often need more of them?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Denormalization Trade-offs and Patterns](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Denormalization_Trade-offs_and_Patterns/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with databases, a JOIN is an operation that combines rows from two or more tables based on a related column between them. In the context of database design, normalized and denormalized schemas each have their use cases, but normalized schemas often require more JOINs to retrieve complete information.

Normalized schemas are designed to eliminate redundancy and improve data integrity by breaking down large tables into smaller ones with atomic columns. This means that every piece of information is stored in one place, ensuring consistency but potentially increasing the complexity when querying data.

**Comparison:**

| Employee Table (employees)                | Department Table (departments)          |
|------------------------------------------|-----------------------------------------|
| employee_id, first_name, last_name, dept_id | dept_id, department_name                 |

**Code:**

```sql
-- Example of a JOIN operation
SELECT employees.first_name, employees.last_name, departments.department_name
FROM employees
JOIN departments ON employees.dept_id = departments.dept_id;
```

**Example input / output:**

```

Input Data:

SQL
Copy
employees:
| employee_id | first_name | last_name | dept_id |
|-------------|------------|-----------|---------|
| 1           | Alice      | Smith     | 2       |
| 2           | Bob        | Johnson   | 3       |

departments:
| dept_id | department_name |
|---------|-----------------|
| 2       | HR              |
| 3       | Marketing       |

Output Data:

SQL
Copy
SELECT employees.first_name, employees.last_name, departments.department_name
FROM employees
JOIN departments ON employees.dept_id = departments.dept_id;

Result:

PLAINTEXT
Copy
| first_name | last_name | department_name |
|------------|-----------|-----------------|
| Alice      | Smith     | HR              |
| Bob        | Johnson   | Marketing       |
```

**Data:**

```text
- **Example Tables:**
  - `employees` (employee_id, first_name, last_name, dept_id)
  - `departments` (dept_id, department_name)
```

**Flow:**

1. Yes
2. No
3. Query Employees
4. Department ID Exists?
5. JOIN Departments Table
6. Retrieve Full Employee Data
7. Return Partial Data or Error

