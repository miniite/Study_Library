---
type: "practice"
difficulty: "Deep"
---

# Why did Postgres change CTEs from an optimization fence to inlinable by default in 12+, and how did that affect your existing queries?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Recursive CTEs and Materialization](<../../../../01_Databases_and_SQL/SQL_Language/CTEs_and_Recursive_Queries/Recursive_CTEs_and_Materialization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

PostgreSQL changed its behavior regarding Common Table Expressions (CTEs) in version 12, making them inlinable by default instead of treating them as an optimization fence. This change impacts how CTEs are executed and can optimize the performance of complex queries. The key idea is that with this new default, PostgreSQL can inline CTEs directly into the main query plan, which can lead to more efficient execution plans.

#### Triage

The question asks for two main points:

Why did Postgres change CTEs from an optimization fence to inlinable?
How does this affect existing queries?

Let's start by understanding what these terms mean:

An optimization fence means that the database engine will treat a CTE as a black box, not inline it directly into the main query plan.
Inlining means that the CTE is expanded and merged with the rest of the query, potentially allowing for more optimized execution.

#### First Principles Solution
Why Did Postgres Change CTEs?

PostgreSQL made this change because of performance improvements. By default inlining CTEs can lead to better query optimization decisions. Here’s a step-by-step reasoning:

Complex Query Optimization: In complex queries, the database optimizer has more information when it can see the full plan, including expanded CTEs.
Reduced Context Switching: Treating CTEs as inlinable reduces context switching between different parts of the query execution, which can improve performance.

#### Cons
istency with Other Features: This change aligns better with other features in PostgreSQL that rely on inline views and window functions.
Impact on Existing Queries

The impact varies depending on the specific use case:

Performance Improvements: For queries where CTEs are used heavily, this can lead to significant performance gains.
Potential Breakage: Some existing queries might behave differently if they relied on certain characteristics of non-inlinable CTEs (e.g., deferred execution).
Query Rewriting: Developers may need to rewrite some queries or use explicit WITH (NOOF) clauses for specific optimizations.

#### Closing Summary
and Common Mistakes

This change in behavior requires careful consideration when upgrading or refactoring existing queries. Key points to remember:

Optimization Opportunities: Leverage the new default to improve performance where possible.
Query Analysis: Use tools like EXPLAIN to understand how your queries are being executed after the upgrade.
Testing and Validation: Thoroughly test all critical paths in your application to ensure that existing logic continues to work as expected.

#### Common Mistakes
Assuming Non-Inlining Behavior: Developers might incorrectly assume their CTEs will behave like they did before, leading to unexpected results.
Over-Reliance on WITH (NOOF): While useful for fine-tuning, overusing these clauses can make the query harder to read and maintain.
Triangulation

Understanding this change requires a deep dive into both the technical details of PostgreSQL execution plans and practical experience with how CTEs are used in real-world applications. The key takeaway is that while this change offers performance benefits, it also introduces complexity that needs careful handling during upgrades.

**Comparison:**

| id | parent_id | level |
|----|-----------|-------|
| 1  | NULL      | 1     |
| 2  | 1         | 2     |
| 3  | 1         | 2     |

**Code:**

```sql
-- Example of an old query with a non-inlinable CTE
WITH RECURSIVE sales_hierarchy AS (
    SELECT * FROM sales WHERE level = 1
    UNION ALL
    SELECT s.* FROM sales s JOIN sales_hierarchy sh ON s.parent_id = sh.id
)
SELECT ... FROM sales_hierarchy;

-- Example of an inlined CTE
WITH sales_hierarchy AS (
    SELECT * FROM sales WHERE level = 1
    UNION ALL
    SELECT s.* FROM sales s JOIN sales_hierarchy sh ON s.parent_id = sh.id
)
SELECT ... FROM sales_hierarchy;
```

**Example input / output:**

```text
Input:
WITH RECURSIVE sales_hierarchy AS (
    SELECT * FROM sales WHERE level = 1
    UNION ALL
    SELECT s.* FROM sales s JOIN sales_hierarchy sh ON s.parent_id = sh.id
)
SELECT * FROM sales_hierarchy;

Output:
| id | parent_id | level |
|----|-----------|-------|
| 1  | NULL      | 1     |
| 2  | 1         | 2     |
| 3  | 1         | 2     |
```

**Data:**

```json
{
  "sales": [
    {"id": 1, "parent_id": null, "level": 1},
    {"id": 2, "parent_id": 1, "level": 2},
    {"id": 3, "parent_id": 1, "level": 2}
  ]
}
```

**Flow:**

1. Yes
2. No
3. CTE as Optimization Fence
4. Context Switching
5. CTE as Inlinable
6. Inline Views Expansion
7. Optimization Decisions
8. Performance Gains?
9. Improved Performance
10. Query Rewrite Required

