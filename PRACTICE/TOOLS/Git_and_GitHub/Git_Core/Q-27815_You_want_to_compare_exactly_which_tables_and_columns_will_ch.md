---
type: "practice"
difficulty: "Deep"
---

# You want to compare exactly which tables and columns will change if you merge a schema branch into main, before merging, to catch a risky ALTER. Explain using triple-dot diff, a DDL-diff tool, and a dry-run migration, and how each treats the merge-base.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Merge Strategies](<../../../../TOOLS/Git_and_GitHub/Git_Core/Merge_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When comparing branches in Git, especially database schema changes, it’s crucial to ensure that merging won't introduce unexpected alterations, such as risky ALTER statements. To achieve this, we can use a combination of triple-dot diffs (git diff --patience), DDL-diff tools like sqlalchemy-utils, and a dry-run migration process.

#### Triaging the Question

The question is asking how to compare schema changes between two branches—specifically, merging from a schema-branch into main. The goal is to identify exactly which tables and columns will change before performing the merge. This ensures that any risky alterations (like ALTER TABLE) can be caught early.

#### Assumptions
You are working with Git for version control.
The branches contain schema changes expressed in DDL statements.
A DDL-diff tool is available to compare these changes effectively.
Dry-run migrations can simulate the actual impact of the merge without altering any production data.
Step-by-Step Solution
1. Using Git Diff for Basic Comparison

First, we use Git's diff capabilities to get a basic sense of the changes between branches. The git diff command is used with appropriate flags to focus on DDL statements:

BASH
# Navigate to the repository root
cd path/to/repo

# Compare schema-branch against main using git diff
git diff main...schema-branch --patience --name-only | grep '\.sql$'

This command lists SQL files that have been modified, indicating potential schema changes.

2. Using a DDL-Diff Tool (e.g., sqlalchemy-utils)

Next, we use a DDL-diff tool to get a more precise comparison of the schema differences. For example, sqlalchemy-utils can be used to parse and compare SQL statements:

PYTHON
from sqlalchemy_utils import create_sqlite_database, drop_sqlite_database

# Set up the database connections
main_conn = create_sqlite_database("sqlite:///main.db", encoding="utf-8")
schema_branch_conn = create_sqlite_database("sqlite:///schema_branch.db", encoding="utf-8")

# Compare the schema
from sqlalchemy_utils import compare_sqlalchemy_databases

diff_result = compare_sqlalchemy_databases(main_conn, schema_branch_conn)

# Print out differences
for change in diff_result.changes:
    print(change)

This script creates temporary SQLite databases from each branch's schema and then compares them to identify any changes.

3. Dry-Run Migration

Finally, a dry-run migration can simulate the merge without altering actual data. This is particularly useful for catching issues before committing the merge. For instance, using pg_dump and psql in PostgreSQL:

BASH
# Create a backup of the current main database schema
pg_dump -U username --schema-only main > main_schema.sql

# Apply changes from schema-branch to a temporary environment
psql -U username -f schema_branch_schema.sql temp_main

# Compare the result with the original schema
diff <(cat main_schema.sql) <(cat temp_main_schema.sql)

This process helps ensure that any changes introduced by the merge are accurately reflected and do not introduce unexpected alterations.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By combining Git's diff capabilities, a DDL-diff tool, and a dry-run migration, we can effectively compare schema branches to catch risky ALTER statements early. Key takeaways include:

Use git diff --patience for initial change identification.
Employ a DDL-diff tool like sqlalchemy-utils for precise comparison.
Simulate the merge with a dry-run to ensure no unintended changes occur.

#### Common mistakes
include relying solely on Git diffs or skipping the dry-run process, which can lead to unexpected alterations post-merge. Always validate schema changes thoroughly before committing them.

**Comparison:**

```
Schema Changes	Description
ALTER TABLE users ADD COLUMN age INTEGER;	Adds a new column to the existing table.
```

**Code:**

```python
from sqlalchemy_utils import create_sqlite_database, drop_sqlite_database, compare_sqlalchemy_databases

# Set up database connections
main_conn = create_sqlite_database("sqlite:///main.db", encoding="utf-8")
schema_branch_conn = create_sqlite_database("sqlite:///schema_branch.db", encoding="utf-8")

# Compare the schema
diff_result = compare_sqlalchemy_databases(main_conn, schema_branch_conn)

# Print out differences
for change in diff_result.changes:
    print(change)
```

**Example input / output:**

```text
$ git diff main...schema-branch --patience --name-only | grep '\.sql$'
main_schema.sql

$ pg_dump -U username --schema-only main > main_schema.sql
$ psql -U username -f schema_branch_schema.sql temp_main

$ diff <(cat main_schema.sql) <(cat temp_main_schema.sql)
```

**Data:**

```sql
-- main_schema.sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL
);

-- schema_branch_schema.sql
ALTER TABLE users ADD COLUMN age INTEGER;
```

**Flow:**

1. Yes
2. Yes
3. Yes
4. No
5. No
6. Start
7. Use Git Diff for InitialComparison
8. Identify Changed SQL Files
9. Use DDL-Diff Tool for PreciseSchema Comparison
10. Run DDL-Diff Tool to GetDetailed Changes
11. Simulate Merge with Dry-RunMigration
12. Perform Dry-Run Migrationsand Validate Changes
13. Commit Safe Merge into MainBranch
14. Review Changed FilesManually

