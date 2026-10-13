---
type: "practice"
difficulty: "Deep"
---

# How do you decide constraint placement — DB-enforced vs application-enforced vs both — for a system with multiple writer services? What's your rule of thumb?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Check and NOT NULL Constraints](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Check_and_NOT_NULL_Constraints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between database-enforced, application-enforced, or a combination of both constraints and keys in a distributed system with multiple writer services, the goal is to balance consistency, performance, and maintainability. A good rule of thumb involves understanding the nature of your data and how it will be accessed, as well as the trade-offs associated with different enforcement strategies.

**Code:**

```sql
-- Example schema for a simple order management system
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50)
);

CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT,
    OrderDate DATE,
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);
```

**Example input / output:**

```text
Input:
Customers table: CustomerID = 1 (John Doe) and CustomerID = 2 (Jane Smith)
Orders table: OrderID = 101, CustomerID = 1; OrderID = 102, CustomerID = 2

Output:
Valid Orders due to proper FK constraints
```

**Data:**

```json
{
  "Customers": [
    {"CustomerID": 1, "FirstName": "John", "LastName": "Doe"},
    {"CustomerID": 2, "FirstName": "Jane", "LastName": "Smith"}
  ],
  "Orders": [
    {"OrderID": 101, "CustomerID": 1, "OrderDate": "2023-06-01"},
    {"OrderID": 102, "CustomerID": 2, "OrderDate": "2023-06-05"}
  ]
}
```

