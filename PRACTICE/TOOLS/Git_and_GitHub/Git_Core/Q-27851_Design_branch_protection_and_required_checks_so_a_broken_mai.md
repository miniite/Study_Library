---
type: "practice"
difficulty: "Expert"
---

# Design branch protection and required checks so a broken main (an un-appliable migration) is nearly impossible for the schema repo, while still allowing an audited break-glass override for an urgent production data-fix. How would you record and review those overrides?

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Branch Protection Design](<../../../../TOOLS/Git_and_GitHub/Git_Core/Branch_Protection_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to ensure that broken migrations in our main branch are nearly impossible by implementing robust Git and GitHub branch protection rules. At the same time, we must maintain an auditable process to handle urgent cases where a migration needs to be forced through despite it being un-appliable.

#### Triaging the Question

The core of this question is about designing a system that prevents accidental broken migrations from reaching the main branch while providing a controlled mechanism for handling emergency overrides. The key areas we need to cover are:

Branch Protection Rules: Ensure that broken migrations do not make it into the main branch.
Audit Logging and Override Mechanism: Document every forced override, making it clear who initiated the action and why.
Step-by-Step Solution

We'll start by setting up Git/GitHub rules to enforce strict checks on the main branch. Then, we’ll outline a process for handling urgent overrides with full audit trails.

1. Branch Protection Rules

To prevent broken migrations from reaching the main branch:

Enable Merge Checks: Require that all pull requests (PRs) be merged only after passing certain checks, such as static code analysis and migration validation.
Required Status Checks: Ensure that PRs must have passing status checks before they can be merged. This includes checks like pre-commit for code quality and a custom script to validate migrations.
Branch Protection
Bare-branch
M
Feature-branch
PR
2. Migration Validation Script

We need a robust script that validates migrations before they can be applied to the main branch. This script should check:

Migration Compatibility: Ensure the migration can still apply without errors.
Database Schema Integrity: Verify that applying the migration won't break existing data or introduce inconsistencies.
PYTHON
# Example validation script (pseudo-code)
def validate_migration(migration_file):
    try:
        # Run migration in a test environment
        execute_sql_script(migration_file, test_db_connection)
        return True
    except Exception as e:
        log.error(f"Migration failed: {e}")
        return False

# Function to execute SQL script
def execute_sql_script(script_path, db_connection):
    with open(script_path) as file:
        sql_commands = file.read()
    cursor = db_connection.cursor()
    try:
        cursor.execute(sql_commands)
        db_connection.commit()
    except Exception as e:
        log.error(f"Database error: {e}")
3. Override Mechanism

For urgent production data-fixes, we need a mechanism to bypass the usual checks temporarily but with strict auditing.

Staging Environment: Use a separate staging environment where the broken migration can be applied and validated before being rolled out to main.
Manual Approval Flow: Implement a process where someone with elevated permissions (e.g., DBA or Release Manager) must approve the override. This approval should be logged and recorded.
Override Mechanism
Bare-branch
M
Feature-branch
PR
Staging Branch
Production Override Request
Release Manager Approval
Merged to Main
4. Audit Logging and Reporting

To ensure transparency, we should maintain logs of all changes, including:

Pull Requests: Record who opened the PR, when it was merged, and what checks passed.
Override Approvals: Log who approved the override and why.
YAML
# Example log entry for a PR merge
{
    "action": "merge",
    "branch": "feature-x",
    "timestamp": "2023-10-05T14:30:00Z",
    "check-status": ["Passed Code Quality", "Passed Migration Validation"],
    "author": "alice"
}

# Example log entry for an override
{
    "action": "override",
    "branch": "staging-x",
    "timestamp": "2023-10-06T15:00:00Z",
    "approved-by": "bob",
    "reason": "Urgent production fix for critical data issue"
}

#### Closing Summary

By implementing these steps, we can ensure that broken migrations are nearly impossible to push through the main branch while still allowing controlled overrides in emergencies. This approach balances safety with flexibility.

#### Common Mistakes
and Follow-Ups
Overlooking Edge Cases: Ensure your validation script covers all possible scenarios where a migration might fail.
Insufficient Logging: Make sure logs capture enough detail for auditing purposes.
Inconsistent Approval Process: Have clear, documented procedures for overrides to avoid confusion or misuse.

**Comparison:**

| Action | Branch    | Timestamp           | Check Status              | Author   |
|--------|-----------|---------------------|--------------------------|----------|
| Merge  | feature-x | 2023-10-05T14:30:00Z | Passed Code Quality, ...  | alice    |
| Action | Branch     | Timestamp           | Approved By | Reason                                    |
|--------|------------|--------------------|-------------|------------------------------------------|
| Override | staging-x | 2023-10-06T15:00:00Z | bob         | Urgent production fix for critical data issue |

**Code:**

```python
# Example validation script (pseudo-code)
def validate_migration(migration_file):
    try:
        # Run migration in a test environment
        execute_sql_script(migration_file, test_db_connection)
        return True
    except Exception as e:
        log.error(f"Migration failed: {e}")
        return False

# Function to execute SQL script
def execute_sql_script(script_path, db_connection):
    with open(script_path) as file:
        sql_commands = file.read()
    cursor = db_connection.cursor()
    try:
        cursor.execute(sql_commands)
        db_connection.commit()
    except Exception as e:
        log.error(f"Database error: {e}")
```

**Example input / output:**

```text
{
  "action": "merge",
  "branch": "feature-x",
  "timestamp": "2023-10-05T14:30:00Z",
  "check-status": ["Passed Code Quality", "Passed Migration Validation"],
  "author": "alice"
}
```

**Data:**

```yaml
# Example log entry for a PR merge
{
    "action": "merge",
    "branch": "feature-x",
    "timestamp": "2023-10-05T14:30:00Z",
    "check-status": ["Passed Code Quality", "Passed Migration Validation"],
    "author": "alice"
}

# Example log entry for an override
{
    "action": "override",
    "branch": "staging-x",
    "timestamp": "2023-10-06T15:00:00Z",
    "approved-by": "bob",
    "reason": "Urgent production fix for critical data issue"
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph LR;
3. subgraph "Branch Protection"
4. M [Main Branch]
5. B1[Bare-branch] --> M
6. B2[Feature-branch] --> PR --> M
7. end
8. subgraph "Override Mechanism"
9. M [Main Branch]
10. B1[Bare-branch] --> M
11. B2[Feature-branch] --> PR --> M
12. O1[Staging Branch] --> B3[Production Override Request]
13. R1[Release Manager Approval] --> B4[Merged to Main]
14. end
15. subgraph "Audit Logging"
16. L [Log Entry] --> M
17. L --> B2
18. L --> B4
19. end

