---
type: "practice"
difficulty: "Deep"
---

# A cross-cutting change needs to touch the shared feature schema, training code, and serving code atomically, but each area has different CODEOWNERS. How do you balance an atomic Git commit against per-area review without deadlocking the merge?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Branching Strategies](<../../../../TOOLS/Git_and_GitHub/Git_Core/Branching_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a cross-cutting change that affects multiple areas of your codebase, ensuring atomic commits while respecting the different CODEOWNERS and their responsibilities can be challenging. The key is to find a balance between adhering to the principles of modular ownership and maintaining efficient development workflows.

This question primarily probes how well you understand Git workflows and how you handle complex dependencies within a distributed team environment. Let's break down the problem step by step:

Triaging the Problem

The challenge lies in ensuring that changes are committed atomically, meaning they go through all necessary reviews before deployment. However, each area (shared feature schema, training code, and serving code) has its own set of CODEOWNERS who may not be aware or available for simultaneous review. This can lead to deadlocks where a change is stuck because different parts of the system are under separate ownership.

#### Assumptions
Each area has distinct CODEOWNERS who are responsible for reviewing changes in their respective domains.
The changes need to be committed atomically but may require asynchronous reviews.
Deadlocks can occur if one part of the change is stuck waiting for approval from an owner unavailable at that moment.

#### First-Principles Solution
Decompose the Change: Break down the cross-cutting change into smaller, more manageable parts. Each part should be self-contained and only affect one area.
Parallel Reviews: Initiate parallel reviews in each area as soon as possible. This can help in ensuring that each part is reviewed quickly, even if the final merge requires coordination among different owners.
Merge Strategy:
Use a strategy like git rebase to keep the commit history clean and atomic.
Utilize GitHub Actions or similar CI/CD pipelines to automate parts of the review process, such as automated tests or linter checks.

#### Implementation
Example

Initial Commit: Make initial commits in each area separately:

BASH
# For shared feature schema
git commit -am "Refactor shared feature schema"

# For training code
git commit -am "Update training model"

# For serving code
git commit -am "Improve serving API"

Parallel Reviews: Start reviewing each of these commits asynchronously:

BASH
# Assuming the shared feature schema owner is @feature-owner
git push --set-upstream origin feature-schema

# Assuming the training code owner is @model-owner
git push --set-upstream origin model-update

# Assuming the serving code owner is @serving-owner
git push --set-upstream origin api-improvements

Merge Strategy:

Use git rebase to integrate these changes into a single atomic commit before merging.
BASH
# Switch to main branch if not on it
git checkout main

# Pull the latest changes from remote
git pull origin main

# Rebase each feature branch onto main
git rebase feature-schema
git rebase model-update
git rebase api-improvements

# Resolve any conflicts that arise during rebasing.

# Now, all changes are integrated into one atomic commit
git checkout -b temp-merge

# Merge the rebased branches atomically
git merge feature-schema --no-commit
git merge model-update --no-commit
git merge api-improvements --no-commit

# Resolve any conflicts and commit
git add .
git commit -m "Merge shared schema, training code, and serving code changes"

# Push the final atomic commit to origin
git push origin temp-merge:main --force

#### Closing Summary

By decomposing the change into smaller parts, initiating parallel reviews, and using Git rebase for atomic commits, you can efficiently manage cross-cutting changes while respecting different CODEOWNERS. This approach ensures that the merge process is smooth and avoids deadlocks.

#### Common Mistakes/Follow-Ups
Overlapping Reviews: Ensuring that all necessary reviews are initiated early to avoid delays.
Manual Conflicts: Using tools like GitHub Actions for automated checks can reduce manual conflicts.
Complex Interactions: Understanding how different parts of the system interact and ensuring that changes do not introduce unintended side effects.

**Code:**

```bash
# Example Git commands
git commit -am "Refactor shared feature schema"
git push --set-upstream origin feature-schema

git checkout main
git pull origin main
git rebase feature-schema
git rebase model-update
git rebase api-improvements

git checkout -b temp-merge
git merge feature-schema --no-commit
git merge model-update --no-commit
git merge api-improvements --no-commit
git add .
git commit -m "Merge shared schema, training code, and serving code changes"
git push origin temp-merge:main --force
```

