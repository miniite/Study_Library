---
type: "practice"
difficulty: "Deep"
---

# How would you store and query a hierarchical / tree structure (like an org chart) in SQL?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Recursive CTEs](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Recursive_CTEs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: How do you model a tree — an org chart where every employee has one manager, a manager has many reports, and depth is arbitrary — inside a relational database that natively speaks in flat tables, and then how do you query it (find all reports under someone, find the chain up to the CEO, move a subtree, etc.)?

What the interviewer is probing for: Do you know the classic modeling patterns (adjacency list, path enumeration, nested sets, closure table), their read/write trade-offs, and when to use each? Can you write a recursive query (recursive CTE) on the default model? Do you understand that the "right" answer depends on the read/write mix and the DB engine?

Clarifying questions I'd ask: How deep does the tree go? Is it read-heavy (rarely reorg) or write-heavy (constant moves)? Do we need "all descendants" fast, or just parent/child? Which engine (Postgres, MySQL 8+, SQL Server, Oracle all support recursive CTEs; older MySQL 5.x does not)? Is it a strict tree (one parent) or a DAG (matrix org, multiple managers)?

Plan: I'll present the four canonical models, show the adjacency-list default with a recursive CTE (the answer expected at most companies), then compare the alternatives and give a decision rule.

From first principles

A tree is nodes + edges. In SQL the natural representation of an edge is a foreign key on the row pointing at its parent. That single idea is the adjacency list, and it is the default model everywhere.

Model 1 — Adjacency list (store parent_id on each row)
SQL
CREATE TABLE employee (
  id         INT PRIMARY KEY,
  name       VARCHAR(100),
  manager_id INT NULL REFERENCES employee(id)   -- NULL = root (CEO)
);

#### Pros
trivial to insert; moving a node = a single UPDATE manager_id; storage minimal; the schema is obvious and normalized.

#### Cons
"give me all reports under Alice, any depth" needs recursion. In modern SQL that's a recursive CTE:
SQL
WITH RECURSIVE subtree AS (
  SELECT id, name, manager_id, 0 AS depth
  FROM employee WHERE id = :root_id          -- anchor
  UNION ALL
  SELECT e.id, e.name, e.manager_id, s.depth + 1
  FROM employee e JOIN subtree s ON e.manager_id = s.id  -- recurse
)
SELECT * FROM subtree;

The anchor selects the start node; the recursive term repeatedly joins the "frontier" to their children until no new rows appear. Reverse the join (s.manager_id = e.id) to walk up to the CEO. This works in Postgres, MySQL 8+, SQL Server (also CONNECT BY in Oracle), and is the answer I'd lead with.

Index the traversal column: CREATE INDEX idx_emp_mgr ON employee(manager_id);

Model 2 — Closure table (materialize every ancestor→descendant pair)

Keep the adjacency list and a second table listing every (ancestor, descendant, depth) pair, including self-links at depth 0.

SQL
CREATE TABLE employee_closure (
  ancestor_id   INT,
  descendant_id INT,
  depth         INT,
  PRIMARY KEY (ancestor_id, descendant_id)
);

#### Pros
all descendants or all ancestors is a plain indexed lookup — no recursion, blazing fast reads. Handles arbitrary depth. Enforces integrity well.

#### Cons
writes are heavier. Inserting a node inserts one closure row per ancestor; moving a subtree means deleting the subtree's old ancestor links and inserting new ones — a bounded but non-trivial set of statements. Extra storage O(nodes × avg depth).
Use when: read-heavy, deep trees, you frequently ask "everything under X" (permissions, category trees, org rollups).
Model 3 — Path enumeration / materialized path

Store the full path as a string: /1/4/9/. "All descendants of 4" = WHERE path LIKE '/1/4/%'.

#### Pros
descendant queries are a single LIKE (index-friendly if the prefix is anchored); shows the whole ancestry in one column; ordering by path gives a natural tree order.

#### Cons
moving a subtree rewrites every descendant's path; the path column can get long; referential integrity to ancestors isn't enforced by FKs. Postgres's ltree type formalizes this pattern with GiST indexes and is genuinely nice.
Model 4 — Nested set (left/right pre-order numbers)

Number nodes with lft/rgt so a node's descendants are exactly the rows where lft is between the parent's lft and rgt.

#### Pros
whole-subtree reads are a single range predicate, very fast, no recursion.

#### Cons
any insert or move renumbers large swaths of the tree (UPDATE ... SET lft = lft + 2 WHERE lft > :x) — brutal for write-heavy org charts and hard to keep correct under concurrency. Mostly a legacy/read-only choice today; closure tables have largely superseded it.
Decision rule (what a strong answer emphasizes)
Default to adjacency list + recursive CTE. It's simplest, normalized, cheap to write, and modern engines traverse it well for typical org depths (a few dozen levels, thousands of nodes).
If reads dominate and you constantly need full subtrees/permission rollups, add a closure table as a read-optimized companion (keep it in sync inside the same transaction or a trigger).
Materialized path (ltree) is a great middle ground when you want cheap prefix queries and don't reorg often.
Nested set only for near-static, read-only hierarchies.
DAG / multiple managers: adjacency list breaks (row has one parent_id); use a separate edge table (parent_id, child_id) — a many-to-many — and traverse it with a recursive CTE, adding a visited-set guard to avoid cycles.

#### Common mistakes
to avoid: assuming a self-join at fixed depth handles arbitrary depth (it doesn't — depth is unbounded); forgetting to index the parent/traversal column; picking nested sets for a write-heavy org chart; not guarding recursion against cycles in a DAG (Postgres offers CYCLE ... SET ... USING); and ignoring that old MySQL 5.x lacks recursive CTEs (there you fall back to closure table or app-side traversal).

#### Follow-up
probes I'd be ready for: write the "path to CEO" query; how to move a subtree in each model; how to prevent cycles; performance at 1M nodes; how permission checks ("can user see everything under node N") map to a closure-table join.

Summary

Store the tree as an adjacency list (parent_id FK) by default and traverse with a recursive CTE; when reads of whole subtrees dominate, add a closure table (or use materialized path / ltree) as a read accelerator; reserve nested sets for static data; and for multi-parent (DAG) orgs switch to an explicit edge table with cycle-safe recursion. The judgment being tested is matching the model to the read/write mix and the engine's capabilities.

**Comparison:**

```
Model	All-descendants read	Insert leaf	Move subtree	Storage	Best for
Adjacency list (parent_id)	Recursive CTE (O(subtree))	1 row	1 UPDATE	Minimal	Default; write-friendly; modern engines
Closure table	Indexed lookup (no recursion)	+1 row per ancestor	Delete+insert subtree links	O(n × depth)	Read-heavy, deep, permission rollups
Materialized path / ltree	LIKE '/1/4/%' (prefix)	1 row	Rewrite all descendant paths	Path string	Cheap prefix queries, rare reorg
Nested set (lft/rgt)	Range predicate (fast)	Renumber many rows	Renumber many rows	Two ints/row	Static, read-only trees (legacy)
```

**Code:**

```sql
-- Model 1: Adjacency list (default)
CREATE TABLE employee (
  id         INT PRIMARY KEY,
  name       VARCHAR(100),
  manager_id INT NULL REFERENCES employee(id)
);
CREATE INDEX idx_emp_mgr ON employee(manager_id);

-- All reports under a given manager (any depth) via recursive CTE
WITH RECURSIVE subtree AS (
  SELECT id, name, manager_id, 0 AS depth
  FROM employee WHERE id = :root_id
  UNION ALL
  SELECT e.id, e.name, e.manager_id, s.depth + 1
  FROM employee e
  JOIN subtree s ON e.manager_id = s.id
)
SELECT id, name, depth FROM subtree ORDER BY depth, id;

-- Chain UP to the CEO (walk ancestors)
WITH RECURSIVE chain AS (
  SELECT id, name, manager_id FROM employee WHERE id = :start_id
  UNION ALL
  SELECT e.id, e.name, e.manager_id
  FROM employee e JOIN chain c ON c.manager_id = e.id
)
SELECT id, name FROM chain;

-- Oracle equivalent for descendants
SELECT id, name, LEVEL AS depth
FROM employee
START WITH id = :root_id
CONNECT BY PRIOR id = manager_id;

-- Model 2: Closure table — all descendants with NO recursion
CREATE TABLE employee_closure (
  ancestor_id   INT,
  descendant_id INT,
  depth         INT,
  PRIMARY KEY (ancestor_id, descendant_id)
);
SELECT e.id, e.name, c.depth
FROM employee_closure c
JOIN employee e ON e.id = c.descendant_id
WHERE c.ancestor_id = :root_id;   -- indexed lookup, fast

-- DAG (multiple managers): edge table + cycle-safe recursion (Postgres)
CREATE TABLE reports_to (parent_id INT, child_id INT, PRIMARY KEY(parent_id, child_id));
WITH RECURSIVE dag AS (
  SELECT child_id AS id, 1 AS depth FROM reports_to WHERE parent_id = :root_id
  UNION
  SELECT r.child_id, d.depth + 1 FROM reports_to r JOIN dag d ON r.parent_id = d.id
) CYCLE id SET is_cycle USING path
SELECT DISTINCT id, depth FROM dag;
```

**Example input / output:**

```text
Recursive-CTE dry run — descendants of Bob (:root_id = 2)

Anchor:      subtree = { (2,Bob,1,depth0) }
Iter 1: join children of {2} -> Dave(4), Eve(5)
             subtree += { (4,Dave,2,d1), (5,Eve,2,d1) }
Iter 2: join children of {4,5} -> Frank(6) (child of 4)
             subtree += { (6,Frank,4,d2) }
Iter 3: children of {6} -> none. Recursion stops.

Result:
id | name  | depth
2  | Bob   | 0
4  | Dave  | 1
5  | Eve   | 1
6  | Frank | 2

Closure-table equivalent (ancestor_id = 2): rows (2,2,0)(2,4,1)(2,5,1)(2,6,2)
-> same descendants, single indexed SELECT, no iteration.
```

**Data:**

```text
-- Sample org (adjacency list rows)
id | name    | manager_id
1  | Alice   | NULL        -- CEO (root)
2  | Bob     | 1
3  | Carol   | 1
4  | Dave    | 2
5  | Eve     | 2
6  | Frank   | 4

-- Same tree materialized in employee_closure (ancestor, descendant, depth)
(1,1,0) (2,2,0) (3,3,0) (4,4,0) (5,5,0) (6,6,0)   -- self links
(1,2,1) (1,3,1) (2,4,1) (2,5,1) (4,6,1)           -- direct
(1,4,2) (1,5,2) (2,6,2)                           -- 2 levels
(1,6,3)                                            -- CEO -> Frank
```

**Flow:**

1. Writes / frequent reorg
2. Read-heavy, need fullsubtrees
3. No, e.g. MySQL 5.x
4. DAG / multi-manager
5. Yes
6. Choose a tree model
7. Write-heavy or read-heavy?
8. Adjacency list + recursive CTE
9. Closure table alongsideadjacency list
10. Engine supports recursive CTE?
11. Strict tree or DAG?
12. Edge table + cycle-saferecursion
13. Want cheap prefix queries, raremoves?
14. Materialized path / ltree

