---
type: "practice"
difficulty: "Deep"
---

# A teammate force-pushed the shared migrations branch to remove a bad changeset, and now teammates whose local databases already applied that changeset are out of sync with Git. Explain the double failure (Git and DB) and how you recover both sides.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Reset vs Revert](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reset_vs_Revert.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we face a double challenge: a Git operation gone wrong in combination with an inconsistent database state. The force push to remove a changeset has left some teammates' local environments out of sync with both their remote repository and their databases. Let's break down the issues and discuss how to recover.

First, let's understand what happened:

Force Push: A teammate used git push --force to remove a specific changeset (migration) from the shared migrations branch.
Inconsistent Databases: Some local environments already applied this changeset, so their databases are now out of sync with both the Git history and any other teammates' unapplied states.

To recover, we need to address both the Git and database states simultaneously. Let's proceed step-by-step:

Understand the Current State:

Identify which migrations have been applied in each environment.
Determine if there are conflicting changesets or dependencies between them.

Revert the Force Push (Git):

Use git reflog to find a previous state before the force push and use git revert or git reset --hard <commit> to restore the branch to that state.
Ensure all teammates pull from the corrected remote repository.

Reconcile Database States:

For those who have applied the removed changeset, manually revert it if possible (e.g., by running a reverse migration).
If reverting is not feasible, you might need to perform an additional data cleanup step or use a database migration tool that supports rolling back specific migrations.

Ensure Consistency Across All Environments:

Use git status and git log to verify all changesets are correctly applied.
Apply any necessary manual steps to ensure the databases match the latest Git state.

Document the Incident:

Create a detailed incident report documenting what went wrong, how you recovered, and preventive measures for future occurrences.

**Comparison:**

| Commit Hash | Applied Migrations       |
|-------------|-------------------------|
| abcdef123456| [0001_initial, 0002_something] |
| ghi7890abcdef| [0001_initial, 0002_something] |

**Code:**

```
SH
Copy
# Example commands to recover from force push
git reflog  # List recent operations
git reset --hard <commit-hash>  # Hard reset to a previous state
git pull origin migrations-branch  # Pull the latest changes

# Reverting a migration in your local database (example using Django)
python manage.py migrate app_name <migration_number> --fake  # Fake apply or reverse migration
```

**Example input / output:**

```text
Initial State:
- Teammate A has applied migrations [0001_initial, 0002_something]
- Teammate B has applied migrations [0001_initial, 0002_something, 0003_new_feature]

Force Pushed to Remove 0003_new_feature

Recovered State:
- All teammates now have the following migration state: [0001_initial, 0002_something]
```

**Data:**

```yaml
current_branch_state:
  - commit_hash: abcdef123456
    applied_migrations: [0001_initial, 0002_something]
  - commit_hash: ghi7890abcdef
    applied_migrations: [0001_initial, 0002_something, 0003_new_feature]
```

**Flow:**

1. Identify Current State
2. Use Git Reflog
3. Find Previous Commit Hash
4. Hard Reset to Previous StateUsing 'git reset --hard'
5. Pull Latest Changes fromRemote Repository
6. Revert Applied MigrationsManually or via Tools
7. Verify All EnvironmentsSynchronized
8. Document Incident for FutureReference

