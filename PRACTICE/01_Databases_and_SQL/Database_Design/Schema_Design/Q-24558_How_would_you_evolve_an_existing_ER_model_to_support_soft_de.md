---
type: "practice"
difficulty: "Intermediate"
---

# How would you evolve an existing ER model to support soft deletes without breaking foreign-key referential integrity across dependent entities?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Polymorphic Associations](<../../../../01_Databases_and_SQL/Database_Design/Schema_Design/Polymorphic_Associations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To evolve an existing ER (Entity-Relationship) model to support soft deletes while maintaining foreign-key referential integrity, we need to carefully consider how to add a "soft delete" flag or timestamp to our tables and ensure that all related dependent entities remain consistent. Let's break down the process step-by-step.

Firstly, soft deletes involve adding a mechanism to mark records as deleted rather than physically removing them from the database. This approach is particularly useful in scenarios where historical data needs to be retained for compliance or audit purposes but isn't actively used.

#### Triaging the Question

The core of this question revolves around ensuring that:

The existing ER model supports soft deletes without breaking relationships.
Foreign keys and referential integrity are maintained across related tables.

We'll assume some standard conventions: primary keys, foreign keys, and a few common attributes like created_at and updated_at.

#### Assumptions
We have an existing database with multiple tables and established relationships.
The current schema does not support soft deletes.
Our goal is to add soft delete functionality without disrupting the integrity of the data.
Step-by-Step Solution
1. Identify Entities Requiring Soft Deletes

First, identify which entities need a soft delete feature. Typically, these are tables that hold historical or frequently queried data, such as Users, Orders, and Products.

2. Add a Soft Delete Column

For each entity requiring soft deletes, we'll add a new column to track the deletion status:

SQL
ALTER TABLE Users ADD COLUMN is_deleted BOOLEAN DEFAULT FALSE;

This adds a boolean flag that can be set to TRUE when a record should no longer be actively used.

3. Modify Foreign Keys and Relationships

For each table with foreign keys, we need to ensure that the soft delete column is respected in queries:

Check Constraints: Ensure constraints are updated or added to reflect the new logic.
Update Queries: Modify existing queries to account for the is_deleted flag.
4. Implement Soft Delete Logic

We'll implement methods in our application layer to handle soft deletes and restore records, ensuring referential integrity:

SQL
-- Example: Soft deleting a user
UPDATE Users SET is_deleted = TRUE WHERE id = 123;

-- Example: Restoring a user
UPDATE Users SET is_deleted = FALSE WHERE id = 456 AND is_deleted = TRUE;
5. Update Business Logic

Revise any business logic that interacts with the database to respect the is_deleted flag:

Query methods should filter out deleted records by default.
Create new methods like fetchActiveUsers() for specific use cases.

#### Closing Summary

By adding a soft delete column and carefully updating all related tables, we can ensure referential integrity is maintained. The key steps involve modifying existing data models and business logic to respect the new constraints without disrupting ongoing operations.

#### Common Mistakes/Follow-ups
Forgetting to update queries that rely on foreign keys.
Not considering edge cases where records need to be restored.
Ignoring performance implications of additional checks in queries.

**Code:**

```sql
-- Adding a soft delete column to the Users table
ALTER TABLE Users ADD COLUMN is_deleted BOOLEAN DEFAULT FALSE;
```

