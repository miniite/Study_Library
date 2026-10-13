---
type: "practice"
difficulty: "Basic"
---

# How do you drop a view and what happens to the underlying tables when you do?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Using Views Effectively](<../../../../01_Databases_and_SQL/Database_Objects/Views/Using_Views_Effectively.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To drop a view in SQL, you simply use the DROP VIEW statement followed by the name of the view. However, it's important to understand that dropping a view does not affect the underlying tables at all; only the metadata associated with the view is removed.

#### Triage

The question is asking about the impact of deleting a database view on its underlying tables. It also aims to test your understanding of how views are managed in SQL databases.

#### First Principles Solution

Let's break down what happens step by step:

Understanding Views: A view is essentially a virtual table that is derived from one or more base tables. The data for the view is not stored separately; instead, it is generated dynamically based on the underlying queries when you query the view.

Dropping a View:

To drop a view, you use the DROP VIEW command followed by the name of the view.
SQL
DROP VIEW [IF EXISTS] view_name;
The [IF EXISTS] clause is optional but useful to avoid errors if the view does not exist.

Impact on Underlying Tables: Dropping a view has no effect on the underlying tables or any other views that might be dependent on it. Only the metadata related to the view itself is removed from the database.

Recreation of View: If you need to recreate the view, you simply write and execute the CREATE VIEW statement again with the same query.

#### Closing Summary

In summary, dropping a view in SQL only removes the metadata associated with it and does not affect any underlying tables or other views that might depend on it. It's important to keep this distinction clear when managing database objects.

#### Common Mistakes/Follow-Ups
Misunderstanding the Impact: A common mistake is thinking that DROP VIEW will also delete rows from the underlying tables.
Dependent Views: Another frequent question could be about what happens if there are dependent views or stored procedures. In such cases, you might need to drop those as well before dropping the view.

