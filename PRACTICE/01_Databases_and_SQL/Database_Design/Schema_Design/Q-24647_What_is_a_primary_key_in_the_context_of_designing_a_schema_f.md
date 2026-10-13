---
type: "practice"
difficulty: "Basic"
---

# What is a primary key in the context of designing a schema from an ER diagram?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Surrogate vs Natural Keys for Lookups](<../../../../01_Databases_and_SQL/Database_Design/Schema_Design/Surrogate_vs_Natural_Keys_for_Lookups.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database design, particularly when working with Entity-Relationship (ER) diagrams and schemas, understanding the concept of a primary key is crucial. A primary key uniquely identifies each record in a table and ensures data integrity by ensuring that no two records have the same value for this key. It's often visualized as an anchor point in ER diagrams, anchoring the schema structure.

#### Triaging the Question

The question asks you to explain what a primary key is in the context of designing a schema from an ER diagram. The interviewer might be probing your understanding of fundamental database design principles and how these principles are represented visually in ER diagrams.

#### First-Principles Solution
Definition: A primary key is a field or combination of fields that uniquely identifies each record (or row) in a table.
Uniqueness Constraint: Each value in the primary key must be unique; no two records can have the same primary key value.
Non-null Constraint: The primary key cannot contain null values, ensuring every record is distinct.
Foreign Key Relationship: Primary keys are often used to establish relationships between tables (e.g., through foreign keys).

#### Example Scenario

Let's consider an example ER diagram for a simple e-commerce system where we have two entities: Customer and Order. The schema design might look like this:

Customer (ID, Name, Email)
ID is the primary key.
Order (OrderID, CustomerID, ProductID, Quantity)
OrderID could be the primary key.
CustomerID would likely reference the Customer table as a foreign key.
Detailed Explanation
Uniqueness and Non-null: In the Customer entity, each customer must have a unique ID. This ensures that there are no duplicate customers in the system. The CustomerID field in the Order entity references this primary key to link orders to specific customers.
Entity-Relationship Diagram (ERD): In an ER diagram, a primary key is typically represented with a small circle or box around it. This visual cue helps quickly identify the unique identifier for each entity.

#### Common Mistakes
and Follow-Ups
Using Non-Unique Values: Failing to ensure that all values in the primary key are unique can lead to data integrity issues.
Allowing Nulls: Allowing null values in a primary key violates its fundamental rule, potentially causing confusion during data retrieval or analysis.

#### Closing Summary

In summary, a primary key is essential for maintaining data integrity and ensuring uniqueness within a database table. When designing schemas from ER diagrams, it's crucial to identify the primary key(s) correctly and understand their role in establishing relationships between tables.

**Code:**

```sql
-- Example SQL schema with primary keys
CREATE TABLE Customer (
    ID INT PRIMARY KEY,
    Name VARCHAR(100),
    Email VARCHAR(100)
);

CREATE TABLE Order (
    OrderID INT PRIMARY KEY,
    CustomerID INT,
    ProductID INT,
    Quantity INT,
    FOREIGN KEY (CustomerID) REFERENCES Customer(ID)
);
```

**Data:**

```plaintext
Customer Entity: ID, Name, Email
Order Entity: OrderID, CustomerID, ProductID, Quantity
```

