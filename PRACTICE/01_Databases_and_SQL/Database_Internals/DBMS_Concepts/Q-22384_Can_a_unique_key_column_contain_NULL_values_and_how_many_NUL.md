---
type: "practice"
difficulty: "Intermediate"
---

# Can a unique key column contain NULL values and how many NULLs are allowed?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Normal Forms](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Normal_Forms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this database interview question, we're exploring the behavior of unique keys in a Database Management System (DBMS) when dealing with NULL values. The key points to consider include whether a unique key can contain NULL, and if so, how many such NULL entries are permissible.

#### Triage

The core of this question is about understanding the rules around NULL values within unique keys in a DBMS. The interviewer might be probing your knowledge on normalization, data integrity constraints, and possibly database design principles. We'll assume that the schema we're working with has a unique key constraint imposed on a column.

#### First-Principles Solution
Understanding Unique Keys: A unique key ensures that each value in the associated columns is distinct. It means no duplicate values are allowed.
NULL Values and Uniqueness:
In most SQL standards, including MySQL, PostgreSQL, and Oracle, a NULL value is considered unique because it is not equal to any other value, including itself.
However, when we impose a uniqueness constraint on a column that can contain NULL, the DBMS treats each NULL as distinct. This means you can have multiple NULL entries in such a column.
Detailed Explanation

Behavior of NULLs with Uniqueness:

#### Cons
ider a table Employees with a unique key on the column EmployeeID.
SQL
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50)
);

If you insert multiple rows where EmployeeID is NULL, they will be allowed because each NULL value is treated as distinct.

Normalization and Data Integrity:

Normalization rules, such as the Third Normal Form (3NF), typically advise against storing duplicate or unnecessary data.
Allowing many NULL values in a unique key column can violate normalization principles if the uniqueness should imply meaningful identity.

#### Common Mistakes
and Follow-Ups
Misunderstanding Uniqueness: Failing to recognize that NULL is treated as distinct can lead to incorrect assumptions about data integrity checks.
Normalization Violations: Not considering whether a unique key with multiple NULLs aligns with the business logic or normalization requirements.

#### Conclusion

A unique key column in a DBMS can indeed contain NULL values, and each NULL value is treated as distinct. Therefore, theoretically, you can have any number of NULL entries in such a column without violating the uniqueness constraint. However, it’s crucial to ensure that this design aligns with business requirements and normalization principles.

**Example input / output:**

```text
Inserted 3 rows with EmployeeID as NULL.
```

