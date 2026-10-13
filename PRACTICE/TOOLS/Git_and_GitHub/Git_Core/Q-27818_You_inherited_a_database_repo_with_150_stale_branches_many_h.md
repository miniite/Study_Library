---
type: "practice"
difficulty: "Intermediate"
---

# You inherited a database repo with 150 stale branches, many holding half-finished migrations that reuse the same version numbers. How would you audit them, decide what to prune safely, and automate branch hygiene so version collisions stop recurring?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Automated Dependency Updates](<../../../../TOOLS/Git_and_GitHub/Git_Core/Automated_Dependency_Updates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When inheriting a database repo with 150 stale branches holding half-finished migrations, the goal is to clean up without disrupting ongoing work while ensuring future changes don't suffer from version number conflicts. Here's how we can approach this:

Firstly, let's understand what needs to be done:

Audit all branches for completeness and relevance.
Identify branches that should be pruned or merged.
Automate a process to prevent future version collisions.

Now, step-by-step, let’s address each of these points:

Audit the Branches:
We need to review each branch to understand its state. This involves checking commit messages and contents for any active development.
Identify Prune Candidates:
Determine branches that are no longer needed or have diverged significantly from master.
Merge or Delete Unnecessary Branches Safely:
Before deleting a branch, ensure it’s not referenced by any other ongoing work.
Automate Versioning and Branch Management:
Implement a policy to avoid version collisions in the future.

Let's dive into each step with more detail:

Step 1: Audit the Branches

We start by listing all branches using git branch. For simplicity, let’s assume we have a script that can output this list. This is our starting point.

BASH
git branch

For each branch, we need to check:

Active Development: Look at recent commits and commit messages for clues about current activity.
Dependencies: Check if the migration code depends on any other unfinished migrations.
Step 2: Identify Prune Candidates

Based on the audit, we can identify branches that are:

Old and no longer in use.
Diverged significantly from master and might have diverging features.
BASH
git branch --merged | grep -v "master" | xargs git branch -d
Step 3: Merge or Delete Unnecessary Branches Safely

Before deleting a branch, we should ensure that it doesn't reference any active development. We can use the git branch command with additional checks:

BASH
for branch in $(git branch --merged | grep -v "master"); do
    if git branch -a | grep -q "$branch"; then
        echo "Branch $branch is merged and safe to delete."
        git branch -d $branch
    else
        echo "Branch $branch is not referenced by remotes, but check locally:"
        git show-ref --verify --quiet refs/heads/$branch || echo "No local reference found for $branch, can be deleted safely."
    fi
done
Step 4: Automate Versioning and Branch Management

To prevent future version collisions:

Version Numbering: Implement a consistent naming convention or use semantic versioning.
Automated Checks: Integrate Git hooks to enforce rules about migration versions.

Here’s an example of setting up pre-commit hooks:

BASH
# Pre-commit hook example in .git/hooks/pre-commit
#!/bin/bash

set -e

# Ensure only valid migrations are committed
for file in $(git diff --name-only | grep "\.sql$"); do
    if [[ ! $file =~ "v[0-9]+_[a-z0-9_\-]+$" ]]; then
        echo "Migration file $file does not match the expected version pattern."
        exit 1
    fi
done

echo "Pre-commit checks passed!"

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, we’ve outlined a process to audit and clean up stale branches by identifying and merging or deleting unnecessary ones. Automation through Git hooks ensures that future migrations follow a consistent versioning scheme.

#### Common mistakes
include:

Not checking references before deletion, leading to accidental data loss.
Missing active development on branches, causing disruptions.
Failing to enforce version naming conventions, resulting in collisions.

For further discussion, we could explore more advanced GitOps strategies or integrate continuous integration tools for even better branch management and version control.

