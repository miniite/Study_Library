---
type: "practice"
difficulty: "Basic"
---

# What does the NOT NULL constraint do and where is it applied?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Check and NOT NULL Constraints](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Check_and_NOT_NULL_Constraints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The NOT NULL constraint in database development ensures that a column must contain a value; it cannot be left empty or null. This concept is crucial to enforce data integrity and ensure that all records have valid information.

#### Triaging the Question

Core of the Question: The interviewer is probing your understanding of what NOT NULL constraints are, their importance in database design, and where they can be applied.

Key Assumptions:

You should understand basic SQL concepts.
You know the difference between primary keys (PK) and foreign keys (FK).
You recognize that NOT NULL is a type of constraint used to enforce data rules.

#### First-Principles Solution

Definition: A NOT NULL constraint specifies that a column cannot have a null value. This means every row inserted or updated must provide a non-null value for this column.

Application Scope:

Can be applied at the table level to columns where you want to ensure no null values are allowed.
Commonly used on columns like name, email, and other critical fields that cannot be empty.

#### Implementation
in SQL:

SQL
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    Email VARCHAR(100) NOT NULL
);

Impact on Data Integrity:

Ensures data consistency and reduces the risk of null values causing errors in applications.
Forces developers to provide values when inserting or updating records, which can lead to more robust application designs.

Enforcement Mechanism: Databases enforce NOT NULL constraints at runtime. If an attempt is made to insert a row with a null value for a column marked as NOT NULL, the database will raise an error and prevent the insertion.

#### Cons
iderations:

While useful, NOT NULL should be applied judiciously. Overusing it can sometimes lead to data redundancy or overly restrictive data entry rules.
Complement with other constraints like unique keys, primary keys, and foreign keys for comprehensive data integrity management.

#### Closing Summary

In summary, the NOT NULL constraint is a fundamental tool in database design used to ensure that columns must contain valid values. It is applied at both the column and table level to enforce strict data rules and maintain consistent, reliable datasets.

#### Common Mistakes/Follow-Ups
Misapplication: Applying NOT NULL constraints excessively can lead to unnecessary complexity or violate business logic.
Understanding Nullable Columns: Ensuring you understand what other columns might need NULL values is crucial. For instance, a status column in a customer table might be nullable if a customer record could exist without an active status yet.
Triaging Common Questions
How does NOT NULL interact with default values?
Can NOT NULL be combined with other constraints like UNIQUE or CHECK?

**Code:**

```sql
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    Email VARCHAR(100) NOT NULL
);
```

