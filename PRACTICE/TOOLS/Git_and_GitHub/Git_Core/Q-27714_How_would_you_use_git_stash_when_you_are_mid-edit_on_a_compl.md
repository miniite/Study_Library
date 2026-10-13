---
type: "practice"
difficulty: "Intermediate"
---

# How would you use git stash when you are mid-edit on a complex query but suddenly need to switch branches to review a teammate's index change?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Git Stash](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_Stash.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you're working on a complex SQL query and suddenly need to switch branches to review a colleague’s indexing changes, git stash is an incredibly useful tool. It allows you to save your current work in progress without committing it or making any changes to your local branch, ensuring that your workspace remains clean for the task at hand.

#### Triage
What's Being Probed

The question aims to understand how you handle situations where you're mid-edit and need to switch branches, possibly temporarily. The interviewer is likely probing your understanding of Git workflow, branch management, and efficient use of Git commands in real-world scenarios.

#### Assumptions
You are currently on a master or feature branch.
You have open changes (uncommitted code) that you want to save without committing them.
You need to switch branches to review another colleague’s work before potentially resuming your current task.
Step-by-Step Solution from First Principles

Check Your Current Status: Before stashing, it's a good practice to check the status of your working directory and staging area:

SH
git status

Stash Your Changes:

If you have uncommitted changes in your working directory that you want to save temporarily, use git stash:
SH
git stash
This command will save your current work and return you to a clean working directory.

Switch Branches: Now that your local branch is clean, you can switch branches easily using:

SH
git checkout <branch_name>

Replace <branch_name> with the name of the branch where the indexing change was made by your colleague.

Review and Merge Changes:

Once on the target branch, review the changes.
If you need to integrate these changes into your current work, you can apply the stashed changes back using git stash pop:
SH
git stash pop
This command will reintroduce the changes and stage them for commit. Alternatively, if you just want to view the changes without staging them immediately, use git stash apply.

Complete Your Work: After reviewing the indexing change, continue working on your complex query.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, using git stash allows you to save your current work and switch branches seamlessly. This method ensures that your workspace remains clean for critical review tasks while preserving your progress. A common mistake is forgetting to apply or pop the stashed changes back when needed, which can lead to confusion about what you're working on.

For a fresher with intermediate experience, it’s crucial to understand these commands and their practical applications. Follow-ups might include discussing how to use git stash in more complex scenarios, such as when multiple branches are involved or when you need to preserve the stashed changes over longer periods.

**Code:**

```
SH
Copy
# Example Commands
git status
git stash
git checkout feature_branch_indexing_changes
```

