---
type: "practice"
difficulty: "Basic"
---

# What is a view in SQL and what are its advantages?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1 · [↑ Views and Materialized Views](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Views_and_Materialized_Views.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

This is a basic-concept question: define a view in SQL and list its advantages. The interviewer wants a clean, correct definition, a simple example, and the concrete benefits (simplicity, security, abstraction, reuse) — plus, to stand out, a brief mention of updatable views and materialized views so they know you're aware of the boundaries. The tone should teach the fundamentals plainly and define every term. No deep system-design needed here.

What is a view?

A view is a virtual table defined by a stored SELECT query. It does not (by default) store any data of its own — it stores the query. When you SELECT from a view, the database runs the underlying query and returns the current results from the base tables. So a view always reflects the latest data in those tables.

Think of it as a saved, named query that you can treat as if it were a table:

SQL
CREATE VIEW active_customers AS
SELECT customer_id, name, email
FROM customers
WHERE status = 'ACTIVE';

Now anyone can run SELECT * FROM active_customers; and get only active customers, without knowing the filter or the underlying table structure.

Key point: virtual vs. stored
A regular (standard) view stores no data. Each query re-executes the definition against the base tables. It's always fresh but pays the query cost every time.
A materialized view does store the result on disk (a physical snapshot). It's fast to read but can be stale until it is refreshed (REFRESH MATERIALIZED VIEW). You trade freshness for read speed. (PostgreSQL and Oracle support these natively; MySQL does not have built-in materialized views.)

Mentioning this distinction shows depth beyond the textbook definition.

Advantages of views

Simplicity / abstraction. A complex query with multiple joins, filters, and aggregations can be wrapped in a view. Users then query a simple "table" name instead of rewriting the join every time. This reduces errors and duplicated SQL.

Security / access control. Views are a core security tool. You can grant a user access to a view but not the underlying tables, exposing only certain columns (e.g., hide salary or password_hash) and certain rows (e.g., only their own department's data). This is column-level and row-level security without touching the base table permissions.

#### Cons
istency and reuse. Business logic (e.g., "what counts as an active customer" or "how we compute net revenue") is defined once in the view. Every report and query using the view stays consistent, and changing the rule in one place updates everyone.

Backward compatibility / decoupling. If you rename or restructure a base table, you can keep an old view name/shape working by updating only the view definition. Applications querying the view don't break — the view insulates them from schema changes.

Readability. Named views make queries self-documenting: SELECT * FROM monthly_sales_summary reads better than a 40-line aggregation.

Small caveats to mention (shows maturity)
Performance: a standard view runs its query every time, so a view over an expensive join isn't automatically fast — indexing and the underlying query still matter. Use a materialized view when read performance matters and slight staleness is acceptable.
Updatability: simple views (single table, no aggregates/DISTINCT/GROUP BY) can often be updated via INSERT/UPDATE/DELETE, which pass through to the base table. Complex views (joins, aggregates) are generally read-only unless you add an INSTEAD OF trigger. Adding WITH CHECK OPTION prevents inserts/updates through the view that would fall outside its WHERE filter.
Summary

A view is a named, virtual table backed by a stored SELECT; it holds no data itself (a materialized view does) and always reflects the current base-table data. Its main advantages are simplicity/abstraction of complex queries, security via restricting exposed rows and columns, consistency/reuse of business logic, decoupling applications from schema changes, and readability. Bonus points for noting that standard views don't improve performance on their own (materialized views do, at the cost of freshness) and that only simple views are directly updatable.

#### Follow-up
probes: "Can you update data through a view?" (simple views yes; complex views need INSTEAD OF triggers), "View vs materialized view?" (virtual/fresh vs stored/stale-until-refresh), and "How do views help security?" (grant on the view, revoke on base tables to expose only selected rows/columns).

**Comparison:**

```
Feature	Standard View	Materialized View
Stores data?	No (virtual, re-runs query)	Yes (physical snapshot)
Data freshness	Always current	Stale until REFRESH
Read speed	Same as underlying query	Fast (precomputed)
Storage cost	~none	Uses disk
Support	All major RDBMS	PostgreSQL, Oracle (not MySQL)
Advantage	What it gives you
Simplicity	Wrap complex joins behind one name
Security	Expose only chosen rows/columns; grant on view
Consistency	Business logic defined once, reused everywhere
Decoupling	Schema changes hidden behind stable view
Readability	Self-documenting query names
```

**Code:**

```sql
-- 1. Create a simple view (virtual table) hiding sensitive columns.
CREATE VIEW public_employee AS
SELECT emp_id, name, department       -- salary and ssn deliberately excluded
FROM employees;

-- 2. Query the view exactly like a table.
SELECT * FROM public_employee WHERE department = 'Engineering';

-- 3. Security: let a reporting role read only the view.
GRANT SELECT ON public_employee TO reporting_role;
-- (reporting_role has NO privilege on the base 'employees' table -> salary hidden)

-- 4. A view wrapping a complex join, reused everywhere.
CREATE VIEW order_summary AS
SELECT c.customer_id, c.name, COUNT(o.order_id) AS num_orders,
       COALESCE(SUM(o.amount), 0) AS lifetime_value
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id
GROUP BY c.customer_id, c.name;

-- 5. Enforce the view's filter on writes (row-level guard).
CREATE VIEW active_customers AS
SELECT customer_id, name, status FROM customers WHERE status = 'ACTIVE'
WITH CHECK OPTION;   -- blocks inserting/updating a row to status <> 'ACTIVE'

-- 6. Materialized view: stored snapshot for fast reads (PostgreSQL/Oracle).
CREATE MATERIALIZED VIEW daily_sales AS
SELECT sale_date, SUM(amount) AS total FROM orders GROUP BY sale_date;
REFRESH MATERIALIZED VIEW daily_sales;   -- update the snapshot on demand

-- Remove a view.
DROP VIEW public_employee;
```

**Example input / output:**

```text
SELECT * FROM public_employee;
  -> (1, Aisha, Engineering)
     (2, Ben,   Sales)          # salary/ssn never returned

-- Standard view reflects live data:
UPDATE employees SET department = 'Platform' WHERE emp_id = 1;
SELECT * FROM public_employee WHERE emp_id = 1;
  -> (1, Aisha, Platform)       # immediately updated, no refresh needed

-- WITH CHECK OPTION guard:
INSERT INTO active_customers(customer_id,name,status) VALUES (9,'X','INACTIVE');
  -> ERROR: new row violates check option for view "active_customers"
```

**Data:**

```text
employees (base table)
+--------+--------+-------------+--------+-----------+
| emp_id | name   | department  | salary | ssn       |
+--------+--------+-------------+--------+-----------+
| 1      | Aisha  | Engineering | 120000 | ***-**-01 |
| 2      | Ben    | Sales       |  70000 | ***-**-02 |
+--------+--------+-------------+--------+-----------+

public_employee (VIEW) exposes only:
+--------+--------+-------------+
| emp_id | name   | department  |   <- salary & ssn are NOT visible
+--------+--------+-------------+
```

**Flow:**

1. SELECT * FROM view
2. runs definition at querytime
3. User / App
4. VIEW: stored SELECT
5. (Base table A)
6. (Base table B)
7. Live result rows

