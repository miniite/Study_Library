---
type: "practice"
difficulty: "Basic"
---

# What are the different types of JOINs in SQL?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Join Types](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Join_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Enumerate and explain the JOIN types in SQL — what each returns and when to use it. This is a bread-and-butter SQL question for any backend/fullstack role. The interviewer wants confirmation that you can combine data across tables correctly, and that you understand which rows survive when there's no match.

What the interviewer is probing: (1) Do you know the four core joins — INNER, LEFT, RIGHT, FULL OUTER — plus CROSS and SELF? (2) Do you understand matched vs. unmatched rows and where NULLs appear? (3) Can you pick the right join for a real task (e.g., "find customers with no orders")?

#### Assumptions
/clarifications: Standard SQL. I'll note that MySQL historically lacks FULL OUTER JOIN (emulate with UNION), while PostgreSQL/SQL Server support all of them — a good detail to mention.

Plan: Set up two sample tables → define each join with what it keeps → show the same query across joins with concrete rows → give a decision table → summarize with common mistakes.

From first principles

A JOIN combines rows from two (or more) tables based on a related column — the join condition (usually ON a.key = b.key). The difference between join types is entirely about which unmatched rows are kept and where NULLs get filled in.

I'll use two small tables throughout:

customers(id, name) — all customers
orders(id, customer_id, amount) — orders, each pointing to a customer

Some customers have no orders; some orders could (in a messy dataset) point to a customer id that isn't listed.

1. INNER JOIN

Returns only rows that match in both tables. If a customer has no order, they don't appear; if an order has no matching customer, it doesn't appear. This is the most common join and the default — writing JOIN alone means INNER JOIN.

Use it when you want records that exist on both sides: "all customers together with their orders."

2. LEFT JOIN (LEFT OUTER JOIN)

Returns all rows from the left table, plus matching rows from the right. Where there's no match on the right, the right-side columns are filled with NULL. So every left row survives.

Use it when the left table is the "anchor" and the right is optional: "list all customers and their orders (if any)." The classic trick — finding rows with no match — uses LEFT JOIN ... WHERE right.key IS NULL (customers who never ordered).

3. RIGHT JOIN (RIGHT OUTER JOIN)

The mirror of LEFT: returns all rows from the right table, with NULLs where the left has no match. Anything a RIGHT JOIN does, you can rewrite as a LEFT JOIN by swapping table order, which is why LEFT is far more common in practice. Use RIGHT mainly for readability when the right table is naturally your anchor.

4. FULL OUTER JOIN

Returns all rows from both tables — matched rows joined together, plus unmatched left rows (right side NULL) and unmatched right rows (left side NULL). Use it to find everything, including mismatches on either side (e.g., reconciliation: customers with no orders and orders with no valid customer). Note: MySQL doesn't support FULL OUTER JOIN natively — emulate it with LEFT JOIN ... UNION ... RIGHT JOIN.

5. CROSS JOIN

Returns the Cartesian product — every row of the left paired with every row of the right, with no join condition. If left has m rows and right has n, you get m × n rows. Use it deliberately for generating combinations (e.g., every size × every color), and beware doing it by accident (a missing ON clause with old comma-join syntax explodes row counts).

6. SELF JOIN

Not a separate keyword — it's joining a table to itself using table aliases. Useful for hierarchical or same-table comparisons, e.g. employees e JOIN employees m ON e.manager_id = m.id to pair each employee with their manager.

Matched vs. unmatched — the mental model

Think of two overlapping circles (a Venn diagram):

INNER = the intersection only.
LEFT = whole left circle (intersection + left-only).
RIGHT = whole right circle.
FULL OUTER = both circles entirely.
CROSS = not a Venn overlap at all — it's every pair.
A precision point on WHERE vs. ON in outer joins

With outer joins, where you put a filter matters. A condition on the right table in the ON clause filters before the outer padding (unmatched left rows still appear with NULLs); the same condition in WHERE runs after and can silently turn a LEFT JOIN back into an INNER JOIN (because NULL = value is false). This is a common bug and a strong point to raise.

Summary — what a strong answer emphasizes
INNER = only matches; LEFT/RIGHT OUTER = keep all of one side, NULL-pad the other; FULL OUTER = keep all of both; CROSS = Cartesian product; SELF = a table joined to itself.
The distinction is entirely about which unmatched rows survive and where NULLs go.
Know the practical idioms: LEFT JOIN ... WHERE r.key IS NULL finds "no match"; RIGHT is a LEFT with swapped tables; MySQL emulates FULL OUTER with UNION.
Mention the ON vs WHERE gotcha for outer joins — it signals real experience.

#### Common mistakes
thinking JOIN alone means something other than INNER; expecting LEFT JOIN to keep unmatched right rows; accidental CROSS JOINs; putting a right-table filter in WHERE and collapsing an outer join.

#### Follow-up
probes: "Write a query for customers who never ordered," "How do you emulate FULL OUTER in MySQL?", "What's the difference between putting a condition in ON vs WHERE?"

**Comparison:**

```
Join type	Keeps left non-matches?	Keeps right non-matches?	Result set (Venn)
INNER JOIN	No	No	Intersection only
LEFT (OUTER) JOIN	Yes (right = NULL)	No	Whole left circle
RIGHT (OUTER) JOIN	No	Yes (left = NULL)	Whole right circle
FULL OUTER JOIN	Yes	Yes	Union of both circles
CROSS JOIN	n/a	n/a	Cartesian product (m × n)
SELF JOIN	depends on inner/outer	depends	table joined to itself via alias
Task	Right join to use
Records existing in both tables	INNER
All of table A, matched data from B	LEFT
Rows in A with no match in B	LEFT + WHERE b.key IS NULL
Everything, incl. mismatches on both sides	FULL OUTER
All combinations of two sets	CROSS
Rows compared within one table (hierarchy)	SELF
```

**Code:**

```sql
-- INNER JOIN: only customers who have at least one order
SELECT c.name, o.amount
FROM customers c
INNER JOIN orders o ON o.customer_id = c.id;

-- LEFT JOIN: every customer, with orders if any (NULL amount if none)
SELECT c.name, o.amount
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.id;

-- Customers who have NEVER ordered (classic anti-join idiom)
SELECT c.name
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.id
WHERE o.id IS NULL;

-- RIGHT JOIN: every order, plus its customer (mirror of LEFT)
SELECT c.name, o.amount
FROM customers c
RIGHT JOIN orders o ON o.customer_id = c.id;

-- FULL OUTER JOIN: everything on both sides
SELECT c.name, o.amount
FROM customers c
FULL OUTER JOIN orders o ON o.customer_id = c.id;

-- FULL OUTER emulation for MySQL
SELECT c.name, o.amount FROM customers c LEFT JOIN orders o ON o.customer_id = c.id
UNION
SELECT c.name, o.amount FROM customers c RIGHT JOIN orders o ON o.customer_id = c.id;

-- CROSS JOIN: every size paired with every color
SELECT s.size, cl.color FROM sizes s CROSS JOIN colors cl;

-- SELF JOIN: each employee with their manager
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

**Example input / output:**

```text
customers                orders
+----+-------+           +----+-------------+--------+
| id | name  |           | id | customer_id | amount |
+----+-------+           +----+-------------+--------+
| 1  | Asha  |           | 10 | 1           | 499    |
| 2  | Ravi  |           | 11 | 1           | 250    |
| 3  | Mira  |           | 12 | 2           | 100    |
+----+-------+           +----+-------------+--------+
   (Mira has no orders)     (all orders map to existing customers)

INNER JOIN  ->  Asha/499, Asha/250, Ravi/100
                (Mira dropped: no match)

LEFT JOIN   ->  Asha/499, Asha/250, Ravi/100, Mira/NULL
                (Mira kept, amount = NULL)

LEFT JOIN ... WHERE o.id IS NULL  ->  Mira
                (customers who never ordered)

RIGHT JOIN  ->  Asha/499, Asha/250, Ravi/100
                (all orders; here same as INNER since every order has a customer)

FULL OUTER  ->  Asha/499, Asha/250, Ravi/100, Mira/NULL

CROSS JOIN (2 sizes x 3 colors) -> 6 rows
```

**Flow:**

1. LEFT-only        Intersection        RIGHT-only
2. ┌───────────┐    ┌───────────────┐    ┌───────────┐
3. │ customers │    │  matched rows │    │  orders   │
4. │ w/o order │    │  (c.id=o.cid) │    │ w/o cust. │
5. └───────────┘    └───────────────┘    └───────────┘
6. INNER  ──────────────►  keep only this  ◄──────────────
7. LEFT   keep ────────►   keep ───────────
8. RIGHT                   keep ───────────►   keep
9. FULL   keep ────────►   keep ───────────►   keep

