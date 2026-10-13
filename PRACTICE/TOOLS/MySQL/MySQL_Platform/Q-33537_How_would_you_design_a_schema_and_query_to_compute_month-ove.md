---
type: "practice"
difficulty: "Deep"
---

# How would you design a schema and query to compute month-over-month growth per product, handling months where a product had zero sales?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Activity Timeline Schema](<../../../../TOOLS/MySQL/MySQL_Platform/Activity_Timeline_Schema.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a schema and queries that compute month-over-month (MoM) growth per product while properly handling months with zero sales, we need to ensure our database schema supports flexible data storage and efficient querying. We'll use MySQL as the database system.

The core idea is to store historical sales data in such a way that it's easy to compare sales from one month to another, even when some months had no sales for certain products. This involves creating tables with appropriate indexing and using window functions or JOIN operations to calculate the required growth metrics.

