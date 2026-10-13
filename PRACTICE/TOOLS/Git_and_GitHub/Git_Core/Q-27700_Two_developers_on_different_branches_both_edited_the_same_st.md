---
type: "practice"
difficulty: "Intermediate"
---

# Two developers on different branches both edited the same stored procedure file. Why does this cause a merge conflict, and how would you resolve it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Merge Conflicts](<../../../../TOOLS/Git_and_GitHub/Git_Core/Merge_Conflicts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When two developers work on the same file but are in different branches, any changes they make to that file can lead to merge conflicts when they try to integrate their code. This is because both developers' local versions of the file may have overlapping changes that GitHub cannot automatically resolve. In this scenario, the stored procedure file serves as the common ground where these developers are making modifications.

To understand why and how to resolve such a conflict, let's break down the problem step by step:

What Causes Merge Conflicts?
When two branches diverge significantly, especially if changes overlap in specific sections of code (like the stored procedure file), Git cannot automatically determine which change should take precedence.

#### Assumptions
Both developers are working on the same branch (main or develop).
One developer creates a new feature branch named feature-branch-A.
The other developer creates another feature branch named feature-branch-B.

#### First-Principles Solution
Step-by-Step Resolution

Identify the Conflict:

Both branches are merged back into the main branch, triggering a merge conflict because both have changes to the same file.

Examine Changes:

Review the diff of feature-branch-A and feature-branch-B against the common ancestor (usually main or develop) to understand what specific parts of the stored procedure were changed by each developer.

Merge Strategy:

Decide how you want to handle these changes. Typically, you can choose one version over the other manually.

Manual Resolution:

Open both versions in a code editor side-by-side or use a merge tool like GitKraken, Beyond Compare, or even built-in features of GitHub.
Carefully review and decide which change should be kept (or if parts can be combined).
After resolving the conflict, commit the merged changes to main.

#### Example
Workflow
BASH
# Fetch latest changes from remote repository
git fetch origin

# Merge branches into main branch
git checkout main
git merge feature-branch-A

If a conflict arises:

PLAINTEXT
Auto-merging stored_procedure.sql
CONFLICT (content): Merge conflict in stored_procedure.sql
Automatic merge failed; fix conflicts and then commit the result.
Manual Resolution

Open stored_procedure.sql and compare both versions of the file. For instance, if Developer A added a new clause to handle edge cases not handled by Developer B:

SQL
-- Developer A's changes (in feature-branch-A)
ALTER PROCEDURE my_stored_procedure
AS
BEGIN
    -- Existing code

    IF @input = 'edge_case'
    BEGIN
        -- Handle specific logic
    END

    -- End of procedure
END;

-- Developer B's changes (in feature-branch-B)
ALTER PROCEDURE my_stored_procedure
AS
BEGIN
    -- Existing code

    IF @input = 'another_edge_case'
    BEGIN
        -- Handle different logic
    END

    -- End of procedure
END;

You might resolve this by:

Combining both changes:

SQL
ALTER PROCEDURE my_stored_procedure
AS
BEGIN
    -- Existing code

    IF @input = 'edge_case'
    BEGIN
        -- Handle specific logic (Developer A's addition)
    END

    IF @input = 'another_edge_case'
    BEGIN
        -- Handle different logic (Developer B's addition)
    END

    -- End of procedure
END;

Choosing one version:

SQL
ALTER PROCEDURE my_stored_procedure
AS
BEGIN
    -- Existing code

    IF @input = 'edge_case'
    BEGIN
        -- Handle specific logic (Developer A's addition)
    END

    -- End of procedure
END;

After resolving the conflict, commit your changes:

BASH
git add stored_procedure.sql
git commit -m "Merge feature-branch-A and resolve merge conflicts"

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Closing Summary
Merge conflicts occur when branches have overlapping changes in the same file. To resolve them, manually review both versions, decide on a strategy (combining or choosing one version), and commit the result.

#### Common Mistakes

Failing to thoroughly understand both conflicting versions.
Not using merge tools to assist in resolving conflicts.
Committing untested code directly.

#### Follow-ups
might include:

Automating tests for stored procedures to catch issues early.
Using more advanced Git strategies like rebase before merging.

