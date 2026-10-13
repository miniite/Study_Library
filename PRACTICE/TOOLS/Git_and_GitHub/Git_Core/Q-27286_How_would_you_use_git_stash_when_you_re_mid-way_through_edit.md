---
type: "practice"
difficulty: "Basic"
---

# How would you use git stash when you're mid-way through editing a prompt and suddenly need to switch branches to review a teammate's PR?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Git Stash](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_Stash.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively manage your workflow when you are in the middle of editing a prompt and suddenly need to switch branches, git stash is an incredibly useful tool. Essentially, git stash saves your current work in a temporary area where it doesn't interfere with any other operations. This allows you to quickly switch contexts without losing progress.

#### Triage
Understanding the Problem

The question asks about using git stash when you're editing a file and need to temporarily switch branches to review another PR (pull request). The key here is to ensure that your ongoing work isn't lost, and you can seamlessly return to it later. The underlying assumption is that you are currently in the middle of making changes but want to pause those changes without committing them.

#### Solution
Using git stash
Step-by-Step Process
Save Your Work: Use git stash to save your current work and revert to a clean working directory.
Switch Branches: Once you have stashed, switch branches using the usual branch switching commands like git checkout <branch_name>.
Review PR: Review the teammate's pull request on the new branch.
Return to Your Original Work: After reviewing, return to your original work by applying the stashed changes.
Command Breakdown
To stash your current changes: Run git stash. This will save your local changes and bring you back to a clean state.
To verify that changes are stashed: Run git status. You should see that there are no uncommitted changes in your working directory.
If you want to inspect the stashed changes, use git stash list or git stash show -p. This will display the diff of the changes you've saved.

#### Example
Workflow
SH
# Assume you're currently on branch 'main' and editing a file called 'prompt.py'
git checkout main

# Make some edits to prompt.py

# Stash your current work:
git stash

# Switch branches to review the PR
git checkout another-branch

# Review the PR, make notes, etc.

# After reviewing and deciding you need to return to 'main', switch back
git checkout main

# Apply the stashed changes
git stash pop  # This will apply the changes from the stash and leave it on top of your current work.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Using git stash is a great way to manage small tasks that need to be temporarily paused. It’s important to remember that stashed changes can build up over time if you forget to apply them, so it's good practice to periodically run git stash list and consider applying or dropping old stashes.

#### Common Mistakes
Forgetting to use git stash pop before returning to your original branch, which might result in uncommitted changes on the new branch.
Not cleaning up stashed changes if they are no longer needed, leading to a cluttered stash history.
Self-CHECK

This answer covers the core usage of git stash, provides an example workflow, and highlights common pitfalls. It ensures completeness by addressing both the practical application and potential issues.

**Code:**

```
SH
Copy
# Example commands
git stash
git checkout another-branch
git stash pop
```

