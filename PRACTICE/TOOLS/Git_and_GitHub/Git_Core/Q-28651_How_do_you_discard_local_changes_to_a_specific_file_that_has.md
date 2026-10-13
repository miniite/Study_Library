---
type: "practice"
difficulty: "Basic"
---

# How do you discard local changes to a specific file that hasn't been committed?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Undoing Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Undoing_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Git, there might be instances where you need to undo or revert uncommitted changes in a specific file. For instance, if you've made some modifications but haven’t yet staged them (i.e., not added via git add), you can discard those changes without affecting other files. Let's walk through the process from first principles.

To triage the question: The core issue here is understanding how Git manages and tracks changes in files, specifically focusing on uncommitted local changes. We need to explore the workflow of Git commands that allow us to undo or revert these changes selectively.

Step-by-Step Solution

Identify Unstaged Changes: First, you need to check which files have unstaged changes using git status. This will show all the files with modifications but not yet added to the staging area.

Reset the Specific File: Once identified, use the git checkout command to discard the uncommitted changes in that specific file. The syntax is:

SH
git checkout -- path/to/your/file.txt

This command will restore the latest committed version of the file and revert any local modifications.

Verify the Changes: After running the above command, you can check git status again to confirm that your specific changes are no longer there.

#### Example

Let's assume we have a project with multiple files and we want to discard changes in app.js.

Check for unstaged changes:
SH
git status
Discard the uncommitted changes in app.js:
SH
git checkout -- app.js
Verify that the file is now clean of local modifications by checking git status again.

#### Closing Summary
and Common Mistakes

To summarize, if you need to discard specific uncommitted changes in a file, use git checkout -- path/to/your/file.txt. This command resets your working directory for just that one file. Be cautious with this as it will erase any local modifications, so ensure you really want to revert these before proceeding.

#### Common mistakes
include not running git status first to identify which files have unstaged changes or mistakenly using the wrong path. Always double-check paths and use the full path to avoid issues.

