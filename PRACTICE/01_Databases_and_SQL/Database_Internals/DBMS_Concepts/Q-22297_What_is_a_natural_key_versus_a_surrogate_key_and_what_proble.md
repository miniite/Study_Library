---
type: "practice"
difficulty: "Intermediate"
---

# What is a natural key versus a surrogate key, and what problems arise from using a natural key?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Composite vs Surrogate Keys](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Composite_vs_Surrogate_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with databases, understanding the difference between natural keys and surrogate keys is crucial for designing robust database schemas. A natural key uniquely identifies a record in a table based on meaningful data within that record, whereas a surrogate key is an artificial identifier assigned to each row. Using natural keys can lead to several issues, such as data duplication, difficulty in maintaining consistency, and increased complexity.

To fully explore this concept, we'll first define the terms, then discuss their implications, and finally outline common problems and best practices.

**Comparison:**

| EmployeeID | Name      |
|------------|-----------|
| 101        | John Doe  |
| 202        | Jane Smith|

**Code:**

```python
# Example of a simple table schema
class Employee:
    def __init__(self, employee_id, emp_name):
        self.employee_id = employee_id  # Surrogate key
        self.emp_name = emp_name  # Natural key

# Example usage
employee1 = Employee(101, "John Doe")
```

**Example input / output:**

```text
Consider an `Employees` table with the following data:

| EmployeeID | Name      |
|------------|-----------|
| 101        | John Doe  |
| 202        | Jane Smith|

Using a natural key, such as "Name," could lead to issues if multiple employees have the same name or if names change over time.
```

**Data:**

```yaml
# Sample data schema for Employees table
employees:
  - id: 101
    name: John Doe
  - id: 202
    name: Jane Smith
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Employee Table
6. Is Name Unique?
7. Use Natural Key
8. Do Names Change Over Time?
9. Consider Using Surrogate Key
10. Natural Key OK but Monitorfor Changes

