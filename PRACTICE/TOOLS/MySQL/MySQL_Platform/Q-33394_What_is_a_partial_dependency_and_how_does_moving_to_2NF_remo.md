---
type: "practice"
difficulty: "Intermediate"
---

# What is a partial dependency and how does moving to 2NF remove it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Constraint Definitions](<../../../../TOOLS/MySQL/MySQL_Platform/Constraint_Definitions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Partial dependencies occur in database design when non-key attributes are dependent on only part of the primary key, leading to data redundancy and update anomalies. Moving from First Normal Form (1NF) to Second Normal Form (2NF) involves eliminating partial dependencies by ensuring that all non-key columns depend fully on the entire primary key.

To understand this concept better, let's break it down step-by-step.

**Code:**

```
MERMAID
Copy
graph TD;
    A[First Normal Form] --> B(1. Data is organized into rows and columns);
    B --> C(2. Each row contains unique data items);
    C --> D{Is there a composite primary key?};
    D -- Yes --> E[No partial dependencies];
    D -- No --> F[Partial dependencies exist];
    F --> G[Identify non-key attributes depending on part of the PK];
    G --> H[Remove these dependencies by decomposing tables];
```

**Example input / output:**

```

Let’s break down the input and output for decomposition:

SQL
Copy
CREATE TABLE EmployeeDetails (
    EmployeeID INT,
    Name VARCHAR(50),
    Salary DECIMAL(10, 2)
);

CREATE TABLE DepartmentAssignments (
    EmployeeID INT PRIMARY KEY,
    DepartmentID INT
);

The EmployeeDetails table contains the non-key attributes that depend on the entire primary key. The DepartmentAssignments table captures the dependency of DepartmentID on EmployeeID.
```

**Data:**

```

Here’s an example to illustrate:

SQL
Copy
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    DepartmentID INT,
    Name VARCHAR(50),
    Salary DECIMAL(10, 2)
);

In this table, EmployeeID is the primary key. However, both DepartmentID, Name, and Salary are dependent on only a part of EmployeeID. Specifically:

DepartmentID: This attribute depends on the entire EmployeeID.
Name: This also depends on the entire EmployeeID.
Salary: This too is dependent solely on EmployeeID.

To move to 2NF, we need to decompose this table.
```

