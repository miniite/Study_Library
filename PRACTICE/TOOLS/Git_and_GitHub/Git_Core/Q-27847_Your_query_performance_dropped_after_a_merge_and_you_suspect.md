---
type: "practice"
difficulty: "Deep"
---

# Your query performance dropped after a merge and you suspect the wrong side of a conflict was taken in a join-rewrite. How do you diff the merge commit against both parents to audit the resolution and reproduce the intended query plan?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Merge Conflicts](<../../../../TOOLS/Git_and_GitHub/Git_Core/Merge_Conflicts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When faced with a performance drop after a merge, especially suspecting that the wrong side of a conflict was taken in a join-rewrite, the key is to carefully diff the merge commit against its parents. This process involves examining the changes made during the merge to identify which version of the conflicting part was chosen and how this affects query execution plans.

First, let’s break down what we need to do:

Identify the Merge Commit: Determine which commit hash corresponds to the problematic merge.
Get Parent Commits: Identify the two parent commits for that merge commit.
Diff Between Merge and Parents: Use git commands to compare changes between these commits.
Audit Changes in SQL Files: Focus on SQL files where the join-rewrite is applied.
Reproduce Query Plans: Run the queries before and after the merge change to understand their impact.

#### Triaging the Question

The core of this question revolves around understanding how merge conflicts are resolved, particularly in a complex query rewriting scenario. Git’s diff functionality will help us trace the changes made during the merge. We assume that:

The database schema or queries underwent significant changes.
A specific join operation was rewritten, which may have been conflictual.
The performance metric (e.g., execution time) has dropped after the merge.
Step-by-Step Solution

Identify the Merge Commit:

SH
git log --oneline --graph --decorate

This will show you a graph of commits, helping you identify the problematic merge commit by its hash (e.g., abc123).

Get Parent Commits: The parents of the merge commit can be obtained using:

SH
git cat-file -p abc123^
git cat-file -p abc123^1

Here, abc123^ and abc123^1 refer to the first parent and second parent of the merge commit, respectively.

Diff Between Merge and Parents: Use git diff to compare changes between the merge commit and its parents:

SH
git diff abc123~1...abc123

This command shows all changes introduced by this merge.

Audit Changes in SQL Files: Filter the diffs specifically for SQL files (assuming they are named with a .sql extension):

SH
git diff --name-only abc123~1...abc123 | grep -i '\.sql$'

This command lists the SQL files that were changed by the merge.

Reproduce Query Plans: Run the queries both before and after the merge change to observe their performance.

SH
# Before merge
mysql> EXPLAIN SELECT ... FROM table1 JOIN table2 ON ...;

# After merge (assuming the SQL file was updated)
git checkout abc123~1 -p <path/to/file.sql>
mysql> EXPLAIN SELECT ... FROM table1 JOIN table2 ON ...;

#### Closing Summary
and Common Mistakes/Follow-Ups

By carefully diffing the merge commit against its parents, we can audit which side of a conflict was taken during the join-rewrite. This process is crucial for understanding why query performance dropped. Common mistakes include not thoroughly diffing all relevant files or overlooking subtle changes in SQL logic that significantly impact execution plans.

For follow-ups, you might be asked to:

Explain how to automate this process.
Discuss potential issues with performance metrics.
Suggest ways to prevent such issues in the future.

**Code:**

```
SH
Copy
git log --oneline --graph --decorate
git cat-file -p abc123^
git cat-file -p abc123^1
git diff abc123~1...abc123 | grep -i '\.sql$'
mysql> EXPLAIN SELECT ... FROM table1 JOIN table2 ON ...
```

