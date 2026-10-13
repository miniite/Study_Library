---
type: "practice"
difficulty: "Deep"
---

# You want reproducible schema builds where the same Git commit always yields a bit-identical DDL bundle for deployment, but generated timestamps and comment headers break determinism. How would you use the commit SHA and normalize outputs to make builds traceable and stable?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Reproducible Builds from Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reproducible_Builds_from_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure reproducible schema builds in a Git-based environment, where each commit always yields bit-identical DDL (Data Definition Language) bundles, we need to address the non-deterministic aspects like generated timestamps and comment headers. The key is to leverage the commit SHA for traceability while ensuring stability through normalization techniques.

Triaging the Problem

The question revolves around making Git-based schema builds deterministic in terms of the output DDL files. This involves:

Identifying Non-Deterministic Elements: Timestamps, UUIDs, and auto-generated comments.
Using Commit SHA for Traceability: Ensuring that each commit is uniquely identifiable while maintaining reproducibility.
Normalization Techniques: Post-processing to remove or replace non-deterministic elements.

#### Assumptions
We are working with a Git repository where schema changes are committed and pushed regularly.
The DDL generation process uses some form of scripting, like SQL scripts or database migrations managed via tools like Liquibase or Flyway.
The goal is to ensure that the same commit always produces the exact same DDL output.

#### First Principles Solution
Step 1: Extracting Commit Information

Use Git hooks or a pre-commit tool to extract the commit SHA and other relevant metadata. For example, you can use git log commands to fetch details like author name, date, etc., but strip out non-deterministic elements.

BASH
# Example of extracting commit information using Git command
commit_sha=$(git rev-parse --short HEAD)
author_name=$(git log -1 --pretty=%an)
commit_date=$(git show -s --format=%cd --date=local)
Step 2: Normalizing DDL Output

Post-processing the generated DDL to remove or replace non-deterministic elements. This can be done using a scripting language like Python, Bash, or even SQL itself.

PYTHON
# Example of normalizing DDL with Python
import re

def normalize_ddl(ddl):
    # Strip out timestamps and comments
    ddl = re.sub(r'/\*.*?\*/', '', ddl)  # Remove multi-line comments
    ddl = re.sub(r'-- .*\n', '', ddl)   # Remove single-line comments
    return re.sub(r'\d{4}-\d\d-\d\d \d\d:\d\d:\d\d', '2023-12-31 23:59:59', ddl)  # Replace timestamps

# Example usage
with open('schema.sql', 'r') as file:
    raw_ddl = file.read()
normalized_ddl = normalize_ddl(raw_ddl)
Step 3: Embedding Traceability Metadata

Embed the commit SHA and other metadata into the DDL file in a deterministic way. This can be done by adding a header or footer to the DDL file.

SQL
-- Example of embedding metadata in SQL DDL
INSERT INTO schema_build_metadata (commit_sha, author_name, build_date)
VALUES ('${commit_sha}', '${author_name}', CURRENT_TIMESTAMP);

-- Normalized DDL content follows here...

#### Closing Summary
and Common Mistakes/Follow-Ups

To ensure reproducible schema builds:

Extract Commit SHA: Use Git hooks to capture the commit information.
Normalize Outputs: Post-process DDL files to remove non-deterministic elements like timestamps and comments.
Embed Metadata: Add deterministic metadata to the DDL file.

#### Common mistakes
include:

Failing to strip out all non-deterministic elements, leading to subtle differences in output.
Not using the commit SHA for traceability, leading to inconsistent builds across commits.
Overlooking edge cases where timestamps or other date-sensitive fields are used.

#### Follow-ups
might include discussing tools like Flyway or Liquibase, how they handle schema migrations, and best practices for managing database schema changes in a Git-based workflow.

**Code:**

```bash
# Example Bash script to extract commit information
commit_sha=$(git rev-parse --short HEAD)
author_name=$(git log -1 --pretty=%an)
commit_date=$(git show -s --format=%cd --date=local)

echo "Commit SHA: $commit_sha"
echo "Author Name: $author_name"
echo "Commit Date: $commit_date"

# Example Python script to normalize DDL
import re

def normalize_ddl(ddl):
    # Strip out timestamps and comments
    ddl = re.sub(r'/\*.*?\*/', '', ddl)
    ddl = re.sub(r'-- .*\n', '', ddl)
    return re.sub(r'\d{4}-\d\d-\d\d \d\d:\d\d:\d\d', '2023-12-31 23:59:59', ddl)

# Example usage
with open('schema.sql', 'r') as file:
    raw_ddl = file.read()
normalized_ddl = normalize_ddl(raw_ddl)
print(normalized_ddl)
```

