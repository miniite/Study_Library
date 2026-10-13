---
type: "practice"
difficulty: "Intermediate"
---

# What does ON DELETE CASCADE do on a foreign key?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Referential Actions](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Referential_Actions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

ON DELETE CASCADE is a constraint in SQL that ensures referential integrity by automatically deleting related records in child tables when a record in the parent table is deleted. This mechanism helps maintain data consistency and avoids orphaned records, which could otherwise lead to inconsistent or invalid data.

In essence, ON DELETE CASCADE tells the database management system (DBMS) to automatically delete all rows from child tables that have foreign key references to the row being deleted from the parent table. Let's break this down in detail:

#### Triaging the Question

The interviewer is probing your understanding of:

Foreign Key Relationships: Understanding how foreign keys work and their relationship with primary keys.
Referential Integrity: The importance of maintaining consistent data across tables.
SQL Constraint Behavior: How constraints like ON DELETE CASCADE influence database operations.

#### Assumptions
You are familiar with the concepts of primary keys (PK) and foreign keys (FK).
You understand basic SQL syntax for defining constraints in a relational database.
Step-by-Step Solution from First Principles
Understanding Foreign Keys: A foreign key in one table refers to the primary key of another table, establishing a relationship between them.
Parent Table vs. Child Table: The parent table is the table that holds the primary key, while the child table contains the foreign key referencing it.
ON DELETE CASCADE Mechanism:
When you delete a record from the parent table, ON DELETE CASCADE ensures that any records in the child table that reference the deleted row are also automatically deleted.
Scenario Example:
Suppose we have two tables: Orders (parent) and OrderItems (child). Each order can have multiple items, so OrderItems has a foreign key referencing the OrderId column in the Orders table.
If you delete an Order, all corresponding OrderItems should be deleted to maintain data consistency.
Code Example
SQL
-- Defining a parent table with a primary key
CREATE TABLE Orders (
    OrderId INT PRIMARY KEY,
    CustomerId INT,
    TotalAmount DECIMAL(10, 2)
);

-- Defining a child table with a foreign key referencing the parent table
CREATE TABLE OrderItems (
    ItemId INT PRIMARY KEY,
    OrderId INT,
    Quantity INT,
    FOREIGN KEY (OrderId) REFERENCES Orders(OrderId) ON DELETE CASCADE
);

#### Common Mistakes
and Follow-Ups
Misunderstanding Cascade Operations: Confusing ON DELETE CASCADE with other cascade operations like SET NULL.

#### Performance Considerations
Ignoring the potential performance impact of cascading deletes, especially in large databases.
Data Integrity Issues: Failing to consider edge cases where data might become orphaned if not managed properly.

#### Closing Summary

Understanding and correctly implementing ON DELETE CASCADE is crucial for maintaining referential integrity in relational databases. It ensures that related records are automatically deleted when their parent record is removed, preventing data inconsistencies.

**Comparison:**

```
Table	Columns
Orders	OrderId, CustomerId, TotalAmount
OrderItems	ItemId, OrderId, Quantity
```

**Code:**

```sql
-- Example of creating tables with foreign key constraint
CREATE TABLE Orders (
    OrderId INT PRIMARY KEY,
    CustomerId INT,
    TotalAmount DECIMAL(10, 2)
);

CREATE TABLE OrderItems (
    ItemId INT PRIMARY KEY,
    OrderId INT,
    Quantity INT,
    FOREIGN KEY (OrderId) REFERENCES Orders(OrderId) ON DELETE CASCADE
);
```

**Example input / output:**

```text
Input:
DELETE FROM Orders WHERE OrderId = 1;

Output:
- All related records in the OrderItems table with OrderId = 1 are automatically deleted.
```

**Data:**

```json
{
  "Orders": [
    { "OrderId": 1, "CustomerId": 101, "TotalAmount": 9.99 },
    { "OrderId": 2, "CustomerId": 102, "TotalAmount": 7.45 }
  ],
  "OrderItems": [
    { "ItemId": 101, "OrderId": 1, "Quantity": 2 },
    { "ItemId": 102, "OrderId": 1, "Quantity": 3 },
    { "ItemId": 103, "OrderId": 2, "Quantity": 1 }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Delete from Orders
4. Is there a foreign keyrelationship?
5. Delete related records in childtables automatically
6. Only delete the parent record

