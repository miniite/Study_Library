---
type: "practice"
difficulty: "Deep"
---

# You need the DVC data version and the Git code version to move as one unit so a rollback restores both, but a teammate committed code without committing the updated .dvc file. How would you enforce that data and code pins always advance together in a single commit?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Versioning Data with DVC](<../../../../TOOLS/Git_and_GitHub/Git_Core/Versioning_Data_with_DVC.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that data and code versions move as one unit, with both being committed together during each change, we need to enforce strict adherence to Git best practices combined with DVC's capabilities. This involves setting up hooks in your repository to automatically check for uncommitted .dvc files before allowing a commit of the associated code changes.

Firstly, let’s summarize the key idea: We need to ensure that every commit contains both the updated code and its corresponding DVC data version, preventing any scenario where only one part is committed. This not only maintains consistency but also ensures that rollbacks are seamless, as all dependencies move together.

#### Triage

The question is essentially asking how to enforce a Git policy that requires both code changes and DVC updates to be committed simultaneously. This involves understanding the Git commit process, DVC’s role in managing data versions, and leveraging Git hooks to enforce these policies.

#### First-Principles Solution

Understand the Problem: The issue arises when a developer modifies code but does not update or commit the corresponding .dvc files. This can lead to inconsistencies where the latest code version depends on outdated data.

Identify Git Hooks: Git hooks are scripts that run automatically before and after specific Git events, such as pre-commit. We will use this feature to our advantage.

Create a Pre-Commit Hook:

Write a script that checks for uncommitted .dvc files when the pre-commit hook is triggered.
If any untracked .dvc files are found, reject the commit and inform the developer about the missing updates.

Implementing the Hook:

BASH
#!/bin/bash

# Check if there are changes to DVC files
git diff --cached --name-only | grep -q '\.dvc$'

if [ $? -eq 0 ]; then
  echo "Please commit all .dvc files before committing the code."
  exit 1
fi

# If no .dvc changes, proceed with normal commit process
git commit -m "$@"

Enforcing the Hook:

Place this script in your repository’s .git/hooks/pre-commit directory.
Make sure it is executable: chmod +x ./pre-commit.

Testing and Deployment:

Test by making a code change without updating any DVC files, which should result in an error message.
Verify that the hook works as expected before deploying it to other team members.

#### Closing Summary

By implementing a pre-commit Git hook, we ensure that both the code changes and their corresponding .dvc file updates are committed together. This prevents inconsistencies and ensures that rollbacks can be performed seamlessly by moving all dependent elements in unison.

#### Common Mistakes/Follow-Ups
Mistake 1: Forgetting to make the script executable (chmod +x). Ensure this step is not skipped.
Mistake 2: Not testing the hook thoroughly. Thorough testing with different scenarios ensures robustness.

#### Follow-Up
Consider additional hooks like pre-push if you want to enforce these rules even when pushing changes to remote repositories.

**Flow:**

1. Yes
2. No
3. Start
4. Is there a change to DVC?
5. Reject the commit
6. Commit the code changes
7. End with error message
8. Proceed with normal commitprocess

