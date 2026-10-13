---
type: "practice"
difficulty: "Expert"
---

# Design a rollback-by-flag architecture where every schema change ships behind a compatibility toggle so reverting never requires a second risky DDL.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Schema Migration Strategy](<../../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/Schema_Migration_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to design a database migration strategy that allows us to roll back to a previous version of our database schema without necessitating a potentially risky Data Definition Language (DDL) operation. The approach involves using a compatibility toggle flag to control the application behavior and selectively apply changes.

#### Triaging the Question

The core of this question is about designing a rollback mechanism where every schema change is controlled by a flag, ensuring that reverting to an older version can be done without re-executing DDL commands. This architecture should handle both the implementation details and potential edge cases, such as handling concurrent operations and maintaining data integrity.

#### First-Principles Solution

To achieve this, we will implement two primary components:

Compatibility Toggle Flag: A logical flag in our application code that can be set to enable or disable a particular schema change.
Conditional Schema Application Logic: Ensuring the schema changes are only applied when the toggle is enabled and rolled back when disabled.
Step 1: Define the Compatibility Toggles

We need a mechanism to define which parts of the database schema should be conditionally applied. This can be implemented using a configuration file or an environment variable in the application code.

YAML
# Example configuration for compatibility toggles
compatibilityToggles:
  - id: "v2"
    description: "Version 2 schema change"
    appliesToTables: ["users", "orders"]
Step 2: Implement Conditional Schema Application Logic

The database migration scripts should be modified to include conditional logic based on the compatibility toggle flags. This can be done using a script that checks if the toggle is enabled before applying any DDL changes.

SQL
-- Example SQL with conditional schema change
IF (SELECT value FROM system.configuration WHERE key = 'compatibilityToggle_v2' = true) THEN
  ALTER TABLE users ADD COLUMN new_column;
ELSE
  -- Do nothing or revert the previous column addition if necessary
END IF;
Step 3: Handling Concurrent Operations

To handle concurrent operations, we need to ensure that the state of the database is consistent even when multiple requests are made. This can be achieved by using a locking mechanism or by implementing a retry policy.

SQL
-- Example of using locks
SELECT FOR UPDATE FROM system.configuration WHERE key = 'compatibilityToggle_v2';
Step 4: Rollback Strategy

When a rollback is needed, the application logic should be updated to disable the toggle and revert any changes made during that period. This can involve dropping columns or tables and reverting data back to previous states.

SQL
-- Example of rolling back a schema change
IF (SELECT value FROM system.configuration WHERE key = 'compatibilityToggle_v2' = false) THEN
  ALTER TABLE users DROP COLUMN new_column;
ELSE
  -- Do nothing as the toggle is already off
END IF;

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing a rollback-by-flag architecture, we ensure that schema changes are only applied when explicitly enabled. This approach significantly reduces the risk associated with DDL operations and allows for more controlled and predictable database migrations.

#### Common mistakes
include:

Failing to account for edge cases such as concurrent access.
Not properly handling data integrity during the rollbacks.
Ignoring the need for thorough testing of compatibility toggles in different environments.

#### Follow-up
questions might include discussing specific implementation details, trade-offs between this approach and traditional migration strategies, and best practices for managing configuration states across multiple applications or services.

**Comparison:**

```
Yes
No
Start
Check Compatibility Toggle
Toggle Enabled?
Apply Schema Changes
Do Nothing or Revert
Rollback if Necessary
End
```

**Code:**

```sql
-- Example SQL with conditional schema change
IF (SELECT value FROM system.configuration WHERE key = 'compatibilityToggle_v2' = true) THEN
  ALTER TABLE users ADD COLUMN new_column;
ELSE
  -- Do nothing or revert the previous column addition if necessary
END IF;

-- Example of rolling back a schema change
IF (SELECT value FROM system.configuration WHERE key = 'compatibilityToggle_v2' = false) THEN
  ALTER TABLE users DROP COLUMN new_column;
ELSE
  -- Do nothing as the toggle is already off
END IF;
```

**Example input / output:**

```text
-- Input (example usage of the toggle)
SET @toggle = (SELECT value FROM system.configuration WHERE key = 'compatibilityToggle_v2');

IF @toggle = true THEN
  -- Schema changes apply here
  ALTER TABLE users ADD COLUMN new_column;
ELSE
  -- No action or revert logic here
END IF;

-- Output (example of reverting the change)
SET @toggle = (SELECT value FROM system.configuration WHERE key = 'compatibilityToggle_v2');

IF @toggle = false THEN
  -- Revert the previous schema change
  ALTER TABLE users DROP COLUMN new_column;
ELSE
  -- No action as the toggle is still on
END IF;
```

**Data:**

```yaml
# Example configuration for compatibility toggles
compatibilityToggles:
  - id: "v2"
    description: "Version 2 schema change"
    appliesToTables: ["users", "orders"]
```

**Flow:**

1. Yes
2. No
3. Start
4. Check Compatibility Toggle
5. Toggle Enabled?
6. Apply Schema Changes
7. Do Nothing or Revert
8. Rollback if Necessary
9. End

