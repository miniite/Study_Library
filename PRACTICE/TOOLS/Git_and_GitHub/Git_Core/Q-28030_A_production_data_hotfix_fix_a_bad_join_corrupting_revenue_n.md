---
type: "practice"
difficulty: "Deep"
---

# A production data hotfix (fix a bad join corrupting revenue numbers) must land on main immediately, but main is mid-way through a large lakehouse-migration refactor that isn't ready. How do you branch from the last good tag, land the fix, and avoid dragging the refactor into prod?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Cherry-picking and Backports](<../../../../TOOLS/Git_and_GitHub/Git_Core/Cherry-picking_and_Backports.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this issue, we need to ensure that our hotfix is applied in a way that minimizes disruption and maintains codebase integrity during an ongoing large refactor. Here’s how we can approach it:

TL;DR: We will branch from the last known good tag on main, apply the critical fix for the revenue join issue, and then rebase this fix onto main once the refactor is complete.

#### Triage

The question is asking how to manage a hotfix in Git/GitHub while avoiding interference with an ongoing large refactor.
We need to branch from a specific point (last_good_tag) on main, apply the fix, and then integrate this change into main once the refactor is complete.

Step-by-Step Solution:

Identify Last Known Good Tag: We start by finding the latest tag that represents a stable state before the current refactor began.
BASH
git fetch --tags
Create Hotfix Branch from Last Known Good Tag:
BASH
git checkout -b hotfix/revenue-join-fix last_good_tag
Apply Fix: Make the necessary changes in the branch to fix the revenue join issue.
Commit and Push: Stage, commit, and push the changes to GitHub.
BASH
git add .
git commit -m "Fix revenue join corruption"
git push origin hotfix/revenue-join-fix
Review Changes: Ensure that the fix does not introduce any unintended side effects or conflicts with other ongoing work.

Integrating Fix into Main:

Once the refactor is complete and main has been updated, we can merge our hotfix branch back to main.
BASH
git checkout main
git pull origin main  # Ensure local `main` is up-to-date
git merge --no-commit hotfix/revenue-join-fix
Resolve any conflicts that arise due to the changes in the refactor. This step is crucial because merging could introduce merge conflicts with the new code.
Cherry-Pick Fix: Instead of a full merge, we can cherry-pick the specific commit from hotfix/revenue-join-fix onto main.
BASH
git checkout main
git cherry-pick <commit-hash>
Test and Validate: Once integrated, thoroughly test the fix to ensure it works as expected in the production environment.
Push Changes:
BASH
git push origin main

#### Closing Summary
By branching from a stable tag and carefully integrating the fix into main once the refactor is complete, we can minimize risk and disruption during this complex merge scenario.

#### Common Mistakes/Follow-Ups

#### Common mistakes
include merging directly without cherry-picking, which could lead to unnecessary conflicts.
Another mistake is not thoroughly testing the merged changes in a staging environment before deploying them to production.

#### Follow-up
questions might involve discussing how to handle similar situations in larger organizations with multiple teams and dependencies.

**Code:**

```bash
# Example commands for branching, committing, and merging
git fetch --tags
git checkout -b hotfix/revenue-join-fix last_good_tag
git add .
git commit -m "Fix revenue join corruption"
git push origin hotfix/revenue-join-fix

git checkout main
git pull origin main  # Ensure local `main` is up-to-date
git merge --no-commit hotfix/revenue-join-fix

# Cherry-picking the specific fix
git checkout main
git cherry-pick <commit-hash>
```

**Example input / output:**

```text
# Example input to create a hotfix branch and push it
$ git fetch --tags
$ git checkout -b hotfix/revenue-join-fix v1.2.0  # Assuming the last good tag is v1.2.0

# Apply fix, commit, and push
$ git add .
$ git commit -m "Fix revenue join corruption"
$ git push origin hotfix/revenue-join-fix

# Example input to cherry-pick changes into main branch once refactor complete
$ git checkout main
$ git pull origin main  # Ensure local `main` is up-to-date
$ git merge --no-commit hotfix/revenue-join-fix
$ git commit -m "Merge fix from revenue-join-fix"
$ git push origin main

# Cherry-picking specific changes
$ git checkout main
$ git cherry-pick <commit-hash>
```

**Flow:**

1. Identify Last Known Good Tag
2. Fetch Tags
3. Create Hotfix Branch from Tag
4. Apply Fix and Commit
5. Push Changes to GitHub
6. Rebase on Main Once RefactorComplete
7. Cherry-Pick Specific Fixes intoMain
8. Test in Staging Environment
9. Publish to Production

