---
type: "practice"
difficulty: "Basic"
---

# What does 'This branch is out of date with the base branch' mean on a PR and how do you fix it?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Rejected Pushes](<../../../../TOOLS/Git_and_GitHub/Git_Core/Rejected_Pushes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you see "This branch is out of date with the base branch" in a Pull Request (PR) on GitHub, it means that there have been commits made to the base branch since you last fetched or pulled from it. To fix this issue, you need to update your local branch so it's up-to-date before you push any new changes.

To understand and solve this problem, let's break it down step by step:

TL;DR
"This branch is out of date" means there are newer commits in the base branch that aren't present in your current branch.
To fix it, you should fetch or pull updates from the base branch, rebase your changes if necessary, and then push them to the remote repository.

#### Triaging the Question

The question is asking about a common Git workflow issue encountered when working with Pull Requests on GitHub. The interviewer might be probing your understanding of:

Basic Git commands like git fetch, git pull, and git rebase.
How branching and merging work in Git.
Best practices for keeping branches up-to-date.

#### Assumptions
You have a local branch that you are working on, which is forked from an upstream base branch (e.g., main or develop).
The base branch has received new commits since your last interaction with it.
Step-by-Step Solution

Fetch the Latest Changes:

SH
git fetch origin

This command updates your local copy of the remote repository without merging changes into your current working branch.

Check for Unmerged Changes: Run git status to see if there are any uncommitted changes that need to be resolved before you update your branch.

Merge or Rebase:

If you want to integrate the latest changes from the base branch into your current branch, use:
SH
git merge origin/<base_branch>
Alternatively, if you prefer a linear history without merge commits, rebase your branch onto the updated base branch:
SH
git rebase origin/<base_branch>

Resolve Any Conflicts: If there are conflicts during the merge or rebase process, Git will pause and require you to resolve these manually before continuing.

Push the Updated Branch: Once your branch is up-to-date and any conflicts are resolved, push it back to the remote repository:

SH
git push origin <your_branch>

Create or Update PR: After pushing, go to GitHub and create a new Pull Request if you haven't already, or update an existing one.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Summary: By following these steps, you can resolve the "out of date" issue in your branch and ensure that it is ready for review via a PR.

#### Common Mistakes
Not running git fetch or git pull before trying to push changes.
Forgetting to address conflicts during merge or rebase operations.

#### Follow-Ups

The interviewer might ask:

How do you handle large codebases with multiple developers working on the same branch?
What are some best practices for maintaining a clean Git history?

