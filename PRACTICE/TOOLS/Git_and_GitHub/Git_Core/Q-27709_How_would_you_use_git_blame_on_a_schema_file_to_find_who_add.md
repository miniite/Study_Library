---
type: "practice"
difficulty: "Intermediate"
---

# How would you use git blame on a schema file to find who added a NOT NULL constraint that is now causing insert failures?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Blame and Ignore Revisions](<../../../../TOOLS/Git_and_GitHub/Git_Core/Blame_and_Ignore_Revisions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to understand how git blame can be used effectively in conjunction with a database schema file to identify which developer introduced a specific change, such as adding a NOT NULL constraint that is now causing insert failures.

#### Triage
and Assumptions
Question: How would you use git blame on a schema file to find who added a NOT NULL constraint causing insert failures?
Key Concepts: Git commands, git blame, schema files, and the concept of NOT NULL constraints.

#### Assumptions
The database schema is version-controlled using Git.
There are multiple developers working on the project.
The most recent changes to the schema file have caused insert failures.
Step-by-Step Solution
Identify the Schema File: Locate the specific schema file where the NOT NULL constraint was added.
Run git blame: Use git blame <schema_file> to see the history of each line in the file, showing who last modified it and when.
Filter by Change: Identify lines that contain or are near the addition of a NOT NULL constraint using filters like grep.
Review Recent Changes: Focus on recent changes where new constraints were added.
Confirm with Logs: Check commit messages to understand context.

#### Example
Workflow

Locate the Schema File:

SH
cd /path/to/schema_directory
ls -la # List files in directory

Run git blame:

SH
git blame schema.sql --reverse

This command shows who last touched each line, with more recent changes appearing first.

Filter by Change: If you know the specific constraint added, use:

SH
git blame schema.sql -L 20-40 # For lines between 20 and 40 (adjust as needed)

Or if you suspect a recent change:

SH
git log --oneline --since="1 week ago" --grep="NOT NULL"

Review Recent Changes: Focus on the commits that add NOT NULL constraints:

SH
git show <commit_hash>

This will display the commit details, including the specific change.

Confirm with Logs: Check if there is any context in the commit messages that might explain why this was done:

SH
git log -p --grep="NOT NULL"

#### Closing Summary
and Common Mistakes/Follow-Ups
By using git blame, we can trace back to the specific commit and developer who added the constraint.

#### Common mistakes
include not filtering effectively or jumping straight to recent changes without context from logs.

#### Follow-ups
could involve:
Checking if this change was intentional or a mistake.
Determining if there are other constraints that should have been implemented earlier.
Ensuring proper testing and documentation before making such changes.

**Code:**

```
SH
Copy
git blame schema.sql --reverse
git log --oneline --since="1 week ago" --grep="NOT NULL"
git show <commit_hash>
git log -p --grep="NOT NULL"
```

