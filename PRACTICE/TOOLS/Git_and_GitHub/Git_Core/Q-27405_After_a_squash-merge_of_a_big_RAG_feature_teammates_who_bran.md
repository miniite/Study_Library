---
type: "practice"
difficulty: "Deep"
---

# After a squash-merge of a big RAG feature, teammates who branched off the pre-squash branch keep re-introducing 'already merged' prompt commits. Explain the DAG reason and the workflow to reset their branches cleanly.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reverting Merges](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reverting_Merges.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Git and GitHub, squash-merges can lead to a confusing situation where teammates who branched off an older version of the repository introduce commits that were already merged by the squash-merge. This is due to how Git’s Directed Acyclic Graph (DAG) represents history. Let's break down why this happens and explain the best workflow to clean up these branches.

TL;DR

The reason teammates see "already merged" prompt commits after a squash-merge is because their local branch still points to an older commit that no longer exists in the merged state, causing Git to think those changes haven't been applied. The solution involves rebasing the branches onto the new merge commit and force-pushing them.

#### Triaging the Question

The question is essentially about understanding why teammates are seeing "already merged" commits after a squash-merge and how to properly sync their local repositories with the updated history. This problem arises because Git tracks branch histories as DAGs, where squashing merges can create historical forks that need to be reconciled.

#### Assumptions
The main repository has undergone a squash-merge.
Teammates have branched from an older version of the pre-squash commit.
These teammates are seeing "already merged" commits when trying to push changes.

#### First Principles Solution

Understanding the DAG:

In Git, every commit is a node in the DAG.
Squash-merges create new merge commits that consolidate multiple commits into one.
When teammates branch from an older version, their branches still reference those older nodes, even after the main branch has been updated.

Why the Problem Occurs:

The squash-merge commit (let’s call it S) effectively removes all intermediate commits, merging them into a single commit.
Teammates’ branches (T1, T2, etc.) might still contain these intermediate commits before they were squashed.

The Workflow to Resolve the Issue:

Ensure everyone is working from the latest main branch (post-squash merge).
Rebase each teammate's branch onto the new main branch.
Force-push the rebased branches if necessary, overwriting any local state.
Detailed Steps

Fetch Latest Changes:

BASH
git fetch origin

Rebase Each Branch: For a branch T1, run:

BASH
git checkout T1
git rebase main

This will bring the branch up to date with the latest commit, effectively squashing any intermediate commits.

Force-Push if Necessary: After rebasing, ensure teammates force-push their branches to overwrite local state:

BASH
git push --force-with-lease origin T1

#### Closing Summary
and Common Mistakes

The key here is understanding that Git tracks history in a DAG where squashing merges can create historical forks. Rebasining these branches ensures they are aligned with the new merge commit, avoiding "already merged" prompts.

#### Common Mistakes

Not Fetching Latest Changes: This can lead to conflicts or unexpected results.
Forgetting Force-Pushes: Overwriting local state is necessary but can be risky if not done correctly.

#### Example

Assume main has been updated with a squash merge from feature-branch at commit S. A teammate branched off the old main (B) and made changes, creating commit C1.

Fetch latest:

BASH
git fetch origin

Rebase branch T1 onto main:

BASH
git checkout T1
git rebase origin/main

Force-push to overwrite local state if needed:

BASH
git push --force-with-lease origin T1
Final Self-Check

The workflow described is both correct and complete, ensuring the branches are in sync with the updated history.

**Code:**

```bash
git fetch origin
git checkout T1
git rebase origin/main
git push --force-with-lease origin T1
```

