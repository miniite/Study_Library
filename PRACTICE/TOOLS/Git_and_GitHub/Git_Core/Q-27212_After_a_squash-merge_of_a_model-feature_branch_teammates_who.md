---
type: "practice"
difficulty: "Deep"
---

# After a squash-merge of a model-feature branch, teammates who branched off the pre-squash work keep re-introducing 'already merged' commits into retrains. Explain the DAG reason and give the exact workflow to reset their branches cleanly onto main.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reverting Merges](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reverting_Merges.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're dealing with a common issue in Git where teammates who branched off code that was subsequently squashed and merged back into main can inadvertently reintroduce commits that have already been merged. The root cause lies in the Directed Acyclic Graph (DAG) structure of Git's commit history.

TL;DR

To reset branches to a clean state after a squash-merge, we need to ensure they are based on the latest main. This involves rebasing and force-pushing their local branches to override any outdated commits. The key is understanding how Git's DAG handles merged branches.

#### Triaging the Question

The core of this question revolves around understanding why teammates see "already merged" commits when retraining code, and what steps can be taken to ensure these branches are correctly aligned with main.

#### Assumptions

Teammates have already pulled the latest changes from main.
The squash merge was performed cleanly.
Commits that were part of the feature branch before squashing are now considered "merged" into main.

#### First Principles Solution

Understand the DAG Structure: In Git, a branch is essentially a pointer to a specific commit in the DAG. When you perform a squash-merge, main gets updated to point directly to the new merged commit, while the feature branch still points to its original commits.

Rebase Local Branches: To clean up branches that have diverged from main, each teammate needs to rebase their local branch onto the current state of main. This will replay their changes on top of the latest main commit.

Force-Push Changes: After rebasing, if teammates want to ensure they don't lose any work, they should force-push the updated branch back to remote. This overwrites the old history with the new one, ensuring everyone's local and remote branches are in sync.

Detailed Workflow

Ensure Up-to-Date Local Repository:

SH
git fetch origin main

Rebase Feature Branch onto Main:

SH
git checkout <feature_branch>
git rebase --onto main <start-point-of-feature-branch> HEAD

Here, main is the tip of the latest merged commit from the squash merge, and <start-point-of-feature-branch> is the first commit in your feature branch before squashing.

Force-Push to Remote:

SH
git push --force-with-lease origin <feature_branch>

The --force-with-lease flag prevents accidental loss of changes if someone else has pushed updates since you last fetched.

Repeat for All Affected Branches: If multiple branches need cleaning, repeat the above steps for each branch.

#### Closing Summary
and Common Mistakes

By following this workflow, teammates can reset their branches to accurately reflect the latest state of main. Key points include:

Always rebase onto the correct base commit.
Use force-push judiciously to avoid losing work or causing conflicts.
Communicate with team members before performing these operations.

#### Common mistakes
often involve not rebasing and force-pushing, leading to outdated commits. Another mistake is assuming that merging main into local branches will automatically clean up any divergences, which is not the case after a squash merge.

#### Example
Dry-Run

Let's say you have a feature branch feature-x that was squashed-merged into main. If your teammate still has their branch:

Fetch latest changes:

SH
git fetch origin main

Rebase the branch onto main:

SH
git checkout feature-x
git rebase --onto origin/main <start-commit-of-feature-x> HEAD

Force-push to remote:

SH
git push --force-with-lease origin feature-x

This ensures that the branch feature-x is correctly aligned with the latest main.

