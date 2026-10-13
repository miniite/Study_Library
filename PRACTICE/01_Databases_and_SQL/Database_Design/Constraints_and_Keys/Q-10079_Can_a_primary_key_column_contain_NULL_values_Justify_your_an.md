---
type: "practice"
difficulty: "Basic"
---

# Can a primary key column contain NULL values? Justify your answer.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Primary and Unique Keys](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Primary_and_Unique_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the question, we need to understand what primary keys are and their constraints in database design. A primary key is a unique identifier for each record in a table, ensuring that no two records have the same value in this column. The fundamental rule of a primary key is that it must contain unique values, but can it include NULLs? Let's break this down step by step.

Firstly, we need to define what constraints and keys are:

#### Cons
traints: These are rules imposed on columns or groups of columns to restrict the input values, ensuring data integrity.
Primary Key (PK): A primary key is a column that uniquely identifies each row in a table. It must be unique and cannot contain NULL values.

Now, let's dive into why a primary key column cannot contain NULL values:

Uniqueness Requirement: By definition, a primary key must ensure that the data it contains is unique for every record. If a primary key could contain NULL, then there would be no way to uniquely identify records because multiple rows could theoretically have NULL in the same primary key column.
Data Integrity: The absence of NULLs ensures strong data integrity since each row can be unambiguously identified.

Here is an example to illustrate this:

#### Cons
ider a table Employees with columns EmployeeID, FirstName, and LastName. If we define EmployeeID as the primary key, it must have no duplicate values. It cannot contain NULLs because having an employee without an ID would violate the uniqueness requirement of the primary key.

To further clarify:

A primary key column is designed to be a unique identifier, which inherently means each value in that column should be distinct and non-null.
If you need to store data that might sometimes lack an ID (e.g., a person who hasn't been assigned one yet), consider using a surrogate primary key instead. Surrogate keys are often auto-generated integers or UUIDs.

In conclusion, a primary key column cannot contain NULL values because doing so would violate the fundamental requirement of uniqueness and compromise data integrity.

**Comparison:**

| EmployeeID | FirstName | LastName |
|------------|-----------|----------|
| 1          | Alice     | Smith    |
| 2          | Bob       | Johnson  |

**Code:**

```sql
-- Example SQL Schema for Employees Table with an Auto-Increment Primary Key
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY AUTO_INCREMENT,
    FirstName VARCHAR(50),
    LastName VARCHAR(50)
);

-- Attempting to Insert a NULL into the primary key column would fail: INSERT INTO Employees (EmployeeID, FirstName, LastName) VALUES (NULL, 'John', 'Doe'); -- This will not work
```

**Example input / output:**

```text
-- Attempting to Insert a NULL into the primary key column in SQL
INSERT INTO Employees (EmployeeID, FirstName, LastName) VALUES (NULL, 'John', 'Doe'); -- This will fail with an error

-- Error Message:
ERROR 1062 (23000): Duplicate entry 'NULL' for key 'PRIMARY'
```

**Data:**

```json
{
  "employees": [
    {
      "EmployeeID": 1,
      "FirstName": "Alice",
      "LastName": "Smith"
    },
    {
      "EmployeeID": 2,
      "FirstName": "Bob",
      "LastName": "Johnson"
    }
  ]
}
```

**Flow:**

1. Define Primary Key
2. Can NULLs be used?
3. No, because it would violatethe uniqueness rule.
4. Yes, but only in specific cases(surrogate keys).
5. Ensure data integrity byavoiding NULLs in primary keycolumns.

