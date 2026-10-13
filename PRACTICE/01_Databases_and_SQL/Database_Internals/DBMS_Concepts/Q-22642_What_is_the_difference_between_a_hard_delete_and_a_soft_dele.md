---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between a hard delete and a soft delete when a source deletes a record, and how does a warehouse capture deletes?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Databases and Schemas](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Databases_and_Schemas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing database operations in data engineering, it's crucial to understand the concepts of hard deletes and soft deletes. Hard deletes permanently remove records from a database, whereas soft deletes mark records as deleted but keep them in the database with an additional flag or timestamp indicating their removal state. This distinction is particularly important when dealing with a source that performs deletions, especially in data warehousing scenarios where capturing historical data integrity is essential.

#### Triage

The question probes your understanding of different deletion mechanisms and how they impact data consistency and auditability, specifically focusing on the differences between hard deletes and soft deletes. You will need to explain these concepts while considering the implications for both the source system and the target data warehouse.

#### Assumptions
The source system performs deletions.
The data warehouse needs to maintain historical data integrity and provide a complete audit trail of all changes, including deletions.

#### First-Principles Solution

Let's break down each concept:

Hard Delete:

Definition: A hard delete permanently removes the record from the database by physically deleting it.
Impact on Data Integrity: Once deleted, there is no way to recover the data without a backup or point-in-time recovery mechanism.
Auditability: Less transparent since the data is completely removed.

Soft Delete:

Definition: A soft delete marks records as deleted by adding an "is_deleted" flag or timestamp column instead of physically removing them from the database.
Impact on Data Integrity: The record remains in the database but can be filtered out during queries, maintaining data integrity while providing visibility into deletion events.
Auditability: More transparent since it retains a history of deletions.

#### Example Scenario

#### Cons
ider a scenario where an order management system (source) needs to delete an order. In this context:

A hard delete would remove the order record from the source database entirely, making recovery difficult without backups.
A soft delete would update the is_deleted flag or add a deleted_at timestamp to the order record in the source database.

To ensure that these changes are captured by the data warehouse:

Source System Logic:
Implement logic to set the is_deleted flag or deleted_at timestamp when an order is deleted.
Data Warehouse Design:
The target schema should include a column to capture the deletion status (e.g., is_active).
Data replication mechanisms like CDC (Change Data Capture) can be used to detect and log these changes.

#### Closing Summary

In summary, hard deletes permanently remove records from the database, while soft deletes mark them as deleted. The choice between these methods depends on the need for data integrity, auditability, and recoverability. In a data warehouse setting, soft deletes are often preferred due to their ability to maintain historical data integrity and provide a clear audit trail.

#### Common Mistakes/Follow-Ups
Failing to Understand the Context: Not considering the specific requirements of the source system or target data warehouse.
Ignoring Replication Mechanisms: Overlooking how changes are propagated from the source to the warehouse, which can lead to inconsistencies if not properly managed.
Not Accounting for Data Recovery: Assuming that once a record is deleted, it cannot be recovered without proper backups and recovery mechanisms.

For follow-ups, you might be asked about specific tools or methods used in implementing soft deletes (e.g., triggers, stored procedures) or how CDC can integrate with ETL pipelines to ensure consistent data replication.

**Comparison:**

```
Column Name	Data Type	Description
order_id	INT	Unique identifier for the order.
customer_id	INT	ID of the customer who placed the order.
order_date	DATE	Date when the order was placed.
is_deleted	BOOLEAN	Flag indicating whether the order has been deleted (true) or not (false). Default value: false.
deleted_at	TIMESTAMP	Timestamp at which the order was marked as deleted. NULL if the order is active.
Sample Data for Soft Deleted Order
order_id: 123, customer_id: 456, order_date: '2023-09-28', is_deleted: true, deleted_at: '2023-09-29 14:30:00'
```

**Code:**

```sql
-- Example of a soft delete in SQL

-- Hard Delete
DELETE FROM orders WHERE order_id = 123;

-- Soft Delete
UPDATE orders SET is_deleted = true, deleted_at = CURRENT_TIMESTAMP WHERE order_id = 123;
```

