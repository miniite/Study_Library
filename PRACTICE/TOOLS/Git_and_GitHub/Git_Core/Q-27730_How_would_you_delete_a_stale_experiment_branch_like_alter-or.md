---
type: "practice"
difficulty: "Basic"
---

# How would you delete a stale experiment branch like alter-orders-v2 both locally and on GitHub after its migration was merged?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Recovering Lost Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Recovering_Lost_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To remove a stale experiment branch, such as alter-orders-v2, both from your local repository and on GitHub, you need to follow a structured process to ensure that no data or code is lost during the cleanup.

#### Triage
of the Question

The question is asking how to clean up a Git branch named alter-orders-v2 after it has been migrated and merged. The key points to address are:

Branch exists locally but not needed anymore.
Clean up both local repository and remote GitHub repository.

#### Assumptions
You have already confirmed that the alter-orders-v2 branch is no longer required and can be safely deleted.
You are using a Git client like git from the command line or an integrated development environment (IDE) with Git integration.
Step-by-Step Solution

Remove Local Branch:

First, check if you have any uncommitted changes in your local branch that need to be committed or stashed.
If there are no uncommitted changes, you can delete the local branch using:
SH
git branch -d alter-orders-v2
If the branch is not fully merged and has been fast-forwarded into a different branch (say main), use:
SH
git branch -D alter-orders-v2

Remove Remote Branch:

Once your local branch is deleted, you need to remove it from GitHub as well.
Use the following command to delete the remote branch:
SH
git push origin --delete alter-orders-v2
Alternatively, you can use the GitHub web interface to manually delete the branch.

#### Closing Summary
and Common Mistakes
Common Mistake: Not checking for any uncommitted changes before deleting a local branch.
Step-by-Step Solution Recap:
Check for uncommitted changes.
Delete local branch using git branch -d or -D.
Remove remote branch using git push origin --delete <branch-name>.
Depth and Language Matching the Level

As a fresher, it's crucial to understand the basic commands and their usage in Git. The steps provided are straightforward but cover all necessary actions to safely remove a stale branch.

**Example input / output:**

```text
$ git status
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean

$ git branch -d alter-orders-v2
error: The branch 'alter-orders-v2' was found but does not appear to be a descendant of your current branch.
If you are sure you want to delete it, use 'git branch -D alter-orders-v2'

# Delete local branch using -D as the branch is fast-forwarded
$ git branch -D alter-orders-v2

# Push remote branch deletion
$ git push origin --delete alter-orders-v2
```

