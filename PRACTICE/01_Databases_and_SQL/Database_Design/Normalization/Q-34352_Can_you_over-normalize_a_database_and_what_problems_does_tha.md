---
type: "practice"
difficulty: "Intermediate"
---

# Can you over-normalize a database and what problems does that cause?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Normalization Fundamentals](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Normalization_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Over-normalizing a database can lead to excessive complexity, decreased query performance, and potential data inconsistencies. The core idea is to find the right balance between normalization to avoid redundancy and denormalization to optimize read operations.

The interviewer is probing your understanding of normalization levels (1NF, 2NF, 3NF), their benefits, and when they might be too fine-grained. You need to explain how over-normalizing can introduce unnecessary joins, reduce data retrieval efficiency, and increase the complexity of maintaining your database schema.

**Code:**

```sql
-- Example of a normalized table with over-normalization issues
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50)
);

CREATE TABLE Departments (
    DepartmentID INT PRIMARY KEY,
    DepartmentName VARCHAR(100)
);

CREATE TABLE EmployeeDepartments (
    EmployeeID INT,
    DepartmentID INT,
    StartDate DATE,
    EndDate DATE,
    PRIMARY KEY (EmployeeID, DepartmentID),
    FOREIGN KEY (EmployeeID) REFERENCES Employees(EmployeeID),
    FOREIGN KEY (DepartmentID) REFERENCES Departments(DepartmentID)
);
```

**Example input / output:**

| Employee | Start Date   | Department |
|----------|--------------|------------|
| John Doe | Jan 1, 2023  | Engineering|
| John Doe | May 1, 2023  | Marketing  |
| Jane Smith| Feb 1, 2023 | Engineering|

**Data:**

```json
{
  "employees": [
    {"employeeId": 1, "firstName": "John", "lastName": "Doe"},
    {"employeeId": 2, "firstName": "Jane", "lastName": "Smith"}
  ],
  "departments": [
    {"departmentId": 100, "departmentName": "Engineering"},
    {"departmentId": 200, "departmentName": "Marketing"}
  ],
  "employeeDepartments": [
    {"employeeId": 1, "departmentId": 100, "startDate": "2023-01-01", "endDate": null},
    {"employeeId": 1, "departmentId": 200, "startDate": "2023-05-01", "endDate": null},
    {"employeeId": 2, "departmentId": 100, "startDate": "2023-02-01", "endDate": null}
  ]
}
```

