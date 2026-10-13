---
type: "practice"
difficulty: "Expert"
---

# What guards do you need in a recursive CTE to avoid infinite recursion on cyclic data?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Recursive CTEs](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Recursive_CTEs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

A recursive Common Table Expression (CTE) can lead to infinite recursion if not properly guarded, especially when dealing with cyclic data. The primary guards needed are a termination condition and an exclusion of already processed rows.

#### Triage

The core question is about ensuring the termination of a recursive CTE in SQL when working with cyclic or circular data structures. We need to understand how the WITH clause works in SQL, specifically focusing on the conditions that prevent an infinite loop.

#### First-Principles Solution
Terminating Condition

A recursive CTE must have at least one condition that guarantees it will eventually stop executing. This is typically achieved by including a termination condition in the recursive part of the CTE. For example, if you are traversing a hierarchy where each node has a parent and children relationship, you can terminate when reaching the root nodes (nodes with no parents).

SQL
WITH RECURSIVE hierarchy AS (
    -- Anchor member: start point for recursion
    SELECT id, name, parent_id FROM nodes WHERE is_root = true

    UNION ALL

    -- Recursive member: move to child nodes
    SELECT n.id, n.name, n.parent_id
    FROM nodes n
    JOIN hierarchy h ON n.parent_id = h.id
)
SELECT * FROM hierarchy;
Exclusion of Already Processed Rows

To avoid revisiting the same row multiple times, which could lead to infinite recursion, we can add a column to track whether a node has already been processed. This is typically done by including an additional condition in the recursive member.

SQL
WITH RECURSIVE hierarchy AS (
    -- Anchor member: start point for recursion
    SELECT id, name, parent_id FROM nodes WHERE is_root = true

    UNION ALL

    -- Recursive member: move to child nodes and exclude already processed rows
    SELECT n.id, n.name, n.parent_id
    FROM nodes n
    JOIN hierarchy h ON n.parent_id = h.id AND n.id NOT IN (SELECT id FROM processed_nodes)
)
SELECT * FROM hierarchy;

#### Example
Table Schema
SQL
CREATE TABLE nodes (
    id INT PRIMARY KEY,
    name TEXT,
    parent_id INT REFERENCES nodes(id),
    is_root BOOLEAN DEFAULT FALSE,
    processed BOOLEAN DEFAULT FALSE
);

INSERT INTO nodes (id, name, parent_id, is_root) VALUES
(1, 'A', NULL, TRUE),  -- Root node
(2, 'B', 1, FALSE),
(3, 'C', 1, FALSE),
(4, 'D', 2, FALSE);
Dry-Run Example

Given the example data and query:

SQL
WITH RECURSIVE hierarchy AS (
    SELECT id, name, parent_id FROM nodes WHERE is_root = true

    UNION ALL

    SELECT n.id, n.name, n.parent_id
    FROM nodes n
    JOIN hierarchy h ON n.parent_id = h.id AND n.processed = FALSE
)
SELECT * FROM hierarchy;

The processed column ensures that a node is only visited once. The recursive call stops when all nodes are processed.

#### Closing Summary
and Common Mistakes

To avoid infinite recursion in a recursive CTE, it's crucial to:

Define a termination condition.
Exclude already processed rows using a tracking mechanism like an additional column.
Ensure that the recursive member does not re-enter rows that have been processed previously.

#### Common mistakes
include neglecting to define a clear termination condition or failing to exclude already visited nodes, leading to unnecessary computation and potential infinite loops.

#### Follow-Ups
How would you optimize the query for large datasets?
What if the data structure is more complex (e.g., multiple parent relationships)?
Can you think of any other methods to guard against infinite recursion?

**Comparison:**

```
A
B
C
D
```

**Code:**

```sql
WITH RECURSIVE hierarchy AS (
    -- Anchor member: start point for recursion
    SELECT id, name, parent_id FROM nodes WHERE is_root = true

    UNION ALL

    -- Recursive member: move to child nodes and exclude already processed rows
    SELECT n.id, n.name, n.parent_id
    FROM nodes n
    JOIN hierarchy h ON n.parent_id = h.id AND n.processed = FALSE
)
SELECT * FROM hierarchy;
```

**Example input / output:**

```

WITH RECURSIVE hierarchy AS ( SELECT id, name, parent_id FROM nodes WHERE is_root = true

CODE
Copy
UNION ALL

SELECT n.id, n.name, n.parent_id
FROM nodes n
JOIN hierarchy h ON n.parent_id = h.id AND n.processed = FALSE

) SELECT * FROM hierarchy;

CODE
Copy
Output:

id | name | parent_id 1 | A | NULL 2 | B | 1 3 | C | 1 4 | D | 2
```

**Data:**

```sql
CREATE TABLE nodes (
    id INT PRIMARY KEY,
    name TEXT,
    parent_id INT REFERENCES nodes(id),
    is_root BOOLEAN DEFAULT FALSE,
    processed BOOLEAN DEFAULT FALSE
);

INSERT INTO nodes (id, name, parent_id, is_root) VALUES
(1, 'A', NULL, TRUE),  -- Root node
(2, 'B', 1, FALSE),
(3, 'C', 1, FALSE),
(4, 'D', 2, FALSE);
```

**Flow:**

1. Not Processed
2. Start Point (Root Node)
3. (Recursive Call)
4. Check if Processed
5. Process and Mark as Processed
6. Move to Next Level orTerminate

