---
type: "practice"
difficulty: "Intermediate"
---

# How do you create a stored procedure with input parameters in SQL Server?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ MERGE and Upserts](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/MERGE_and_Upserts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To create a stored procedure with input parameters in SQL Server, we first need to understand what a stored procedure is and its benefits. A stored procedure is a set of Transact-SQL statements that are compiled and saved on the database server. It can accept one or more parameters as input, perform operations such as data manipulation, and return output. Using stored procedures improves performance by reducing network traffic and allows for better code reuse.

In this scenario, we will create a simple stored procedure in SQL Server to retrieve customer information based on an ID. The stored procedure will take an int parameter representing the customer ID.

**Comparison:**

```

No tables are needed for this example as we are dealing with a simple stored procedure creation.
```

**Code:**

```sql
CREATE PROCEDURE GetCustomerById
    @customerId INT
AS
BEGIN
    SELECT * FROM Customers
    WHERE CustomerID = @customerId;
END;
```

**Example input / output:**

```

To test the stored procedure, you can use a query like:

SQL
Copy
EXEC GetCustomerById 1;

This will return the customer information where CustomerID is 1.
```

**Data:**

```text
No additional data blocks are required for this example as we are focusing on creating the stored procedure.
```

**Flow:**

1. No flowchart is necessary for creating a stored procedure with parameters in SQL Server, but you could represent it with an ASCII diagram if needed:
2. +-----------------+
3. | CREATE PROCEDURE|
4. | GetCustomerById |
5. | @customerId INT  |
6. +-----------------+
7. | BEGIN           |
8. | SELECT * FROM   |
9. | Customers       |
10. | WHERE CustomerID= |
11. | @customerId;    |
12. +-----------------+
13. | END             |
14. +-----------------+

