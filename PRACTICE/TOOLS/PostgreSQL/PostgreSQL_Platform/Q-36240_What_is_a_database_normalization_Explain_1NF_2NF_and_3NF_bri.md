---
type: "practice"
difficulty: "Intermediate"
---

# What is a database normalization? Explain 1NF, 2NF, and 3NF briefly.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Materialized CTEs](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Materialized_CTEs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Database normalization is a process of organizing data in a database to minimize redundancy and dependency. It involves structuring tables in such a way that they are both efficient and error-free. For an intermediate-level understanding, we'll focus on the first three normal forms (1NF, 2NF, and 3NF).

First Normal Form (1NF) is about ensuring that each table contains atomic values—no repeating groups or arrays. Second Normal Form (2NF) builds on this by requiring that all non-key columns are fully dependent on the primary key. Third Normal Form (3NF) takes it a step further, eliminating transitive dependencies.

Let's dive into these in more detail.

**Comparison:**

```
1NF Table (example)	2NF Table (example)	3NF Table (example)
Table A	Table B	Table C
Id	Id	Id
Name	EmployeeId	Name
Age		Salary
Address	DepartmentId
	DeptName
Example Scenario

Consider a simple database for an employee management system.

1NF Table (Table A):
Id
Name
Age
Address

This table is not normalized because the Address field contains multiple pieces of information that should be separated.

2NF Table (Table B and C):
Employee (Table B) has Id, Name, Age.
Department (Table C) has Id, DeptName.

The Address is moved to a separate table, but there are still dependencies between the tables that need fixing.

3NF Table:
Employee now only contains Id, Name, and Age.
Department contains Id and DeptName.
Address has its own table with fields Id, Line1, City, State.

This ensures that all non-key columns are fully dependent on the primary key.
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[1NF] --> B[2NF];
    B --> C[3NF];
```

