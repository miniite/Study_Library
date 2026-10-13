---
type: "practice"
difficulty: "Basic"
---

# How would you use git checkout or git restore to discard your uncommitted, experimental changes to a query file and go back to the last commit?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Undoing Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Undoing_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this Git/GitHub scenario, we'll focus on using git checkout and git restore. The key is understanding that these commands help you revert changes in your working directory without affecting the version history of those files.

#### Triage

The question asks how to discard uncommitted experimental changes to a query file and return to the last commit. Here, we need to understand the differences between git checkout and git restore. git checkout is used when you want to switch branches or discard staged or working directory modifications. git restore is newer (introduced in Git 2.23) and helps restore files from a specific version or commit.

#### Solution
Identify the File: Determine which query file has uncommitted changes.
Stage Changes (if necessary): Use git add <filename> to stage your experimental changes before using git checkout or git restore.
Discard Uncommitted Changes with git checkout:
If you want to go back to the last commit and discard all uncommitted changes, use git checkout -- <filename>. This will reset the file in your working directory to match the last committed version.
Using git restore (Optional):
Alternatively, if you’re using Git 2.23 or later, you can use git restore <filename> to achieve similar results without staging. This command directly restores the file from a specific commit.

#### Closing Summary

In summary, both git checkout and git restore help you revert changes but differ in usage context. For a fresher, it's important to understand that git checkout -- <filename> is straightforward for discarding uncommitted changes and reverting to the last committed version. Experienced developers might opt for git restore depending on their Git version.

#### Common Mistakes
Misunderstanding the difference between git checkout and git reset.
Not using -- before <filename> when using git checkout, which can lead to unexpected behavior.
Assuming staged changes are automatically undone, requiring additional steps like git reset HEAD <filename>.

**Code:**

```bash
# Discarding uncommitted changes with git checkout
git add query.sql  # Stage the changes if necessary
git checkout -- query.sql

# Using git restore (Git 2.23+)
git restore query.sql
```

**Example input / output:**

```

Input:

PLAINTEXT
Copy
Current working directory has uncommitted changes to 'query.sql'.

Output:

PLAINTEXT
Copy
$ git add query.sql  # Stage the experimental changes (if necessary)
$ git checkout -- query.sql  # Discard uncommitted changes and revert to last commit version

OR for Git 2.23+:

$ git restore query.sql  # Directly restore from the last committed version
```

