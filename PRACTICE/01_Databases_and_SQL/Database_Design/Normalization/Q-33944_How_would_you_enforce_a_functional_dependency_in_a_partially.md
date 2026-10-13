---
type: "practice"
difficulty: "Deep"
---

# How would you enforce a functional dependency in a partially denormalized schema where the database can't express it as a constraint?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Transitive Dependencies and 3NF](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Transitive_Dependencies_and_3NF.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce a functional dependency in a partially denormalized schema when your database can't express it as a constraint, you need to implement logic within application code or via database triggers. This approach involves ensuring that the data remains consistent and adheres to the specified functional dependencies despite the lack of support from the database schema.

#### Triaging the Question

The core issue is how to maintain data integrity when your database doesn't natively support functional dependencies as constraints. The interviewer is probing your understanding of normalization, denormalization trade-offs, and strategies for ensuring consistency in a partially denormalized design. Key assumptions include that you are working with a system where schema-level constraints are limited or non-existent.

#### First-Principles Solution
Identify the Functional Dependency: Clearly define which columns or sets of columns functionally determine others.
Implement Validation Logic:
Application-Level Validation: Ensure your application code validates data before it is stored in the database.
Database Triggers: Use database triggers to enforce constraints after data has been inserted, updated, or deleted.
Transaction Management: Design transactions that ensure consistency even if individual steps might violate the dependency temporarily.

#### Example
Implementation

Let's say you have a Customer and an associated Order entity where each customer can only place one order with a specific status at any given time (e.g., "Pending", "Shipped"). In a partially denormalized schema, this functional dependency cannot be expressed as a database constraint.

SQL
CREATE TABLE Customer (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE Order (
    id INT PRIMARY KEY,
    customer_id INT,
    status VARCHAR(50),
    FOREIGN KEY (customer_id) REFERENCES Customer(id)
);

To enforce that a Customer can only have one order with a specific status, you would:

Define the Functional Dependency: A Customer's active order should be uniquely identified by their ID and the current status.

Implement Application-Level Validation:

PYTHON
def place_order(customer_id, new_status):
    existing_order = get_active_order(customer_id)
    if existing_order and existing_order['status'] != new_status:
        raise ValueError("A customer can only have one order with a specific status at any time.")
    # Proceed to insert or update the order

Use Database Triggers (if supported by your DBMS):

SQL
CREATE TRIGGER ensure_unique_order ON Order
AFTER INSERT, UPDATE
FOR EACH ROW
BEGIN
    DECLARE @customer_id INT = NEW.customer_id;
    DECLARE @new_status VARCHAR(50) = NEW.status;

    IF EXISTS (
        SELECT 1 FROM Orders
        WHERE customer_id = @customer_id AND status != @new_status
    ) THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'A customer can only have one order with a specific status at any time';
    END IF;
END;

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing these strategies, you ensure that your application or database enforces the necessary functional dependencies even in a partially denormalized schema. Key mistakes to avoid include:

Failing to validate data on both the client and server sides.
Not considering transactional integrity when using triggers.

Common follow-ups might involve discussing performance implications of such validation mechanisms, especially in high-concurrency environments, or exploring more advanced techniques like materialized views for complex dependencies.

**Comparison:**

```
Customer
id
1
Order
id
1
```

**Code:**

```

def place_order(customer_id, new_status): existing_order = get_active_order(customer_id) if existing_order and existing_order['status'] != new_status: raise ValueError("A customer can only have one order with a specific status at any time.") # Proceed to insert or update the order

CODE
Copy
```sql
CREATE TRIGGER ensure_unique_order ON Order
AFTER INSERT, UPDATE
FOR EACH ROW
BEGIN
    DECLARE @customer_id INT = NEW.customer_id;
    DECLARE @new_status VARCHAR(50) = NEW.status;

    IF EXISTS (
        SELECT 1 FROM Orders
        WHERE customer_id = @customer_id AND status != @new_status
    ) THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'A customer can only have one order with a specific status at any time';
    END IF;
END;
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Active Order?
5. Throw Error
6. Proceed to Insert/UpdateOrder
7. End
8. Commit Transaction
9. End

