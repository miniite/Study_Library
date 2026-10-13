---
type: "practice"
difficulty: "Basic"
---

# How would you amend your last commit to add a prompt file you forgot to stage, before you have pushed it?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Undoing Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Undoing_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To amend your last commit in Git and include a missed prompt file, follow these steps:

Understand the Problem: You need to modify your last committed state by adding a file that was not staged and then committing this change.
Restate the Question: The question is asking how to add a file (in this case, a prompt file) to the previous commit without altering any other files or changes made in that commit.
Step-by-Step Solution
Check Current State: Run git status to see which files are staged and unstaged.
Add Missed File to Last Commit: Use git add with the -p option (interactive mode) or directly specify the file path.
Amend the Last Commit: Use git commit --amend -m "Your commit message".
Detailed Steps
Step 1: Check Current State

Run the following command to see which files are in your staging area and untracked:

SH
git status

This will give you a list of files that have been staged for the next commit, as well as any files that are not being tracked.

Step 2: Add Missed File to Staging Area

If the file is already present in your working directory but was missed during staging, simply add it using:

SH
git add path/to/prompt

Alternatively, you can use interactive mode with git add -p:

SH
git add -p path/to/prompt

This will open an interactive prompt where you can choose which hunks of the file to stage.

Step 3: Amend the Last Commit

Once the missed file is staged, amend your last commit by running:

SH
git commit --amend -m "Add forgotten prompt file"

If you have already pushed the commit and need to force push (which should be done carefully), use:

SH
git push --force-with-lease

#### Common Mistakes
and Follow-Ups
Mistake: Forgetting to check git status before adding or amending a commit.
Mistake: Amending the wrong commit by mistake, which can lead to unnecessary conflicts with history.

#### Follow-Up
If you have already pushed your changes and need to force push, ensure you communicate this to collaborators to avoid conflicts.

#### Closing Summary

Amending your last commit in Git is a straightforward process once you understand the steps involved: check your current state, add any missed files, and then amend the commit message. Careful use of git commands ensures that your version history remains clean and accurate.

