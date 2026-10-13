---
type: "practice"
difficulty: "Basic"
---

# How would you use git stash when you're mid-edit on an anomaly-detection notebook and suddenly need to review a teammate's urgent alert-rule PR?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Git Stash](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_Stash.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Git, git stash is a powerful command that allows you to temporarily save your work in progress so you can switch branches or pull in updates from others without losing your current changes. In this scenario, if you're mid-edit on an anomaly-detection notebook and suddenly need to review a teammate's urgent alert-rule PR, using git stash ensures you don't lose your progress.

To triage the question, we first need to understand that Git is a version control system used for managing code changes. The concept of git stash allows developers to save their work temporarily without committing it. This is particularly useful in this scenario where urgent tasks require immediate attention.

Let's break down the solution step by step:

Identify Your Current Working Directory: Confirm you are in the directory containing your anomaly-detection notebook.
Save Your Changes Temporarily:
SH
git stash
Switch to the Branch with the Urgent PR:
SH
git checkout <branch-name>
Fetch and Merge the Urgent PR:
SH
git pull origin <branch-name>
Review the Changes: Once you have reviewed the changes, you can return to your original branch.
Apply Your Stashed Changes:
SH
git stash pop

This approach ensures that your work on the anomaly-detection notebook is not lost and you can seamlessly switch back to it after reviewing the PR.

**Code:**

```
SH
Copy
git stash  # Save current changes temporarily

# Switch branches
git checkout <branch-name>

# Fetch and merge the urgent PR
git pull origin <branch-name>

# Review the changes, make any necessary adjustments

# Return to your original branch with your work still intact
git checkout <original-branch>
git stash pop  # Apply stashed changes back to current branch
```

**Flow:**

1. Start
2. Check current directory
3. Use `git stash` to save changes
4. Switch branches using 'gitcheckout <branch-name>'
5. Pull latest updates from PRbranch with 'git pull origin<branch-name>'
6. Review the merged PR
7. Return to original branch using'git checkout<original-branch>'
8. Apply stashed changes back tocurrent branch with `git stashpop`
9. Finish review and continuework

