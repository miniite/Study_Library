---
type: "practice"
difficulty: "Deep"
---

# A rebase went wrong halfway and the branch is in a weird half-applied state with the wrong commits. Walk through diagnosing the state and getting back to the exact pre-rebase HEAD using reflog and ORIG_HEAD.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Recovering Lost Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Recovering_Lost_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When rebasing goes awry, the first step is to understand the current state of your branch and how it relates to the original commit history. This involves diagnosing the situation by leveraging Git's reflog and ORIG_HEAD features. Let’s walk through a deep understanding of what went wrong, how to identify the issue, and finally, how to revert to the exact pre-rebase HEAD.

#### Triage
Understanding the Problem

When you encounter an issue during a rebase, it typically means that something unexpected happened halfway through the process, leaving your branch in a half-applied state. This state can be confusing because Git has already started moving commits but hasn’t fully committed the changes to the history yet. The key is to use git reflog and ORIG_HEAD to navigate back to the correct commit before any rebase modifications were applied.

Diagnosing the State

Identify Current HEAD: First, identify your current state by running:

SH
git status

This will give you an idea of which files are staged or uncommitted. However, it won’t show what changes have been partially applied during the rebase.

Use git reflog: The reflog command records every single action performed on a branch, including those that were later undone by operations like rebases. Run:

SH
git reflog

This will list all actions taken on your current branch, showing the commit hashes and dates. Look for entries related to the rebase process.

Locate ORIG_HEAD: The ORIG_HEAD is a special reference that points to where HEAD was before any changes were made by a rebase or cherry-pick operation. It helps in reverting to the state of the branch just before the problematic rebase started. You can use:

SH
git show ORIG_HEAD

This command will display the commit details at ORIG_HEAD, giving you a clear picture of what was there before any changes.

#### First-Principles Solution
Step-by-Step Process:

Check Reflog for Rebase Actions:

SH
git reflog

Find the entry that marks the beginning of your rebase process, which might look something like rebase 9b4e5c7 (the hash will vary).

Identify Problematic Commit: If you see commits being applied but not fully committed, identify the commit where things went wrong.

Reset to ORIG_HEAD:

SH
git reset --hard ORIG_HEAD

This command resets your branch to ORIG_HEAD, effectively undoing any partial or unapplied changes from the rebase process.

Clean Up Reflog: After correcting the state, you might want to clean up the reflog by removing old entries:

SH
git reflog expire --expire=now --all && git gc --prune=now

Verify State:

SH
git status

Ensure that your branch is back in a stable state, with no uncommitted changes or conflicts.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By following the steps above, you can diagnose and correct issues arising from a rebase that went wrong. The key is to use git reflog and ORIG_HEAD to navigate back to the pre-rebase state without losing any relevant changes. Here are some common mistakes to avoid:

Forgetting to Use --hard: When resetting, always use --hard unless you want to preserve unstaged changes.
Confusing Reflog with History: Remember that reflog is a history of your branch actions and not the actual commit history.

If you find yourself frequently running into issues like this, consider using git rebase -i (interactive mode) or creating backups before starting major rebase operations to minimize risks.

