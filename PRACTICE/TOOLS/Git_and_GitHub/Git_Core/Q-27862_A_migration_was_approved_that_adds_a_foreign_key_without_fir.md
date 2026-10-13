---
type: "practice"
difficulty: "Deep"
---

# A migration was approved that adds a foreign key without first indexing the referencing column, which will slow every write. What review checklist and automated CI rule would you put in place to catch missing-supporting-index patterns on FK migrations?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Automated Dependency Updates](<../../../../TOOLS/Git_and_GitHub/Git_Core/Automated_Dependency_Updates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of adding foreign keys without supporting indexes that can significantly slow down writes, it's crucial to establish a robust review checklist and automated CI rule. This ensures that every migration is thoroughly vetted for necessary indexes before deployment.

#### Triage

The core question is about creating a comprehensive process to ensure that when a foreign key (FK) is added, an accompanying index on the referencing column is also created. The goal is to prevent performance bottlenecks during write operations and maintain database integrity.

#### Assumptions
The database system supports the creation of indexes.
Migrations are managed through Git and GitHub workflows.
There is a CI/CD pipeline in place for deploying changes.

#### First Principles Solution
Pre-Migration Review Checklist:
Verify that the referencing column(s) have appropriate indexing.
Ensure that any necessary indexes are present before adding the FK constraint.
Automated CI Rule Implementation:
Use a pre-commit hook to run tests locally.
Integrate these checks into the CI pipeline.
Step-by-Step Solution
Pre-Migration Review Checklist
Code Review:
Ensure that the migration script explicitly adds an index on the referencing column before adding the FK constraint.
SQL
-- Example SQL for creating a unique index and then adding the FK
CREATE UNIQUE INDEX idx_referencing_column ON referenced_table (referencing_column);

ALTER TABLE target_table ADD CONSTRAINT fk_target_referenced FOREIGN KEY (target_column) REFERENCES referenced_table(referencing_column);
Manual Testing:
Manually verify that the index exists and is functioning as expected.
Peer Review:
Have another developer review the migration script for any potential issues.
Automated CI Rule Implementation
Pre-Commit Hook:
BASH
# Example pre-commit hook in a `.git/hooks/pre-commit` file
echo "Running pre-commit checks..."

if ! db_migration_index_check; then
    echo "Index check failed! Please add the necessary indexes before committing."
    exit 1
fi

echo "Pre-commit checks passed!"
Script for Index Check:
BASH
# Example script to ensure indexes are present
db_migration_index_check() {
    tables=$(grep -oP '(?<=ALTER TABLE \K[^ ]+)' migration_file.sql)

    for table in $tables; do
        referencing_columns=$(grep -oP "(?<=FOREIGN KEY .*\()[^ )]+" "$table".sql)

        for column in $referencing_columns; do
            if ! index_exists "$table" "$column"; then
                echo "Missing index on table '$table' and column '$column'"
                return 1
            fi
        done
    done

    return 0
}

index_exists() {
    sqlite3 -header -csv <(sqlite3 database.db ".tables") | grep -w "$1"
    if [ $? -ne 0 ]; then
        return 1
    fi

    sqlite3 -header -csv <(sqlite3 database.db "PRAGMA index_list($1)" | tail -n +2) | awk -F, '{print $2}' | grep -w "$2"
    if [ $? -ne 0 ]; then
        return 1
    fi

    echo "Index exists on table '$1' and column '$2'"
    return 0
}
CI Pipeline Integration:
Add steps to the CI pipeline that run similar checks as part of the pre-commit hook.
Ensure these checks are also performed when deploying changes.

#### Closing Summary

By implementing a robust review checklist and automated CI rule, we can ensure that foreign key migrations always come with necessary supporting indexes. This approach helps prevent performance issues and maintains database integrity.

#### Common Mistakes/Follow-Ups

Mistake: Skipping the pre-migration index checks.

#### Follow-up
Ensure these steps are documented in the code review guidelines.

Mistake: Failing to update the CI pipeline with equivalent checks.

#### Follow-up
Make sure all migration scripts pass through both local and CI checks.

**Code:**

```
Example pre-commit hook script

echo "Running pre-commit checks..."

if ! db_migration_index_check; then echo "Index check failed! Please add the necessary indexes before committing." exit 1 fi

echo "Pre-commit checks passed!"

CODE
Copy

```bash
# Example index check script
db_migration_index_check() {
    tables=$(grep -oP '(?<=ALTER TABLE \K[^ ]+)' migration_file.sql)

    for table in $tables; do
        referencing_columns=$(grep -oP "(?<=FOREIGN KEY .*\()[^ )]+" "$table".sql)

        for column in $referencing_columns; do
            if ! index_exists "$table" "$column"; then
                echo "Missing index on table '$table' and column '$column'"
                return 1
            fi
        done
    done

    return 0
}

index_exists() {
    sqlite3 -header -csv <(sqlite3 database.db ".tables") | grep -w "$1"
    if [ $? -ne 0 ]; then
        return 1
    fi

    sqlite3 -header -csv <(sqlite3 database.db "PRAGMA index_list($1)" | tail -n +2) | awk -F, '{print $2}' | grep -w "$2"
    if [ $? -ne 0 ]; then
        return 1
    fi

    echo "Index exists on table '$1' and column '$2'"
    return 0
}
```

**Flow:**

1. Missing index
2. All good
3. Missing index
4. All good
5. Pre-Commit Hook
6. Check Indexes
7. Error: Add necessary indexes
8. Commit Check Passed
9. CI Pipeline
10. Run Index Checks
11. Error: Fix and Re-run CI
12. Pipeline Passes

