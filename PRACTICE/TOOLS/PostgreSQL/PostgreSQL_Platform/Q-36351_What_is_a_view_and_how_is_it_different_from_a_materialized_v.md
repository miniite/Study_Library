---
type: "practice"
difficulty: "Intermediate"
---

# What is a view and how is it different from a materialized view?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Materialized CTEs](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Materialized_CTEs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In PostgreSQL, both views and materialized views are used to abstract data access and improve query performance, but they operate in fundamentally different ways. Views provide a virtual table derived from one or more tables, while materialized views store the result of a query as an actual table in the database.

To understand this better, let's break it down step by step:

#### Triage
What is Being Probed

The interviewer is likely looking for your understanding of the key differences between views and materialized views. They are probing your knowledge of how each works under the hood, their performance implications, and when to use one over the other.

#### Assumptions
You have a basic understanding of SQL and database concepts.
We will compare these constructs in terms of their implementation, usage scenarios, and trade-offs.

#### First Principles Solution

Views:

A view is essentially a stored query that returns data from one or more tables. It’s like a virtual table created by combining rows returned by the underlying SQL statement.
Views do not store any data themselves but are based on a SQL SELECT statement.
They can be used to simplify complex queries, provide security (by hiding certain columns), and encapsulate business logic.

Materialized Views:

A materialized view is a special type of view that stores the results of a query in the database as an actual table. It’s like creating a snapshot of data based on your query.
Materialized views store the result set permanently, which can improve performance for frequently accessed queries or when the underlying tables are updated infrequently.
They require periodic refreshes to keep the data up-to-date.

#### Implementation
Details

Creation and Usage:

SQL
-- Creating a view
CREATE VIEW employees_view AS
SELECT first_name, last_name, department_id FROM employees;

-- Using a materialized view
CREATE MATERIALIZED VIEW employees_materialized_view AS
SELECT first_name, last_name, department_id FROM employees;

Refresh Mechanism:

Views: No refresh mechanism; they always reflect the latest data from the underlying tables.
Materialized views: Need to be refreshed manually or automatically using triggers.

#### Performance Considerations
Speed: Materialized views can offer faster query performance since the data is stored in a table and does not require executing complex SQL queries on-the-fly. However, this comes at the cost of additional storage and maintenance overhead.
Maintenance:
Views: Lower maintenance; they are just references to underlying tables.
Materialized views: Higher maintenance due to the need for refresh operations.
Use Cases

Use Case for View: Simplifying complex queries or providing a security layer by hiding sensitive data. For example, you might have a view that aggregates sales data but hides customer-specific details.

Use Case for Materialized View: When dealing with large datasets and frequently accessed reports where the underlying data does not change often. For instance, financial reports that are generated quarterly.

#### Example

Let’s say we have an employees table and want to create a view that shows only active employees:

SQL
CREATE VIEW active_employees_view AS
SELECT * FROM employees WHERE is_active = true;

And if you frequently query this data and it rarely changes, consider using a materialized view:

SQL
CREATE MATERIALIZED VIEW active_employees_materialized_view AS
SELECT * FROM employees WHERE is_active = true;

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, views are useful for simplifying complex queries or providing security, while materialized views offer performance benefits but require more maintenance. Always consider your specific use case and the trade-offs involved.

#### Common mistakes
include:

Overusing materialized views in situations where they don’t provide significant performance improvements.
Not refreshing materialized views regularly if data changes frequently.
Ignoring the space and storage costs associated with materialized views.

#### Follow-ups
might involve discussing best practices for when to use each, or delving deeper into the specifics of view definitions and refresh strategies.

**Code:**

```sql
-- Creating a simple view
CREATE VIEW active_employees_view AS
SELECT * FROM employees WHERE is_active = true;

-- Creating a materialized view
CREATE MATERIALIZED VIEW active_employees_materialized_view AS
SELECT * FROM employees WHERE is_active = true;
```

