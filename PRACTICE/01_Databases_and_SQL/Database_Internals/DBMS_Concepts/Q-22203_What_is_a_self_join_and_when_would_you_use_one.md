---
type: "practice"
difficulty: "Intermediate"
---

# What is a self join and when would you use one?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Views and Materialized Views](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Views_and_Materialized_Views.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Explain what a self join is (a table joined to itself) and give the situations where you'd reach for one. This is an intermediate DBMS question, so the interviewer wants a precise definition, the mechanics (table aliases are mandatory), a couple of realistic use cases, and awareness of the join type and pitfalls.

What the interviewer is probing for:

Do you understand that a self join is just a regular join whose left and right inputs are the same table, disambiguated by aliases?
Can you name the canonical use cases — hierarchies (employee→manager), comparing rows within the same table (pairs, duplicates, consecutive rows), and finding relationships between rows of one entity?
Do you know the pitfalls: needing aliases, the INNER vs LEFT choice (e.g., the CEO has no manager), and avoiding duplicate/mirror pairs.

Plan: define it, show why aliases are required, walk through the classic employee-manager example, then list the other patterns (self-referencing hierarchy, same-table comparisons, gaps/sequences), and close with pitfalls.

What a self join is

A self join is a join in which a table is joined to itself. There is no special SELF JOIN keyword — you write a normal INNER JOIN / LEFT JOIN where the same table appears twice, each occurrence given a different alias so the engine (and you) can tell the two "copies" apart. Conceptually the database takes two logical copies of the table and matches rows across them using a join condition.

SQL
SELECT ...
FROM employees e          -- copy 1: "the employee"
JOIN employees m          -- copy 2: "the manager"
  ON e.manager_id = m.employee_id;

Aliases are mandatory. Without e and m, a column like employee_id would be ambiguous — the parser can't know which copy you mean. The aliases give each copy a role ("employee" vs "manager").

When you'd use one

1. Self-referencing hierarchies (the #1 use case). When a table has a foreign key pointing back to its own primary key, a self join walks that relationship. The classic is employees(employee_id, name, manager_id) where manager_id references another employee_id. To list each employee alongside their manager's name, you join the table to itself.

Use a LEFT JOIN here so top-level people (CEO with manager_id IS NULL) still appear with a NULL manager; an INNER JOIN would silently drop them. This pattern also covers categories→parent-category, comments→parent-comment (threads), parts→sub-parts (BOM), and org charts. (Note: a self join gives you one level up; for arbitrary depth you use a recursive CTE, which is the natural follow-up.)

2. Comparing rows within the same table. When the relationship you care about is between two rows of the same entity, a self join pairs them:

Find employees who earn more than their manager: join employee↔manager and filter e.salary > m.salary.
Find pairs of people in the same department (but different people): join on a.dept = b.dept AND a.id < b.id — the < avoids self-pairs and mirror duplicates.
Find duplicate records: join on the columns that should be unique, with a.id <> b.id, to surface rows that collide.

3. Comparing consecutive / adjacent rows (gaps, deltas, sequences). Before window functions, self joins were the standard way to compare a row to the "next" or "previous" one — e.g., day-over-day change by joining sales s1 to sales s2 ON s2.date = s1.date - 1. (In modern SQL, LAG/LEAD window functions usually do this more cleanly, which is worth mentioning as the modern alternative.)

Worked mechanics

For "employee with their manager's name":

SQL
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m
  ON e.manager_id = m.employee_id
ORDER BY e.name;
e plays "employee," m plays "manager."
ON e.manager_id = m.employee_id links each employee to the row that is their manager.
LEFT JOIN keeps employees with no manager (the row shows manager = NULL).
Pitfalls to call out
Missing aliases → "ambiguous column" errors. Always alias both copies with meaningful names.
Wrong join type → INNER JOIN drops root rows (no manager, no parent). Use LEFT JOIN when the self-reference can be NULL.
Duplicate/mirror pairs in "find pairs" queries → without a.id < b.id, you get (A,B) and (B,A) and (A,A). Use an inequality on the id.
Only one level deep → a plain self join climbs one hop; for full ancestry/descendants use a WITH RECURSIVE CTE.
Performance → the join condition should be indexed (e.g., index on manager_id and PK on employee_id); a self join is a normal join and benefits from the same indexing.
Summary / what a strong answer emphasizes

A self join joins a table to itself using distinct aliases and an ON condition that relates rows of the same table. Reach for it when the meaningful relationship is row-to-row within one table: self-referencing hierarchies (employee↔manager, category↔parent), comparing rows (earns-more-than-manager, same-department pairs, duplicates), and adjacent-row comparisons. Choose LEFT JOIN when the reference can be NULL, use an inequality to avoid mirror pairs, and remember that recursion (recursive CTE) and window functions (LAG/LEAD) are the modern extensions for multi-level and adjacent-row cases. Follow-up probes: "How would you get the whole management chain?" (recursive CTE), and "Rewrite the consecutive-row comparison with a window function."

**Comparison:**

```
Use case	Join condition idea	Join type
Employee → manager name	e.manager_id = m.employee_id	LEFT (keep roots)
Category → parent category	c.parent_id = p.category_id	LEFT
Earns more than manager	e.manager_id = m.employee_id + e.salary > m.salary	INNER
Distinct same-group pairs	a.grp = b.grp AND a.id < b.id	INNER
Find duplicates	join on "unique" cols + a.id <> b.id	INNER
Consecutive-row delta	s2.date = s1.date - 1 (or use LAG/LEAD)	INNER/LEFT
```

**Code:**

```

-- Employee with their manager's name (LEFT JOIN keeps the CEO) SELECT e.name AS employee, m.name AS manager FROM employees e LEFT JOIN employees m ON e.manager_id = m.employee_id ORDER BY e.name;

CODE
Copy

```sql
-- Employees who earn more than their manager
SELECT e.name AS employee, e.salary, m.name AS manager, m.salary AS mgr_salary
FROM employees e
JOIN employees m ON e.manager_id = m.employee_id
WHERE e.salary > m.salary;
SQL
Copy
-- Distinct pairs of employees in the same department (no self/mirror pairs)
SELECT a.name AS emp_a, b.name AS emp_b, a.department_id
FROM employees a
JOIN employees b
  ON a.department_id = b.department_id
 AND a.employee_id  < b.employee_id;
```

**Example input / output:**

```

"Employee with manager" (LEFT JOIN):

CODE
Copy
employee  manager
Riya      NULL      <- kept because LEFT JOIN; INNER JOIN would drop Riya
Sanjay    Riya
Tara      Sanjay
Umang     Sanjay

"Earns more than manager": Umang(130000) vs manager Sanjay(180000) -> no; Tara(120000) vs Sanjay -> no; Sanjay(180000) vs Riya(250000) -> no. Result: (empty set) for this data — a correct and instructive empty result.

"Same-department pairs" with a.id < b.id:

CODE
Copy
emp_a   emp_b   department_id
Riya    Sanjay  10
Tara    Umang   20

(Each pair appears once; no Riya-Riya, no mirrored Sanjay-Riya.)
```

**Data:**

```text
employees
+-------------+---------+------------+--------+---------------+
| employee_id | name    | manager_id | salary | department_id |
+-------------+---------+------------+--------+---------------+
| 1           | Riya    | NULL       | 250000 | 10            |   <- CEO, no manager
| 2           | Sanjay  | 1          | 180000 | 10            |
| 3           | Tara    | 2          | 120000 | 20            |
| 4           | Umang   | 2          | 130000 | 20            |
+-------------+---------+------------+--------+---------------+
```

**Flow:**

1. employees
2. e.manager_id =m.employee_id
3. copy e: 'employee'
4. copy m: 'manager'
5. (row: employee + managername)

