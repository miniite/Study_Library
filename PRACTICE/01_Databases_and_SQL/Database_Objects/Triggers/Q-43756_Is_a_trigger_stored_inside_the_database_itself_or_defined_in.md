---
type: "practice"
difficulty: "Basic"
---

# Is a trigger stored inside the database itself or defined in the application code, and why does that distinction matter?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Triggers Behavior Performance and Alternatives](<../../../../01_Databases_and_SQL/Database_Objects/Triggers/Triggers_Behavior_Performance_and_Alternatives.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In simple terms, triggers are special types of stored procedures that automatically execute when an event occurs on a specific table. They are defined inside the database itself rather than in application code. This placement is crucial because it ensures that the database enforces certain rules and performs actions without requiring external applications to be involved.

First, let's clarify what we mean by "triggers" and where they reside:

Triggers are pieces of code (stored procedures or functions) attached to a table in a relational database management system (RDBMS). They get executed automatically when specific events occur on the table, such as an insert, update, or delete operation.
These triggers can be stored directly within the RDBMS.

Now, let's dive into why storing triggers inside the database is beneficial:

#### Cons
istency and Reliability: When a trigger is defined in the database, it ensures that operations on the table are consistent. The same logic runs every time, regardless of whether multiple applications or external systems are involved.
Security: Storing business logic in the database can enhance security by keeping sensitive information within a controlled environment. If this logic were stored in application code, it could pose risks if the application is compromised.
Efficiency: Running triggers directly from the database can be more efficient because they avoid additional network latency and communication overhead that might occur when sending events to an external system for processing.

To illustrate with a simple example, consider a scenario where you want to ensure that whenever a new row is inserted into the orders table, its status should automatically be set to "pending". If this logic were written in application code, every application interacting with the database would need to perform additional steps. However, by defining a trigger on the orders table, this behavior can be handled seamlessly within the database.

SQL
CREATE TRIGGER orders_status_trigger
AFTER INSERT ON orders
FOR EACH ROW
BEGIN
    UPDATE orders SET status = 'pending' WHERE id = NEW.id;
END;

In this example, whenever a new row is inserted into the orders table, the trigger automatically sets its status to "pending".

The distinction between database-defined triggers and application-defined logic also matters in terms of maintenance and evolution:

Maintenance: If business rules change, modifying them within the database ensures consistency across all applications. However, it's crucial to have proper version control and documentation.
Evolving Logic: As requirements evolve, having well-designed triggers can make your codebase more robust and easier to maintain. Conversely, if too much logic is scattered in various application components, managing those changes can become cumbersome.

In conclusion, storing triggers inside the database promotes a cleaner separation of concerns, enhances performance and consistency, and simplifies maintenance. However, it's essential to weigh these benefits against potential downsides like increased complexity when dealing with distributed systems or specific use cases where external logic is required.

