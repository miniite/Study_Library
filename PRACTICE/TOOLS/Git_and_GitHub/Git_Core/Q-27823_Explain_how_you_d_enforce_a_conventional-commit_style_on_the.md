---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you'd enforce a conventional-commit style on the schema repo (feat/fix/perf plus a table-scope) so an automated changelog and semantic schema version can be derived, and how a squash-merge can destroy that if handled naively.

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Conventional Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Conventional_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce a conventional-commit style on the schema repo and derive an automated changelog and semantic schema version, we need to ensure that commit messages follow a specific format. This involves setting up Git hooks in the repository to validate commit messages against the desired format. We'll also need to manage merge strategies carefully, especially when squash-merging pull requests.

The key steps are:

Define the conventional-commit format.
Implement pre-commit and pre-push hooks to enforce this format.
Use semantic versioning for schema changes.
Handle merges with care to preserve commit history integrity.

Let's break it down:

#### Triage
What the Question is Asking: The question requires setting up a Git workflow that enforces conventional commits, manages semantic versions of database schemas, and ensures that squash-merges do not destroy this structure.

#### Assumptions
You have access to git hooks (pre-commit and pre-push).
The schema repo uses Git for version control.
The team is familiar with conventional commit standards.

#### Solution
Step 1: Define the Conventional-Commit Format

Conventional commits follow a specific format:

CODE
<type>(<optional scope>): <subject>
<body>
<footer>

Where <type> can be feat, fix, or perf. The scope is an optional part that describes the area affected.

Step 2: Implement Git Hooks

To enforce this, we'll create pre-commit and pre-push hooks:

Pre-Commit Hook (pre-commit)

BASH
#!/bin/bash

# Check if the commit message follows the conventional-commit format.
if ! git diff --cached --name-status | grep -qE '^(\w+\/)?feat(?!\/)|^(\w+\/)?fix(?!\/)|^(\w+\/)?perf(?!\/)'; then
  echo "Commit message does not follow the conventional-commit format."
  exit 1
fi

# Optionally, you can add checks for body and footer if needed.

Pre-Push Hook (pre-push)

BASH
#!/bin/bash

# Check commit messages against the pre-commit hook to ensure they are valid before pushing.
if ! git diff --cached --name-status | grep -qE '^(\w+\/)?feat(?!\/)|^(\w+\/)?fix(?!\/)|^(\w+\/)?perf(?!\/)'; then
  echo "Commit message does not follow the conventional-commit format."
  exit 1
fi

Explanation:

The pre-commit hook ensures that only commits following the specified pattern are committed.
The pre-push hook prevents pushing invalid commits to remote repositories.
Step 3: Semantic Versioning for Schema Changes

To manage schema changes, we can use semantic versioning (semver). Each commit message should indicate a change in the database schema. For example:

CODE
feat(schema)/username: Add new column 'is_active' to users table

This ensures that every commit is linked to specific changes and helps in generating an automated changelog.

Step 4: Handling Merges with Care

Squash-merging pull requests can destroy the commit history, making it difficult to track individual schema changes. To avoid this:

Do Not Squash-Merge: Instead of squash-merging, encourage feature branches to be merged using a rebase and merge strategy.
Rebase the branch onto the latest master.
Merge into master with a merge commit, which preserves the history.

#### Example
Workflow:

Create a feature branch from main.
Make changes and commit following conventional commit format.
Push to remote repository and open a pull request.
Rebase the feature branch onto main locally before merging.
Merge the branch into main using git merge.

#### Closing Summary

By setting up pre-commit and pre-push hooks, ensuring semantic versioning for schema changes, and handling merges with rebase-then-merge strategy, we can enforce a conventional-commit style that allows for automated changelogs and clean semantic schema versions. Naive squash-merges can destroy this structure, so it's crucial to adopt a disciplined merge workflow.

#### Common Mistakes/Follow-Ups
Ignoring Commit Messages: Developers might skip writing proper commit messages.
Misusing Squash-Merge: Not rebasing feature branches before merging them into main.
Overlooking Pre-Commit Hooks: These can be easily bypassed if not properly configured or understood.

By addressing these points, we ensure a robust and maintainable database schema development process.

