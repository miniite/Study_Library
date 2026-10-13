---
type: "practice"
difficulty: "Deep"
---

# Explain when you'd reach for a recursive CTE with cycle detection, walk through a bill-of-materials explosion, and describe how the recursion could blow up memory on bad data.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Recursive CTEs](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Recursive_CTEs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with hierarchical or tree-like structures in SQL, such as those found in bills of materials (BOM), product hierarchies, or organizational charts, recursive Common Table Expressions (CTEs) are a powerful tool. Recursive CTEs allow you to traverse these complex data relationships efficiently and accurately. However, it's crucial to understand when to use them and how they can lead to issues like stack overflow if not properly managed.

#### Triage
What the Question is Asking

The core of this question revolves around understanding recursive CTEs, specifically in contexts where cycle detection and memory usage are critical concerns. The interviewer is probing your knowledge of:

When to employ recursive CTEs for tasks such as BOM explosions.
How to implement cycle detection within these structures.
Potential pitfalls related to recursion, particularly regarding memory consumption.

#### Assumptions
We're working with a database that supports recursive CTEs (such as PostgreSQL or SQL Server).
The data model includes hierarchical relationships and potentially cyclic dependencies.
Step-by-Step Solution
Scenario: Bill of Materials Explosion

Imagine you have a BOM table where each row represents an item in the manufacturing process, and parent-child relationships link items to their components. You want to expand this BOM to show all sub-components recursively until no more direct children exist for any component.

SQL
WITH RECURSIVE bom_expansion AS (
    -- Anchor member: select initial rows
    SELECT *
    FROM bom_table

    UNION ALL

    -- Recursive member: join with self and add child components
    SELECT b.*
    FROM bom_table b
    INNER JOIN bom_expansion be ON b.parent_item_id = be.item_id
)
SELECT * FROM bom_expansion;
Cycle Detection in Recursive CTEs

In a BOM, it's possible for items to have circular dependencies (e.g., item A is part of B, and B is part of A). To detect such cycles, you can use an additional table or column that tracks the state of each item. For instance:

SQL
WITH RECURSIVE bom_expansion AS (
    SELECT item_id, parent_item_id, 0 AS cycle_detected
    FROM bom_table

    UNION ALL

    SELECT b.item_id, b.parent_item_id,
           CASE WHEN exists (SELECT 1 FROM bom_expansion be WHERE be.item_id = b.item_id) THEN 1 ELSE 0 END as cycle_detected
    FROM bom_table b
    INNER JOIN bom_expansion be ON b.parent_item_id = be.item_id
)
SELECT * FROM bom_expansion;

Here, the cycle_detected column helps identify cycles.

Memory Blow-Up on Bad Data

Recursive CTEs can lead to stack overflow if there are large cycles or deeply nested hierarchies. The database limits the recursion depth to prevent infinite loops and memory exhaustion. However, this limit is often not sufficient for very complex data structures.

To mitigate this:

Limit Recursive Depth: Use WITH RECURSIVE directly with a depth limitation.
Materialize Results: Store intermediate results in temporary tables or views.
Optimize Data Structure: Ensure that your data model avoids deep nesting and cycles where possible.

#### Closing Summary

Recursive CTEs are essential for handling hierarchical and tree-like structures, such as BOM explosions. However, careful consideration must be given to cycle detection and potential memory issues. Understanding the underlying mechanisms and limitations is key to leveraging recursive CTEs effectively without running into performance or stability problems.

#### Common Mistakes/Follow-Ups
Not Handling Cycles: Failing to implement cycle detection can lead to infinite recursion.
Ignoring Recursive Depth Limits: Not considering the stack limits of your database can result in a crash due to excessive recursion depth.
Inefficient Data Model: Poorly designed data structures, such as deeply nested hierarchies without proper normalization, can exacerbate performance issues.

**Comparison:**

| item_id | parent_item_id | cycle_detected |
|---------|---------------|----------------|
| 1       | null          | 0              |
| 2       | 1             | 0              |
| 3       | 2             | 0              |
| 4       | 3             | 0              |
| 5       | 2             | 1              |

**Code:**

```sql
WITH RECURSIVE bom_expansion AS (
    SELECT item_id, parent_item_id, 0 AS cycle_detected
    FROM bom_table

    UNION ALL

    SELECT b.item_id, b.parent_item_id,
           CASE WHEN exists (SELECT 1 FROM bom_expansion be WHERE be.item_id = b.item_id) THEN 1 ELSE 0 END as cycle_detected
    FROM bom_table b
    INNER JOIN bom_expansion be ON b.parent_item_id = be.item_id
)
SELECT * FROM bom_expansion;
```

**Example input / output:**

```text
Input:
WITH RECURSIVE bom_expansion AS (
    SELECT item_id, parent_item_id, 0 AS cycle_detected
    FROM bom_table

    UNION ALL

    SELECT b.item_id, b.parent_item_id,
           CASE WHEN exists (SELECT 1 FROM bom_expansion be WHERE be.item_id = b.item_id) THEN 1 ELSE 0 END as cycle_detected
    FROM bom_table b
    INNER JOIN bom_expansion be ON b.parent_item_id = be.item_id
)
SELECT * FROM bom_expansion;

Output:
+--------+-------------+----------------+
| item_id | parent_item_id | cycle_detected |
+--------+-------------+----------------+
| 1      | null        | 0              |
| 2      | 1           | 0              |
| 3      | 2           | 0              |
| 4      | 3           | 0              |
| 5      | 2           | 1              |
+--------+-------------+----------------+
```

**Data:**

```json
{
  "bom_table": [
    {"item_id": 1, "parent_item_id": null},
    {"item_id": 2, "parent_item_id": 1},
    {"item_id": 3, "parent_item_id": 2},
    {"item_id": 4, "parent_item_id": 3},
    {"item_id": 5, "parent_item_id": 2}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is item_id in bom_expansion?
5. Return 1
6. Insert into bom_expansion andset cycle_detected = 0
7. Recursively process childcomponents
8. Check for cycles on children
9. If no cycle, continue recursion;else mark as detected

