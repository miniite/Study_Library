---
type: "practice"
difficulty: "Intermediate"
---

# A production hotfix needs to go to main immediately but main is behind an in-progress big feature merge. How do you branch, fix, and land it safely without dragging the unfinished feature into prod?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Cherry-picking and Backports](<../../../../TOOLS/Git_and_GitHub/Git_Core/Cherry-picking_and_Backports.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When handling a situation where a hotfix needs to be deployed immediately, but the main branch is behind an in-progress merge of a significant feature, the key objective is to isolate and apply the fix while preserving the ongoing work on the feature. This requires careful branching, testing, and merging practices to ensure both stability and integrity.

The core challenge here is to create a safe and isolated environment for the hotfix without interfering with the main development process. Let's break down the steps needed to achieve this goal:

Understand the Problem: Main branch contains an in-progress feature merge, which means it may not be stable or fully tested.

#### Assumptions
The hotfix is critical and needs immediate deployment.
There are no conflicts between the hotfix and the existing codebase.

#### First-Principles Solution
Create a new feature branch from main.
Apply the hotfix to this feature branch.
Test the hotfix thoroughly.
Merge the hotfix into main, ensuring that it does not affect the ongoing feature work.

Here's how you would approach this:

Step-by-Step Solution

Create a New Feature Branch:

BASH
git checkout -b hotfix-branch-name origin/main

Apply the Hotfix:

Make sure to thoroughly understand and test the hotfix in an isolated environment (e.g., local development machine or a separate integration branch).

Test the Fix:

Run unit tests, integration tests, and any other relevant checks.
Ensure that the fix does not break existing functionality.

Commit and Push the Fix to GitHub:

BASH
git commit -am "Apply hotfix for critical issue"
git push origin hotfix-branch-name

Create a Pull Request (PR) for Main Branch:

Open a PR from hotfix-branch-name to main.
Ensure that the PR is thoroughly reviewed and tested before merging.

Merge the Fix into Main:

BASH
git checkout main
git merge hotfix-branch-name --no-commit  # This prevents automatic merge commit messages.
git commit -am "Merge hotfix from hotfix-branch-name"  # Manually add a descriptive message.
git push origin main

Verify the Fix in Production:

Deploy main to your staging environment and thoroughly test it before going live.

#### Closing Summary

By following these steps, you ensure that the hotfix is safely and independently applied without disrupting ongoing feature development. The key is to maintain clear separation between the hotfix branch and the main development flow, using Git branches and PRs as a means of governance and collaboration.

#### Common Mistakes

Not creating a dedicated hotfix branch from main can lead to confusion and potential merge conflicts.
Failing to thoroughly test the fix before merging it into main.
Merging directly without reviewing or testing the changes.

**Code:**

```bash
# Example commands for branching, committing, and pushing the hotfix
git checkout -b hotfix-branch-name origin/main
# Apply the necessary code changes
git commit -am "Apply hotfix for critical issue"
git push origin hotfix-branch-name

# Merge the fix into main
git checkout main
git merge hotfix-branch-name --no-commit
git commit -am "Merge hotfix from hotfix-branch-name"
git push origin main
```

**Flow:**

1. Yes
2. No
3. Create Hotfix Branch
4. Is the hotfix ready?
5. Apply and Test Fix
6. Merge into Main
7. Deploy to StagingEnvironment for Testing
8. Wait until fix is ready

