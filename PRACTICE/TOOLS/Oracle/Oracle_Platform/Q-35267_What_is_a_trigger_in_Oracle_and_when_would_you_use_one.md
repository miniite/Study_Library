---
type: "practice"
difficulty: "Basic"
---

# What is a trigger in Oracle and when would you use one?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ PL/SQL Overview](<../../../../TOOLS/Oracle/Oracle_Platform/PLSQL_Overview.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Triggers in Oracle are database objects that automatically execute a stored procedure (or PL/SQL) whenever certain events occur on a specific table or view. The primary purpose of using triggers is to enforce complex business rules, maintain data integrity, or perform actions before or after data modifications.

#### Triage

The question asks for an overview of what triggers are in Oracle and when they should be used. We need to explain the basic concept, their main benefits, common use cases, and potential pitfalls.

#### First-Principles Solution
Definition:

A trigger is a stored procedure in PL/SQL that gets automatically executed by the database when certain events occur on a table or view. These events include insert, update, delete, or select operations.

When to Use Triggers:
Enforcing Business Rules:
To ensure data integrity and consistency, especially when multiple related tables are involved.
Maintaining Data Integrity:
Automatically updating other tables based on changes in a primary table.
Logging Activities:
Recording activities in separate logging tables for auditing purposes.
Preventing Invalid Operations:
Disallowing certain operations that would violate defined rules (e.g., preventing deletion of records with active orders).

#### Example Scenario

Imagine you have two tables, Orders and OrderDetails. You want to ensure that when an order is deleted from the Orders table, all corresponding details in the OrderDetails table are also removed. This can be achieved using a trigger.

Step-by-Step Solution
Define the Trigger:
SQL
CREATE OR REPLACE TRIGGER delete_order_details_trigger
BEFORE DELETE ON Orders
FOR EACH ROW
BEGIN
    DELETE FROM OrderDetails WHERE order_id = :OLD.order_id;
END;
Explanation of Key Components:
BEFORE DELETE: The trigger fires before the delete operation.
ON Orders: The table on which the event occurs.
FOR EACH ROW: Ensures that each row deletion is handled individually.
:OLD.order_id: Represents the value of order_id in the deleted row.

#### Common Mistakes
and Follow-Ups
Overuse: Triggers can lead to complex interactions and performance issues if not managed carefully. They should be used sparingly, especially for simple operations that can be handled by application logic.
Performance Impact: Triggers can slow down database operations, particularly large delete or update operations, as they execute additional code.
Debugging Challenges: Debugging trigger-based issues can be tricky since the logic is spread across multiple places (the main query and the trigger).
Maintenance Overhead: Managing triggers over time can become cumbersome due to their complexity and interdependencies.

#### Closing Summary

Triggers in Oracle provide a powerful mechanism for enforcing complex business rules, maintaining data integrity, and logging activities. However, they should be used judiciously to avoid performance bottlenecks and maintain manageable database architecture.

