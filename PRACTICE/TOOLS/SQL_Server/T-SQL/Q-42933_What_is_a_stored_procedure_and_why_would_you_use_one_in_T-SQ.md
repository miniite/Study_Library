---
type: "practice"
difficulty: "Basic"
---

# What is a stored procedure and why would you use one in T-SQL?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ T-SQL Basics](<../../../../TOOLS/SQL_Server/T-SQL/T-SQL_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

A stored procedure in T-SQL is a precompiled set of SQL statements that can be executed by name, making it easier to reuse code and manage database operations. It helps improve performance by reducing network traffic and simplifies security management.

#### Triaging the Question

The question asks you to explain what a stored procedure (SP) is in the context of T-SQL and why one might use an SP. This involves understanding the basic concept of stored procedures, their benefits, and when they are appropriate to use.

Step-by-Step Solution:
1. What is a Stored Procedure?

A stored procedure (SP) in T-SQL is a set of Transact-SQL statements and expressions that can be compiled once and executed multiple times with different input parameters. Essentially, it's like having a script or a function stored within the database itself.

2. Key Components:
Precompiled Code: The SP code is compiled when it’s created and stored in the database.
Parameterized Input: It can accept input parameters to perform dynamic operations.
Multiple Executions: Can be executed multiple times with different sets of data or configurations.
3. Use Cases for Stored Procedures:
Code Reusability: You can call an SP multiple times without rewriting the same code, making maintenance easier.
Security: Since SPs are stored in the database, you can control who has access to them and what they can do with them.
Performance Optimization: By precompiling the SQL statements, the database engine can optimize execution plans, reducing network traffic and improving performance.
4. Example Use Case:

Imagine a scenario where multiple applications need to perform the same query on a table but with different conditions. Instead of writing the exact same query in each application, you can define an SP that accepts parameters and is reused across all applications.

SQL
CREATE PROCEDURE GetCustomerOrders
    @CustomerId INT
AS
BEGIN
    SELECT * FROM Orders WHERE CustomerId = @CustomerId;
END;

You can then call this procedure from different applications with varying @CustomerId values.

5. Benefits:
Reduced Network Traffic: Since the SP is precompiled, less data is transferred over the network.

#### Cons
istency: Ensures that the same logic is applied consistently across all executions.
Security: Allows fine-grained access control, reducing the risk of unauthorized operations on sensitive data.
6. Common Mistakes and Follow-Ups:
Overusing SPs for Simple Operations: While useful, they should not be overused for simple queries that can be executed directly in SQL.
Not Considering Maintenance: As your application evolves, you may need to update the SP, which requires downtime or careful versioning.

#### Closing Summary
and Common Mistakes:

In summary, stored procedures are a powerful tool in T-SQL that enhance code reusability, security, and performance. They should be used judiciously based on their benefits and when they solve specific problems.

#### Common Mistakes

Overusing SPs for simple queries.
Neglecting maintenance of the SPs as your application evolves.
Trialing a Scenario:

Let’s consider an example where you have a reporting tool that needs to run various queries against customer data. Instead of writing different SQL scripts in the application, you could create multiple SPs with parameterized inputs and call them based on user preferences.

**Comparison:**

| OrderId | CustomerId | OrderDate       | TotalAmount |
|---------|------------|-----------------|-------------|
| 1       | 12345      | 2023-09-15      | 150.00      |
| 2       | 12345      | 2023-09-16      | 200.00      |

**Code:**

```sql
-- Example of creating a stored procedure
CREATE PROCEDURE GetCustomerOrders
    @CustomerId INT
AS
BEGIN
    SELECT * FROM Orders WHERE CustomerId = @CustomerId;
END;

-- Calling the SP from SQL Server Management Studio or another tool
EXEC GetCustomerOrders 12345;
```

**Example input / output:**

```sql
-- Executing the SP with sample data
EXEC GetCustomerOrders 12345;
OrderId	CustomerId	OrderDate	TotalAmount
1	12345	2023-09-15	150.00
2	12345	2023-09-16	200.00
```

**Data:**

```sql
-- Sample data schema for Orders table
CREATE TABLE Orders (
    OrderId INT,
    CustomerId INT,
    OrderDate DATE,
    TotalAmount DECIMAL(10, 2)
);
INSERT INTO Orders (OrderId, CustomerId, OrderDate, TotalAmount) VALUES
(1, 12345, '2023-09-15', 150.00),
(2, 12345, '2023-09-16', 200.00);
```

**Flow:**

1. Start
2. Define SP
3. Execute SP with parameters
4. Fetch results
5. End

