---
type: "practice"
difficulty: "Basic"
---

# Why should a foreign key generally reference a primary key or unique key of the other table?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Surrogate vs Natural Keys for Lookups](<../../../../01_Databases_and_SQL/Database_Design/Schema_Design/Surrogate_vs_Natural_Keys_for_Lookups.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a database, using a foreign key to reference another table's primary key or unique key is crucial for maintaining data integrity and ensuring that relationships between tables are well-defined. This practice helps in preventing orphaned records and ensures referential integrity.

#### Triaging the Question

The question is essentially asking about the importance of referencing primary keys (PK) or unique keys (UK) when establishing foreign keys (FK). The interviewer is probing your understanding of how database constraints work, particularly how they ensure data consistency and reduce potential errors during transactions. Assumptions include a basic understanding of the difference between PKs, UKs, and FKs.

#### First-Principles Solution
Primary Key: A primary key uniquely identifies each record in a table without any duplicates. It is essentially an index that helps maintain unique data.
Unique Key: A unique key also ensures uniqueness but can be null or have one null value, making it slightly more flexible than a PK.
Foreign Key: A foreign key references the primary key (or unique key) of another table, establishing a relationship between tables.

When you use a foreign key to reference a primary key (or unique key):

Referential Integrity: This ensures that every value in the foreign key column must match a value in the referenced primary key or unique key.
Data Consistency: It prevents orphaned records—records in one table with no corresponding record in another table, which can lead to logical inconsistencies.
Detailed Explanation

Primary Key as Reference:

A primary key uniquely identifies each row and ensures that every value is distinct. This means there are no duplicate entries.
When a foreign key references a primary key, it guarantees that the referenced record exists in the other table. If you try to insert a foreign key into a child table that doesn't exist in the parent table (i.e., its PK), the database will throw an error.

Unique Key as Reference:

A unique key also ensures uniqueness but can have one null value.
Using a unique key as a reference still helps maintain referential integrity, ensuring no duplicate references are created. However, it allows for flexibility in case of null values.

Why Not Use Any Other Column?:

If you use any other column (e.g., an ordinary non-primary or non-unique field) to create a foreign key, there's a risk that the referenced value might be duplicated or not unique, leading to referential integrity issues.
For example, if multiple rows in the parent table have the same value for this column, and you try to use it as a foreign key, it would violate the uniqueness requirement.

#### Closing Summary

In summary, using a primary key (or unique key) as a reference for a foreign key ensures that relationships between tables are well-defined and maintain referential integrity. This practice prevents orphaned records, avoids data inconsistencies, and helps in maintaining a clean and efficient database schema.

#### Common Mistakes/Follow-Ups
Mistake 1: Using an ordinary column to create a foreign key without ensuring its uniqueness.

#### Follow-Up
Question: "What are some scenarios where using a unique key instead of a primary key as a foreign key reference might be preferred?"
Answer: If the referenced field is allowed to have null values, or if there's a need for more flexibility in relationships.

**Comparison:**

| Employee | ID | Name         |
|----------|----|--------------|
| 1        | John         |
| 2        | Jane         |
| Department | ID | Name       |
|-------------|----|------------|
| 10          | HR  |
| 20          | Engineering|
| EmployeeDepartment | EmployeeID | DepartmentID |
|---------------------|------------|--------------|
| 1                   | 1          | 10           |

**Code:**

```sql
-- Example Schema with Foreign Key Referencing PK and UK

CREATE TABLE Employee (
    ID INT PRIMARY KEY,
    Name VARCHAR(100)
);

CREATE TABLE Department (
    ID INT PRIMARY KEY,
    Name VARCHAR(100)
);

CREATE TABLE EmployeeDepartment (
    EmployeeID INT,
    DepartmentID INT UNIQUE,  -- Using a unique key
    FOREIGN KEY (EmployeeID) REFERENCES Employee(ID),
    FOREIGN KEY (DepartmentID) REFERENCES Department(ID)
);
```

**Example input / output:**

```text
Inserting a new department:
INSERT INTO Department (ID, Name) VALUES (30, 'Marketing');

Inserting an employee-department relationship with valid primary key references:
INSERT INTO EmployeeDepartment (EmployeeID, DepartmentID) VALUES (1, 10);

Trying to insert invalid reference: will fail due to unique constraint and foreign key integrity check.
INSERT INTO EmployeeDepartment (EmployeeID, DepartmentID) VALUES (2, 35);  -- Error due to non-existent ID in Department table
```

**Data:**

```json
{
  "Employee": [
    {"ID": 1, "Name": "John"},
    {"ID": 2, "Name": "Jane"}
  ],
  "Department": [
    {"ID": 10, "Name": "HR"},
    {"ID": 20, "Name": "Engineering"}
  ]
}
```

**Flow:**

1. Employee
2. Foreign Key
3. Department
4. Primary/Unique Key

