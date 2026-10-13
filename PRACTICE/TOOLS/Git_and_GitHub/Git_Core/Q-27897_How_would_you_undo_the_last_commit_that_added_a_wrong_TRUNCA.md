---
type: "practice"
difficulty: "Basic"
---

# How would you undo the last commit that added a wrong TRUNCATE step to your load script, but keep the file so you can fix it, before pushing?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Undoing Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Undoing_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this issue, we need to understand how Git manages commits and how to revert specific changes without losing the file or its contents. When you added a wrong TRUNCATE step in your load script, it might have caused issues during data processing. Here’s how you can undo that last commit while keeping the file intact:

TL;DR

To undo the last commit that introduced an incorrect TRUNCATE step and keep the file for further modifications, follow these steps: identify the commit, reset your working directory, edit the script to fix the mistake, stage the changes, and then commit with a descriptive message.

#### Triaging the Question

The question is asking how to roll back the last commit that altered your load script inappropriately. Git provides various commands to handle such situations effectively. We need to:

Identify the commit hash.
Reset our working directory to an earlier state.
Stage and commit the corrected changes.

#### Assumptions
You are familiar with basic Git operations like git add, git commit, and git log.
The file containing the script is tracked in Git.
The last commit that introduced the incorrect TRUNCATE step can be identified by its hash or a message.
Step-by-Step Solution

Identify the Last Commit:

BASH
git log --oneline

This command will display the commit history in a concise format, showing the commit messages and hashes.

Reset Your Working Directory: Suppose the last commit hash is abc123. You can use git reset to undo this commit:

BASH
git reset abc123 --hard

This command will move your working directory back to the state of the specified commit, effectively removing any changes made after it.

Edit the Script: Open and edit the script in your text editor.

BASH
nano load_script.sql  # or whatever your file is named

Stage the Changes: After fixing the TRUNCATE step, stage the changes:

BASH
git add load_script.sql

Commit with a Descriptive Message: Commit the staged changes with a descriptive message to explain what was fixed.

BASH
git commit -m "Fixed incorrect TRUNCATE in data loading script"

#### Closing Summary
and Common Mistakes/Follow-Ups
Make sure you understand the importance of keeping your repository clean and organized. A well-maintained commit history helps in tracking changes and debugging issues.
If you accidentally use --hard with a wrong hash, it can result in lost data. Always ensure to double-check before executing destructive commands.

#### Common Mistakes
Using git reset --soft abc123 instead of --hard: This will only unstage changes but not revert them completely.
Not checking the commit history thoroughly: Make sure you know exactly which commit you are resetting to avoid accidental loss of work.
Additional Follow-Ups
What would you do if there were multiple commits since the last one?
How can you ensure this doesn’t happen in the future?

**Code:**

```bash
git log --oneline
git reset abc123 --hard
nano load_script.sql
git add load_script.sql
git commit -m "Fixed incorrect TRUNCATE in data loading script"
```

