---
type: "practice"
difficulty: "Basic"
---

# What does git stash -u do differently from a plain git stash?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Git Stash](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_Stash.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you use git stash -u, it adds all untracked files and changes to tracked files (including those not yet staged) to the stash, whereas git stash only stashes tracked files that are not added or committed. This distinction is crucial for handling different states of your working directory before switching contexts or making significant changes.

#### Triage

The question is asking about the differences between git stash -u and a plain git stash. The interviewer might be probing your understanding of how Git manages tracked versus untracked files, as well as the implications of using the -u flag. We need to consider what happens to both types of files when you use these commands.

#### First-Principles Solution
Tracked Files: These are files that have been added to the index and are part of your last commit.
Untracked Files: These are new files not yet added to Git or ignored by .gitignore.
Stashing Tracked Files (git stash): This command stashes only those tracked files which haven’t been committed (i.e., they’re staged but not part of the last commit).
Stashing All Changes with -u (git stash -u): This command includes both untracked and modified tracked files in the stash.
Detailed Explanation

git stash:

Only stashes changes to tracked files that are staged but not yet committed.
Unmodified, unstaged tracked files are left as they were.
Untracked files are ignored by default unless explicitly added with git add.

git stash -u:

Adds all modified tracked files (both staged and unstaged) to the stash.
Includes untracked files in the stash, essentially saving everything that has changed from your working directory.

#### Example Scenario

Suppose you have a project with the following state:

A file app.js is staged but not committed.
Another file config.json has changes but hasn’t been added to Git yet.
There are also some untracked files like notes.txt.

If you run git stash -u, it will save all these changes, including the unstaged changes in app.js, the modified but untracked config.json, and even the new notes.txt. If you just use git stash without -u, only the staged app.js would be saved.

#### Closing Summary

In summary, git stash -u is more comprehensive as it handles both tracked and untracked files. It's useful when you want to save your current state without worrying about which files are staged or untracked. A plain git stash, on the other hand, focuses only on changes that have been added but not yet committed.

#### Common Mistakes
Confusing git stash with git commit: Remember that stash does not commit changes; it saves them temporarily.
Overlooking untracked files: git stash -u can inadvertently save files you didn't mean to, so be mindful of what you're including in the stash.
Final Thoughts

Understanding these nuances is key for managing your work effectively and avoiding common pitfalls when using Git. Always consider the implications of your actions on tracked versus untracked files.

**Comparison:**

```
Command	What it Does
git stash -u	Stashes all untracked and modified tracked files
git stash	Stashes only staged but not committed changes in tracked files
```

**Code:**

```
SH
Copy
# Example usage
git stash -u  # Stashes all changes, both tracked and untracked
git stash    # Only stashes staged but not committed changes in tracked files
```

**Flow:**

1. No
2. Yes
3. Git Stash
4. Is there a -u flag?
5. Stashes only staged,unmodified tracked files
6. Stashes all modified anduntracked files

