---
type: "practice"
difficulty: "Intermediate"
---

# How would you decide the placement of a nullable versus non-nullable foreign key when a relationship is optional on only one side, and how does that affect indexing?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Surrogate vs Natural Keys for Lookups](<../../../../01_Databases_and_SQL/Database_Design/Schema_Design/Surrogate_vs_Natural_Keys_for_Lookups.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing an ER diagram or schema, deciding whether to use a nullable or non-nullable foreign key in a relationship where the relationship is optional on only one side can significantly impact data integrity and performance. This decision involves balancing between enforcing referential integrity and optimizing query performance. Here’s how you would approach it:

TL;DR
Nullable Foreign Key: Use when the relationship is optional, meaning the referenced row may not always exist in the related table.
Non-Nullable Foreign Key: Use when the relationship must always be enforced and there should never be a case where the reference does not exist.

#### Triage
What’s Being Probed

The core of this question revolves around understanding the implications of nullability on foreign keys, specifically in scenarios where relationships are optional. The interviewer is probing your knowledge of:

Referential Integrity: Understanding how foreign keys maintain consistency between tables.
Null Values and Constraints: Recognizing the impact of nullable vs non-nullable constraints.

#### Performance Considerations
Evaluating indexing strategies.

#### First-Principles Solution

Understand the Relationship:

Determine if the relationship is optional on one side only. For example, in a Customer to Order scenario, an order might be optional for a customer (i.e., a customer may not have placed any orders yet).

Nullable Foreign Key Decision:

If the relationship can be optional, use a nullable foreign key.
SQL
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT NULL REFERENCES Customers(CustomerID),
    -- Other columns
);
This allows for flexibility; not every customer will have an order, and this is acceptable because the relationship is optional.

Non-Nullable Foreign Key Decision:

If the relationship must always be enforced (e.g., a user must belong to a specific role in a system), use a non-nullable foreign key.
SQL
CREATE TABLE Users (
    UserID INT PRIMARY KEY,
    RoleID INT NOT NULL REFERENCES Roles(RoleID),
    -- Other columns
);
This ensures that every user has a valid role associated with them, maintaining strong referential integrity.

Indexing Considerations:

Nullable Foreign Keys: A nullable foreign key can still be indexed to improve query performance.
SQL
CREATE INDEX idx_customer_orders ON Orders (CustomerID);
However, null values might affect the effectiveness of the index, especially if they are common or if there is a high cardinality.

Implications on Query Performance:

Queries involving nullable foreign keys must handle NULL values properly.
SQL
SELECT * FROM Orders WHERE CustomerID = 123; -- Returns rows with and without orders for the customer.
Non-nullable foreign keys simplify queries but may require additional handling to ensure data integrity.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Common Mistake: Assuming that a non-nullable foreign key must always be enforced, leading to unnecessary constraints in optional relationships.

#### Follow-Up
Question: How would you handle cascading updates or deletions when using nullable and non-nullable foreign keys? This helps assess your understanding of referential integrity rules.
Depth + Language Match the Level

Given an intermediate experience level, this approach balances simplicity with essential considerations. Understanding the nuances between nullable and non-nullable foreign keys is crucial for designing efficient and robust database schemas.

**Comparison:**

```
Customers	Orders	Roles	Users
CustomerID	Name	OrderID	UserID
1	John Doe	101	2
```

**Code:**

```sql
-- Nullable Foreign Key Example
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT NULL REFERENCES Customers(CustomerID),
    -- Other columns
);

-- Non-Nullable Foreign Key Example
CREATE TABLE Users (
    UserID INT PRIMARY KEY,
    RoleID INT NOT NULL REFERENCES Roles(RoleID),
    -- Other columns
);
```

**Example input / output:**

```text
-- Nullable Foreign Key Scenario
INSERT INTO Orders (OrderID, CustomerID) VALUES (101, 1);

-- Non-Nullable Foreign Key Scenario
INSERT INTO Users (UserID, RoleID) VALUES (2, 1);
```

**Data:**

```yaml
Customers:
  - CustomerID: 1
  - Name: John Doe

Orders:
  - OrderID: 101
  - CustomerID: 1

Roles:
  - RoleID: 1
  - Name: Admin

Users:
  - UserID: 2
  - RoleID: 1
```

**Flow:**

1. Customer
2. Nullable FK to Orders
3. User
4. Non-Nullable FK to Roles

