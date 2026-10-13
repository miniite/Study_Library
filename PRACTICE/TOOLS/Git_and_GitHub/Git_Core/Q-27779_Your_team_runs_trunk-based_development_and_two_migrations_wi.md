---
type: "practice"
difficulty: "Deep"
---

# Your team runs trunk-based development and two migrations with adjacent timestamps merged out of order, so Flyway applied them in a sequence that violated a foreign key on a busy table. How would you redesign the branching and migration-numbering scheme so ordering is deterministic regardless of merge order?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Automated Dependency Updates](<../../../../TOOLS/Git_and_GitHub/Git_Core/Automated_Dependency_Updates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this issue effectively, we need to ensure that migrations are always applied in a deterministic order, even when branches are merged out of order. The key here is to rethink our branching and migration numbering strategy.

TL;DR

We will redesign the branching model by using feature branches for each logical change and adopting a centralized trunk-based development flow with versioned migration numbers. This approach ensures migrations are applied in a predictable sequence, reducing conflicts like foreign key violations.

#### Triaging the Question

The question is essentially asking how to design a robust branching strategy that maintains deterministic ordering of database migrations, even when branches are merged out of order. The core of this problem lies in ensuring that migration numbers and application logic respect the dependencies between migrations. We will assume Flyway is being used as the migration tool, which follows a strict sequence for applying migrations.

#### Assumptions
The team uses Git for version control.
Migrations are managed with Flyway.
The current issue arises from merged branches where adjacent migrations have been applied out of order.

#### First-Principles Solution

Branching Strategy:

Use a single, centralized trunk branch (main or master) for production code and database schema versions.
Each developer works on feature branches based on this trunk branch. This ensures that all changes are isolated until they are ready to be merged back into the trunk.

Migration Numbering:

Assign a unique, monotonically increasing migration number to each migration. These numbers should follow a strict sequence and be applied in order.
For Flyway, this can be achieved by using version numbers that incrementally increase with each new migration file.

Merging Strategy:

When merging feature branches back into the trunk, ensure they are merged in the correct order relative to their last commit date or migration number.
Implement a pre-merge hook or script that checks if all migrations are up-to-date before allowing a merge. This can be done by comparing the branch's latest migration numbers with the trunk.

Migration Scripts:

Each migration file should have a --depends-on parameter to specify its dependencies on other migrations.
Ensure that the --run-sql and --rollback-script parameters are correctly set in each migration script to handle both forward and backward operations properly.
Detailed Implementation Steps

Feature Branches:

MARKDOWN
- Create a feature branch for each new development task or bug fix.

#### Example
BASH
git checkout -b feature/add-new-feature main

Migration Files Naming and Versioning:

Use Flyway's naming conventions to ensure version numbers are in the correct format (V1__description.sql).
Ensure each migration file has a unique, sequential version number.

Merging Strategy with Pre-merge Checks:

Implement a pre-merge script that checks if all migrations are up-to-date and applied in order.
BASH
# Example check script (pseudo-code)
for branch in feature_branches; do
  migration_versions=$(git log --oneline $branch | grep -oP '(?<=V\d+__).*(?=\.sql)')
  trunk_migration_versions=$(git log --oneline main | grep -oP '(?<=V\d+__).*(?=\.sql)')
  if [[ ! " ${trunk_migration_versions[@]} " =~ " ${migration_versions[@]} " ]]; then
    echo "Migrations are not up-to-date. Please update before merging."
    exit 1
  fi
done

Flyway Configuration:

Ensure Flyway configurations in flyway.conf or equivalent file allow for sequential application of migrations.
INI
flyway.sqlMigrationPrefix=V
flyway.sqlMigrationSeparator=__

#### Closing Summary
and Common Mistakes

By adopting a feature-branching model and ensuring that all migrations are properly versioned and ordered, we can mitigate the risk of foreign key violations during merge operations. Common mistakes include:

Forgetting to update migration dependencies.
Not implementing pre-merge checks for migration consistency.
Applying migrations out of order without proper validation.
Common Follow-Ups
How often do you run these checks in your CI/CD pipeline?
Do you have any specific tools or scripts that help automate this process?

**Comparison:**

| Migration | Description |
|-----------|-------------|
| V1        | Initial schema setup         |
| V2        | Add new column               |
| V3        | Rename table                 |

**Code:**

```
INI
Copy
# Example flyway.conf settings
flyway.sqlMigrationPrefix=V
flyway.sqlMigrationSeparator=__
```

**Example input / output:**

```
Input: Merging feature/new-feature into main.
Output: Pre-merge check fails because migration V2__add_new_column.sql is missing.
BASH
Copy
Migrations are not up-to-date. Please update before merging.
```

**Data:**

```json
{
  "migration_versions": [
    "V1__initial_schema.sql",
    "V2__add_new_column.sql",
    "V3__rename_table.sql"
  ]
}
```

**Flow:**

1. Pass
2. Fail
3. Feature Branch
4. Merge Request
5. Pre-merge Check
6. Merge into Trunk
7. Fix Migrations and RetryMerge
8. Trunk Main
9. Migrate to Latest Version

