---
type: "practice"
difficulty: "Deep"
---

# How would you enforce that only one row can be marked as the default/primary per user using a partial unique index?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Adding Constraints Online](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Adding_Constraints_Online.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that only one row can be marked as the primary/default per user in PostgreSQL, we need to design a mechanism that restricts multiple rows from having the same is_default flag set to true. We'll use a partial unique index on the combination of user_id and is_default columns to enforce this constraint.

#### Triaging the Question

The core question is about enforcing data integrity rules in PostgreSQL using a specific type of index—partial unique indexes. The interviewer might be probing your understanding of:

How to use constraints effectively.
Your ability to design database schemas that meet specific requirements.
Knowledge of PostgreSQL's indexing capabilities and how they can be used for constraint enforcement.

#### Assumptions
We have a table called user_preferences with columns: id, user_id, preference_key, and is_default.
The is_default column is boolean and represents whether the preference is marked as default.
Each user can have multiple preferences, but only one should be marked as default.

#### First-Principles Solution
Step 1: Define the Table Schema

First, let's define the table schema with a partial unique index on user_id and is_default.

SQL
CREATE TABLE user_preferences (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    preference_key TEXT NOT NULL,
    is_default BOOLEAN DEFAULT FALSE,
    UNIQUE (user_id, is_default)
);
Step 2: Explanation
The UNIQUE constraint ensures that no two rows can have the same combination of user_id and is_default.
By setting is_default to FALSE as the default value, we ensure that by default, a preference is not marked as default.
Step 3: Enforcing the Constraint

When you insert or update a row, PostgreSQL will check if there already exists another row with the same user_id and is_default combination. If it does, the operation will fail because of the unique constraint.

SQL
INSERT INTO user_preferences (user_id, preference_key, is_default)
VALUES (1, 'theme', TRUE); -- This would succeed

UPDATE user_preferences SET is_default = TRUE WHERE id = 1; -- This would fail if there's already a default preference for this user.
Step 4: Handling Multiple Defaults

If you need to support the scenario where multiple preferences can be marked as default but only one at any given time, you could consider using a different approach such as a trigger or a check constraint. However, PostgreSQL does not natively support disabling a unique constraint temporarily in transactions.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, by leveraging a partial unique index on user_id and is_default, we can ensure that only one row per user is marked as the default preference. This approach requires careful consideration of how you handle data insertion and updates to maintain consistency.

#### Common mistakes
include:

Forgetting to set a default value for is_default.
Not understanding the implications of partial unique indexes.
Ignoring transactional integrity when dealing with multiple updates or concurrent transactions.

#### Follow-up
questions might include:

How would you handle scenarios where a row needs to be updated from non-default to default?
What are the performance implications of this approach?

**Code:**

```sql
-- Creating the table and index
CREATE TABLE user_preferences (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    preference_key TEXT NOT NULL,
    is_default BOOLEAN DEFAULT FALSE,
    UNIQUE (user_id, is_default)
);
```

**Example input / output:**

```text
-- Successful insertion
INSERT INTO user_preferences (user_id, preference_key, is_default) VALUES (1, 'theme', TRUE);

-- Failed update due to constraint violation
UPDATE user_preferences SET is_default = TRUE WHERE id = 1;
```

