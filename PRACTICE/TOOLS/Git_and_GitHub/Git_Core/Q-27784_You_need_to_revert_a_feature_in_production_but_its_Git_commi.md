---
type: "practice"
difficulty: "Expert"
---

# You need to revert a feature in production, but its Git commit bundled application code with an already-applied irreversible data migration. How do you reason about what git revert can and cannot undo here, and what compensating migration would you author?

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Reverts in GitOps Rollouts](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reverts_in_GitOps_Rollouts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle this scenario effectively, we need to understand Git's revert mechanism and recognize its limitations in dealing with irreversible data migrations. The goal is to rollback the feature without compromising the integrity of the database state.

TL;DR:
Understand the Problem: git revert can only roll back code changes but not data migrations.
Mitigate Data Loss Risk: Carefully assess and potentially reverse the migration using a compensating transaction.
Document and Test Changes: Ensure thorough testing before applying any compensatory migration.

#### Triage

The question is essentially asking how to handle an irreversible data state change (migration) that is bundled with code changes in Git, while ensuring the application's functionality can be restored without data loss or corruption. The key assumptions include that the feature was deployed and now needs to be reverted due to a bug or user feedback.

#### First-Principles Solution
Identify the Commit:
Identify the specific commit that introduced both the code changes and the irreversible migration.
Understand git revert Limitations:
git revert only works for code changes; it cannot undo data migrations or state changes.
Rollback Code Changes with git revert:
Use git revert <commit-hash> to revert the feature's code, creating a new commit that undoes those changes.
Step-by-Step Solution:

Identify and Revert Code Changes:

SH
git checkout production_branch
git revert -n <commit-hash>
# Review the diff and ensure it makes sense to undo these changes.
git commit -m "Revert 'Feature implementation X'"

Assess Data State:

Since git revert cannot fix data migrations, we need to look at the schema or model changes introduced by the migration.
Check if there are any compensating transactions or scripts that can be used to reverse the data state.

Develop a Compensatory Migration:

Write a new script or migration that reverts the changes made by the original migration.
Ensure this script is thoroughly tested in a staging environment before applying it in production.

Apply Compensatory Migration:

SH
# Example SQL to revert the data state change (assuming PostgreSQL)
psql -d database_name -c "ALTER TABLE table_name RENAME COLUMN old_column TO new_column;"

Test Thoroughly:

Run extensive tests on a staging environment to ensure that both the code and the data are in their correct states.
Verify application behavior after applying the compensatory migration.

Rollout Changes:

Once tested, apply the changes to production with minimal downtime if possible.
Monitor closely for any issues post-deployment.

#### Closing Summary
+ Common Mistakes/Follow-Ups:
Mistake 1: Assuming git revert can fix irreversible data state changes.
Mistake 2: Failing to test the compensatory migration thoroughly, which could lead to data loss or corruption.

#### Follow-Up
Discussing the use of database transactions and ACID properties in ensuring data integrity during migrations.

**Comparison:**

| Step | Action                 |
|------|------------------------|
| 1    | Identify and revert code changes with `git revert`         |
| 2    | Assess data state      |
| 3    | Develop a compensatory migration script                     |
| 4    | Apply the compensatory migration in a staging environment   |
| 5    | Test thoroughly        |
| 6    | Rollout to production carefully                            |

**Code:**

```
SH
Copy
# Example script for reverting code changes with git revert
git checkout production_branch
git revert -n <commit-hash>
git commit -m "Revert 'Feature implementation X'"
```

**Example input / output:**

```

Input:

SH
Copy
git checkout production_branch
git revert -n <commit-hash>

Output:

SH
Copy
[ ... diff output of reverted changes ... ]

Compensatory Migration Example Output:

SQL
Copy
psql -d database_name -c "ALTER TABLE table_name RENAME COLUMN old_column TO new_column;"
```

**Data:**

```sql
-- Example SQL to revert a migration (PostgreSQL)
ALTER TABLE table_name RENAME COLUMN old_column TO new_column;
```

**Flow:**

1. Identify and revert codechanges
2. Revert with `git revert`
3. Avoid data state issues bycompensating for migrations
4. Test thoroughly in staging
5. Rollout to production

