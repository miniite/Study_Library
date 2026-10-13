---
type: "practice"
difficulty: "Basic"
---

# What is a view in a relational database and why is it called a virtual table?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Using Views Effectively](<../../../../01_Databases_and_SQL/Database_Objects/Views/Using_Views_Effectively.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A view in a relational database is essentially a stored query that presents data from one or more tables as if it were a separate table. It's often referred to as a "virtual table" because, unlike actual tables which store data persistently on disk, views are dynamically generated based on the underlying query when they are accessed.

#### Triage

The question is asking for an explanation of what a view is and why it’s called a virtual table. The interviewer is probing your understanding of how database objects work, specifically focusing on the concept of views versus actual tables in relational databases.

#### First-Principles Solution
Definition: A view is a result set defined by a query that exists as an independent object in the database.
Creation: It is created using SQL commands like CREATE VIEW and can be based on one or more base tables or other views.
Purpose: Views serve various purposes such as data abstraction, security, and simplifying complex queries.
Virtual Table: The term "virtual" refers to the fact that the view does not hold any data by itself; it merely defines a query that can retrieve the necessary information from one or more tables when needed.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

In summary, views provide an abstraction layer over the underlying schema and are useful for various reasons. They enhance security by hiding sensitive columns or rows of data, simplify complex queries, and abstract away table names and structures. A common mistake is confusing a view with a materialized view (which physically stores the result set), leading to potential performance issues if not managed properly.

When discussing views, you might be asked about their differences from materialized views, how they handle updates, or how they are used in conjunction with other database features like indexing and constraints.

**Comparison:**

```
Field	Type
EmployeeID	INT
FirstName	VARCHAR(50)
LastName	VARCHAR(50)
DepartmentName	VARCHAR(100)
```

**Code:**

```sql
-- Example of creating a view
CREATE VIEW EmployeeDetails AS
SELECT EmployeeID, FirstName, LastName, DepartmentName
FROM Employees E
JOIN Departments D ON E.DepartmentID = D.DepartmentID;
```

**Flow:**

1. User Query
2. DBMS
3. View Definition (SQL)
4. Base Table(s)
5. Result Set
6. User Result

