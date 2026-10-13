---
type: "practice"
difficulty: "Intermediate"
---

# What is a transitive dependency and which normal form removes it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Normal Forms](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Normal_Forms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Transitive dependencies occur in database schemas where a non-prime attribute (a field that is not part of any key) depends on another non-prime attribute, rather than directly on a primary or candidate key. In simpler terms, if we have a table with three attributes A, B, and C such that A → B and B → C, then C is transitively dependent on A via B. The Normal Form (NF) that removes transitive dependencies is Third Normal Form (3NF).

To properly address the question, let's break down what each term means:

Transitive Dependency: Imagine a scenario where if A determines B, and B in turn determines C, then C depends transitively on A.
Third Normal Form (3NF): This form ensures that every non-prime attribute is fully functionally dependent on the primary key. In other words, any transitive dependencies should be removed.

Now let's reason through this step-by-step:

Understanding the Problem: We need to identify situations where a non-prime field depends indirectly on another non-prime field.
Identifying Transitive Dependencies: Consider a simple example: in an Employee table, we have columns like EmployeeID, DepartmentID, and Salary. If Salary is dependent on DepartmentID, but not directly on EmployeeID, it might mean that DepartmentID indirectly influences the salary. To eliminate this dependency, we should ensure that each non-key attribute (like Salary) depends only on a key (like EmployeeID).
Removing Transitive Dependencies: To achieve 3NF, we need to normalize the schema such that every non-prime attribute is fully dependent on some candidate key(s). This often involves breaking down tables into smaller ones where each table has its own primary key and avoids transitive dependencies.

#### Example
of Transition from 2NF to 3NF: Consider a table Employee with columns EmployeeID, DepartmentID, and Salary. If there is a dependency such that DepartmentID → Salary, then the schema needs to be decomposed so that each non-prime attribute (like Salary) directly depends on the primary key.

Here’s an example of how this might look in practice:

SQL
-- Original Table with 2NF issues
CREATE TABLE Employee (
    EmployeeID INT PRIMARY KEY,
    DepartmentID INT,
    Salary DECIMAL(10, 2)
);

-- After decomposing to achieve 3NF
CREATE TABLE Employee (
    EmployeeID INT PRIMARY KEY,
    DepartmentID INT,
    -- Other attributes...
);

CREATE TABLE Department (
    DepartmentID INT PRIMARY KEY,
    Salary DECIMAL(10, 2),
    -- Other attributes...
);

In this schema:

Employee table now has a primary key (EmployeeID) and no transitive dependencies.
Department table ensures that every non-prime attribute (like Salary) is directly dependent on the primary key.

**Code:**

```sql
CREATE TABLE Employee (
    EmployeeID INT PRIMARY KEY,
    DepartmentID INT,
    -- Other attributes...
);

CREATE TABLE Department (
    DepartmentID INT PRIMARY KEY,
    Salary DECIMAL(10, 2),
    -- Other attributes...
);
```

