---
type: "practice"
difficulty: "Deep"
---

# Two teammates each created a migration with the same sequence number on separate branches. How does this conflict surface in Git and how do you resolve it?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Merge Conflicts](<../../../../TOOLS/Git_and_GitHub/Git_Core/Merge_Conflicts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When two developers create database migrations with the same sequence number on separate branches, this can lead to a conflict in Git because the migration files are considered as having identical names and possibly similar content (since they both have the same sequence number). This conflict will surface during the merge process of these branches into a common branch like main or develop. To resolve it, we need to understand how Git handles conflicts and then devise a strategy that ensures our migrations are correctly applied.

#### Triaging the Question

What is being asked? The question is about identifying how such a conflict appears in Git and what steps you would take to handle and resolve it. The interviewer is probing your understanding of Git's branching strategies, merge conflicts, and best practices for version control in database migrations.

#### Assumptions

We are using the default Git flow where branches are created for feature development and merged into a main branch.
Migrations are stored as scripts with sequence numbers to ensure they can be applied in order.

#### First Principles Solution
Identifying the Conflict

When you try to merge feature-branch1 and feature-branch2, if both have migrations with the same sequence number, Git will detect this as a conflict. The merge tool (like git mergetool) will highlight these files for manual resolution because they are named identically.

Resolving the Conflict

Identify Merged Files:

Use git status to see which files have conflicts during the merge.

Review Migration Scripts:

Open the conflicting migration scripts and compare their content. Identify any differences or additional changes that need to be integrated.

Refactor Migrations (if necessary):

If one of the migrations is a more refined version, you can update both branches with the better version.
Alternatively, if they serve different purposes, you might need to rename or resequence them.

Apply and Test:

After resolving the conflict, apply the migrations in sequence on a test environment.
Ensure that no data loss or inconsistencies occur during this process.

Commit and Push:

Once tested, commit the changes and push the updated branches to GitHub.

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Closing Summary
Resolving conflicts in Git when dealing with migrations involves identifying which files are conflicting, reviewing their content, refactoring if necessary, applying them on a test environment, and then committing and pushing. It’s crucial to maintain the integrity of your database schema during this process.

#### Common Mistakes/Follow-Ups

Overwriting Important Changes: Always check that you don’t overwrite important changes made in one branch by accident.
Ignoring Sequencing: Ensure migrations are correctly sequenced, as out-of-order application can lead to database inconsistencies.
Manual Conflict Resolution: While Git can handle many conflicts automatically, manual resolution is often needed for more complex scenarios. Always test after resolving any conflicts.

Would you like to dive deeper into a specific part of the process or discuss how this might scale in a larger project?

**Code:**

```bash
# Example commands to resolve conflict
git merge feature-branch2
git mergetool  # Resolve conflicts using Git's tool
```

