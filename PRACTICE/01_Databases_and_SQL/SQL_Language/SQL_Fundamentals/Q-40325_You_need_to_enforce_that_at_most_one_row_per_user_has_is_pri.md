---
type: "practice"
difficulty: "Intermediate"
---

# You need to enforce that at most one row per user has is_primary = true. Show the partial unique index approach and explain why a plain unique constraint or trigger is worse.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Unique Constraints for Dedup](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Unique_Constraints_for_Dedup.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that only one row per user can have is_primary set to true, we need to enforce this uniqueness constraint at the database level. A partial unique index approach is more suitable than a plain unique constraint or trigger because it allows us to apply the uniqueness condition based on specific columns, in this case, user_id. Let's break down why each method might be less ideal and how to implement the partial unique index.

#### Triage

The question asks for an explanation of using a partial unique index over other methods (like a plain unique constraint or trigger). We need to explain the limitations of these alternatives and demonstrate how a partial unique index can achieve the desired behavior effectively.

#### First-Principles Solution
Plain Unique Constraint

A plain unique constraint enforces that no two rows can have identical values in the specified column(s). If we apply this to is_primary alone, it would mean that only one row across all users can be marked as primary. This does not meet our requirement of ensuring that at most one row per user is primary.

Trigger

A trigger can run custom logic when a database operation occurs (e.g., INSERT, UPDATE). While this gives us fine-grained control, it introduces complexity and potential performance overhead, especially in high-traffic scenarios. Triggers also add an extra layer of implementation that needs to be maintained.

Partial Unique Index

A partial unique index allows us to create a unique constraint on a subset of rows, based on the value of another column(s). In this case, we can create a unique index on is_primary = true combined with user_id. This ensures that only one row per user can have is_primary set to true.

#### Implementation

Here’s how you would implement this in SQL:

SQL
CREATE UNIQUE INDEX idx_user_is_primary ON your_table (user_id) WHERE is_primary = true;

This SQL statement creates a unique index on the user_id column, but only for rows where is_primary is true. This ensures that no two users can have more than one row with is_primary set to true.

Why It’s Better
Simplicity: The partial unique index is simpler and easier to maintain compared to a complex trigger logic.
Performance: Triggers can introduce significant overhead, especially in high-traffic systems. A partial unique index is generally faster because it does not require the execution of additional code paths.
Scalability: As the system scales, managing triggers becomes more challenging. Partial indexes scale better and are easier to manage.

#### Closing Summary

In summary, a partial unique index on user_id where is_primary = true ensures that only one row per user can have is_primary set to true. This approach is simpler, performs better, and scales well compared to using plain unique constraints or triggers.

#### Common Mistakes/Follow-Ups
Misunderstanding the Index Scope: Ensure you understand that a partial index applies only to specific conditions.
Performance Impact: Be aware of potential performance bottlenecks if the table is very large or frequently updated.
Maintenance Overhead: Triggers can introduce maintenance overhead, whereas indexes are generally easier to manage.

**Code:**

```sql
CREATE UNIQUE INDEX idx_user_is_primary ON your_table (user_id) WHERE is_primary = true;
```

