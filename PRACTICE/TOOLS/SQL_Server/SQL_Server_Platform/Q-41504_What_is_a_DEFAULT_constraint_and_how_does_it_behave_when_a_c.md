---
type: "practice"
difficulty: "Basic"
---

# What is a DEFAULT constraint and how does it behave when a column is omitted in an INSERT?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ CTEs in SQL Server](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/CTEs_in_SQL_Server.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL Server, a DEFAULT constraint is used to provide a default value for a column if no value is provided during an INSERT operation. This concept ensures that even when a user omits specifying a column's value in the INSERT statement, it still has a valid value by falling back on the predefined default.

#### Triaging the Question

The question asks us to explain what a DEFAULT constraint is and how it behaves when a column is omitted during an INSERT. The core of this question tests your understanding of SQL Server's data constraints and how they handle missing values in database operations. We will assume that we are working with a typical SQL Server environment where DEFAULT constraints have been set up.

#### First-Principles Solution

Let's start from the basics:

Understanding Constraints: In relational databases, constraints like NOT NULL, UNIQUE, and DEFAULT ensure data integrity by imposing specific rules on column values.
Default Constraint: A DEFAULT constraint sets a default value for a column if no explicit value is provided during an insert operation.

Here's how it works in practice:

Suppose we have a table named Orders with columns: OrderID, CustomerID, and OrderDate.
We set up the OrderDate column to have a DEFAULT constraint that assigns the current date (e.g., GETDATE()).
SQL
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT NOT NULL,
    OrderDate DATETIME DEFAULT GETDATE()
);

In this scenario, if we execute an INSERT statement without specifying a value for OrderDate, SQL Server will automatically use the default value (current date).

#### Example

#### Cons
ider inserting data into the table:

SQL
INSERT INTO Orders (OrderID, CustomerID) VALUES (101, 2023);

The resulting row in the Orders table would look like this:

OrderID	CustomerID	OrderDate
101	2023	2023-10-05 14:30:00

Notice how OrderDate automatically got the default value (GETDATE()).

#### Closing Summary
and Common Mistakes

In summary, a DEFAULT constraint ensures that every row has a valid value for the column by setting an automatic fallback when no explicit value is provided. This helps maintain data integrity without requiring every user to specify all values during inserts.

#### Common mistakes
include:

Misunderstanding how default values work (e.g., thinking they only apply in certain contexts).
Forgetting to check the constraint definition and its associated function (GETDATE() or others).
Trialing the Concept

To further solidify your understanding, consider the following scenario: If you try to insert a row with all NULL values, SQL Server will use the default value for each column that has one defined.

SQL
INSERT INTO Orders (OrderID, CustomerID, OrderDate) VALUES (102, NULL, NULL);

The result would be:

OrderID	CustomerID	OrderDate
102	NULL	2023-10-05 14:30:00
Self-Check

This answer covers the key aspects of DEFAULT constraints and their behavior in SQL Server, ensuring correctness and completeness.

**Code:**

```sql
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT NOT NULL,
    OrderDate DATETIME DEFAULT GETDATE()
);

INSERT INTO Orders (OrderID, CustomerID) VALUES (101, 2023);
```

**Example input / output:**

```text
-- Input
INSERT INTO Orders (OrderID, CustomerID) VALUES (101, 2023);

-- Output
+----------+------------+-------------------+
| OrderID  | CustomerID | OrderDate         |
+----------+------------+-------------------+
| 101      | 2023       | 2023-10-05 14:30:00 |
+----------+------------+-------------------+
```

