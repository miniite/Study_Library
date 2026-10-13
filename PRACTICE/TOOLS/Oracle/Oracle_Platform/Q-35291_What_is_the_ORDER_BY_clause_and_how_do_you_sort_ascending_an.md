---
type: "practice"
difficulty: "Basic"
---

# What is the ORDER BY clause and how do you sort ascending and descending with NULLS FIRST or NULLS LAST in Oracle?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Oracle Data Types](<../../../../TOOLS/Oracle/Oracle_Platform/Oracle_Data_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The ORDER BY clause in Oracle Database is used to sort result sets by one or more columns, ensuring that the results are presented in a specific order—either ascending (ASC) or descending (DESC). To understand how this works with NULL values, we need to delve into the details of ordering them as well. Let's break it down step-by-step.

**Code:**

```sql
-- Example Table Creation
CREATE TABLE employees (
    id NUMBER,
    name VARCHAR2(50),
    salary NUMBER,
    department_id NUMBER
);

-- Insert Sample Data
INSERT INTO employees (id, name, salary, department_id) VALUES (1, 'John', 5000, 10);
INSERT INTO employees (id, name, salary, department_id) VALUES (2, 'Jane', NULL, 10);
INSERT INTO employees (id, name, salary, department_id) VALUES (3, 'Mike', 6000, 20);
```

**Data:**

```sql
-- Table: employees
| id | name   | salary | department_id |
|----|--------|--------|--------------|
| 1  | John   | 5000   | 10           |
| 2  | Jane   | NULL   | 10           |
| 3  | Mike   | 6000   | 20           |
```

**Flow:**

1. Ascending
2. Descending
3. FIRST
4. LAST
5. ORDER BY Clause
6. Order Direction?
7. NULLS LAST (Default)
8. NULLS LAST
9. NULL Position?
10. NULLS FIRST
11. NULLS LAST

