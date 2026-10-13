---
type: "practice"
difficulty: "Intermediate"
---

# How would you use branches so a log-parser change and an alert-threshold change can be reverted independently later?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Branching Strategies](<../../../../TOOLS/Git_and_GitHub/Git_Core/Branching_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To manage changes to a log-parser and alert thresholds independently, we need to leverage Git branches effectively. By creating separate branches for each feature, we ensure that if one needs reverting, the other can remain stable. Here’s how you would approach this:

TL;DR: Create two separate Git branches for the log-parser change and the alert-threshold change. Use a feature or release branch to merge these changes before deploying them.

#### Triage
The question is about using Git branches to manage independent features in version control, allowing us to revert specific changes if necessary without affecting others. Assumptions include that you are familiar with basic Git commands and branching strategies.

#### First-principles Solution

Create a New Branch for Each Feature:

SH
git checkout -b feature/log-parser-update master

This command creates a new branch named feature/log-parser-update from the master branch.

Make and Commit Changes to Log-Parser Branch:

SH
# Make changes in your log parser code
vim src/parser.js

# Add, commit, and push changes
git add .
git commit -m "Improve log parsing logic"
git push origin feature/log-parser-update

Create a New Branch for Alert-Threshold Change:

SH
git checkout -b feature/alert-threshold-update master

This command creates feature/alert-threshold-update from the master branch.

Make and Commit Changes to Alert Threshold Branch:

SH
# Make changes in your alert threshold configuration
vim config/alarms.yaml

# Add, commit, and push changes
git add .
git commit -m "Adjust alert thresholds for better performance"
git push origin feature/alert-threshold-update

Merge Changes into a Common Branch: After both features are tested and validated, merge the branches into a common branch (e.g., release or develop) to prepare for deployment.

SH
# Merge log-parser updates
git checkout develop
git merge --no-ff feature/log-parser-update

# Merge alert threshold updates
git checkout develop
git merge --no-ff feature/alert-threshold-update

Test Merged Changes: Ensure both features work as expected in a staging environment before deploying to production.

#### Closing Summary
and Common Mistakes: Using separate branches for independent changes allows you to manage each feature’s lifecycle independently. This approach makes it easier to revert or merge changes when needed. A common mistake is not creating distinct branches, leading to accidental overwriting of one change by another.

#### Common mistakes
include:

Not understanding the implications of rebasing vs merging.
Failing to follow a standardized branching strategy that aligns with team workflows and deployment processes.
Overcomplicating simple features without proper branch management.

**Code:**

```
SH
Copy
git checkout -b feature/log-parser-update master
git add .
git commit -m "Improve log parsing logic"
git push origin feature/log-parser-update

git checkout -b feature/alert-threshold-update master
git add .
git commit -m "Adjust alert thresholds for better performance"
git push origin feature/alert-threshold-update

git checkout develop
git merge --no-ff feature/log-parser-update
git merge --no-ff feature/alert-threshold-update
```

