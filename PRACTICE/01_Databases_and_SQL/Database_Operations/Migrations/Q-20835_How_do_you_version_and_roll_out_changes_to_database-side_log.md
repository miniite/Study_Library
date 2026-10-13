---
type: "practice"
difficulty: "Deep"
---

# How do you version and roll out changes to database-side logic (triggers, stored procs, RLS policies) with the same rigor and reversibility as table DDL?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Schema Migration Strategy](<../../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/Schema_Migration_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure rigorous and reversible updates to database-side logic such as triggers, stored procedures, and Row-Level Security (RLS) policies, we need a robust migration strategy akin to what we use for schema changes (table DDL). This process involves versioning each change, applying migrations systematically, and maintaining the ability to roll back or revert to any previous state. Let's break down this approach step by step.

**Comparison:**

| Migration ID | Name          | Description             |
|--------------|---------------|-------------------------|
| 001          | Initial Setup | Set up initial database structure |
| 002          | Create Triggers | Add triggers for data validation        |

**Code:**

```python
# Example of a simple migration script structure
class Migration:
    def __init__(self, name, description):
        self.name = name
        self.description = description

    def up(self):
        """Apply the migration."""
        pass

    def down(self):
        """Revert the migration."""
        pass

migrations = [
    Migration("001_initial_setup", "Set up initial database structure"),
    Migration("002_create_triggers", "Add triggers for data validation"),
    # Add more migrations as needed
]
```

**Example input / output:**

```text
# Example input and output for a migration process
Input: Apply migration 001
Output:
- Created table `users`
- Added trigger `validate_username` to validate usernames

Input: Rollback migration 002
Output:
- Removed trigger `validate_username`
```

**Data:**

```yaml
# Example of a migration file structure in YAML
migration-001:
  name: Initial Setup
  changes:
    - CREATE TABLE users (id INT, username VARCHAR(255));
    - CREATE TRIGGER validate_username BEFORE INSERT ON users FOR EACH ROW SET NEW.username = LOWER(NEW.username);
```

**Flow:**

1. Yes
2. No
3. Start Migration
4. Is migration reversible?
5. Apply migration
6. Verify application
7. Mark as applied
8. Handle non-reversible changesmanually
9. Rollback if needed
10. Revert to previous state

