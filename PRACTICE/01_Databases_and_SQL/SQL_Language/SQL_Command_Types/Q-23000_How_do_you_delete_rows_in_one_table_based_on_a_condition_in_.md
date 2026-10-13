---
type: "practice"
difficulty: "Intermediate"
---

# How do you delete rows in one table based on a condition in another table?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ DROP and CASCADE](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/DROP_and_CASCADE.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the task of deleting rows in one table based on a condition in another table, we need to understand the relationship between these tables and ensure that our SQL query is both effective and safe. This involves using a DELETE statement with appropriate join conditions to target the correct rows.

#### Triaging the Question

The question essentially asks how to delete records from Table A where a specific condition exists in Table B. The key aspects being probed are:

Understanding Joins: How to use joins between two tables.
Conditional Deletion: Applying filters or conditions during deletion.
Transaction Management: Ensuring data integrity through proper handling of transactions.

#### First-Principles Solution

To delete rows in Table A based on a condition from Table B, we can leverage an INNER JOIN to match records and apply the necessary condition. Here’s the step-by-step process:

Identify Tables and Conditions: Determine which tables are involved (Table A and Table B) and the specific condition that needs to be checked in Table B.
Write a Join Query: Use an INNER JOIN to find matching records between the two tables based on the key fields.
Apply Conditional Filter: Add a WHERE clause to apply the condition from Table B.
Use DELETE Statement with JOIN: Combine the join and filter in a single DELETE statement.

#### Example
SQL Query

Suppose we have two tables, Orders (Table A) and Customers (Table B), where we want to delete orders that belong to customers who are inactive.

SQL
-- Sample Table Structures
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT,
    OrderDate DATE
);

CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    IsActive BIT
);

The SQL query would look like this:

SQL
DELETE FROM Orders
WHERE OrderID IN (
    SELECT OrderID
    FROM Orders AS O
    INNER JOIN Customers AS C ON O.CustomerID = C.CustomerID
    WHERE C.IsActive = 0
);

#### Closing Summary
and Common Mistakes/Follow-Ups

The query ensures that only orders associated with inactive customers are deleted. However, it's crucial to handle transactions properly to avoid accidental data loss:

Use Transactions: Wrap the delete operation in a transaction to roll back changes if something goes wrong.

SQL
BEGIN TRANSACTION;

DELETE FROM Orders
WHERE OrderID IN (
    SELECT OrderID
    FROM Orders AS O
    INNER JOIN Customers AS C ON O.CustomerID = C.CustomerID
    WHERE C.IsActive = 0
);

IF @@ERROR <> 0 ROLLBACK;
ELSE COMMIT;

Review the Result: Always run a SELECT statement first to review the records that would be deleted.

SQL
SELECT OrderID
FROM Orders AS O
INNER JOIN Customers AS C ON O.CustomerID = C.CustomerID
WHERE C.IsActive = 0;

Check for Nulls or Inconsistencies: Ensure there are no null values in the join keys that might cause unexpected behavior.

#### Performance Considerations
For large tables, ensure indexes on CustomerID and OrderID to optimize performance.

#### Common Mistakes
Incorrect Join Condition: Using an incorrect join condition can lead to deleting unintended records.
No Transaction Handling: Failing to use transactions can result in data loss if the operation fails mid-execution.
Ignoring Dry Runs: Not reviewing the affected rows before executing a delete statement can cause accidental deletion of important data.

**Comparison:**

| OrderID  |
|        102|

**Code:**

```

-- Example Tables and Query CREATE TABLE Orders ( OrderID INT PRIMARY KEY, CustomerID INT, OrderDate DATE );

CREATE TABLE Customers ( CustomerID INT PRIMARY KEY, IsActive BIT );

CODE
Copy

DELETE statement:
```sql
DELETE FROM Orders
WHERE OrderID IN (
    SELECT OrderID
    FROM Orders AS O
    INNER JOIN Customers AS C ON O.CustomerID = C.CustomerID
    WHERE C.IsActive = 0
);
```

**Example input / output:**

```text
-- Example of a SELECT statement to preview records before deletion

SELECT OrderID FROM Orders AS O INNER JOIN Customers AS C ON O.CustomerID = C.CustomerID WHERE C.IsActive = 0;

-- Output:
+----------+
| OrderID  |
+----------+
|        102|
+----------+

-- Running the DELETE Statement

DELETE FROM Orders
WHERE OrderID IN (
    SELECT OrderID
    FROM Orders AS O
    INNER JOIN Customers AS C ON O.CustomerID = C.CustomerID
    WHERE C.IsActive = 0
);
```

**Data:**

```sql
-- Sample Data for Tables

INSERT INTO Customers (CustomerID, IsActive) VALUES (1, 1), (2, 0), (3, 1);

INSERT INTO Orders (OrderID, CustomerID, OrderDate) VALUES
(101, 1, '2023-01-01'),
(102, 2, '2023-01-02'),
(103, 3, '2023-01-03');
```

**Flow:**

1. Yes
2. Start
3. Is there a need to deleterecords?
4. Identify Tables and Conditions
5. Write Join Query
6. Apply Conditional Filter
7. Use DELETE Statement withJOIN
8. Wrap in Transaction for Safety
9. Review Records with SELECT
10. Execute DELETE Statement
11. End

