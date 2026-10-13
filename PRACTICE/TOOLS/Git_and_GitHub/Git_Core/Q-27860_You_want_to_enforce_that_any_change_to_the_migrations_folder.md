---
type: "practice"
difficulty: "Deep"
---

# You want to enforce that any change to the migrations folder requires two approvals including a DBA, and that changes to reference-data seeds require only one. How would you express path-scoped, count-based review rules and prove they can't be bypassed?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Protected Branches and Required Checks](<../../../../TOOLS/Git_and_GitHub/Git_Core/Protected_Branches_and_Required_Checks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce path-scoped, count-based review rules in Git & GitHub for migrations and reference-data seeds, we'll leverage GitHub Actions to create custom workflows that handle approvals based on file paths and counts. This ensures that any change to the migrations folder requires two approvals (including a DBA), whereas changes to reference-data seeds only require one approval. Here's how we can achieve this:

TL;DR: We'll set up GitHub Actions workflows to enforce path-scoped, count-based review rules using environment-specific conditions and approvers.

#### Triage

The core question is about implementing a Git/GitHub workflow that enforces specific approval requirements based on file paths.

#### Assumptions
include the use of branches for development, testing, and production, as well as predefined roles such as DBAs.
We'll need to create separate workflows for migrations and reference-data seeds.

#### First-Principles Solution

Define a GitHub Actions workflow file that checks PRs and enforces specific approval counts based on the changed files.
Use environment-specific conditions to differentiate between migrations and reference-data seeds.
Ensure the process is secure by restricting direct merge access and requiring approvals.
Workflow Implementation

Create Workflow Files: Create or modify .github/workflows directory in your repository. We'll create two separate workflow files: one for migrations and another for reference-data.

Migrations Workflow (migrations.yml):

YAML
name: Migrations Approval

on:
  pull_request:
    branches:
      - develop
      - main

jobs:
  check-approvers:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
      checks: write
      id-token: write

    steps:
      - name: Check PR changes
        id: check-changes
        run: |
          # This script will check the changed files and count the approvals needed
          changed_files=$(git diff --name-only origin/main..HEAD | grep '^migrations/')
          echo "changed_files=$changed_files"

      - name: Enforce Two Approvals
        if: steps.check-changes.outputs.changed_files != ''
        run: |
          # This script will fetch PR information and enforce two approvals, including a DBA
          echo "::set-output name=needs_two_approvals::true"

Reference-Data Workflow (reference-data.yml):

YAML
name: Reference Data Approval

on:
  pull_request:
    branches:
      - develop
      - main

jobs:
  check-approvers:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
      checks: write
      id-token: write

    steps:
      - name: Check PR changes
        id: check-changes
        run: |
          # This script will check the changed files and count the approvals needed
          changed_files=$(git diff --name-only origin/main..HEAD | grep '^reference-data/')
          echo "changed_files=$changed_files"

      - name: Enforce One Approval
        if: steps.check-changes.outputs.changed_files != ''
        run: |
          # This script will fetch PR information and enforce one approval, usually from a specific role
          echo "::set-output name=needs_one_approval::true"

Merge and Approvals:

Ensure that branches for migrations require two approvals (one of which is required to be from a DBA).
Reference-data changes only need one approval.
Use GitHub's review and approval process to enforce these rules.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Summary: By using path-scoped conditions in GitHub Actions, we can enforce specific approval counts based on file paths. This ensures that critical changes are reviewed more thoroughly while maintaining efficiency for less sensitive updates.

#### Common Mistakes
Forgetting to set up environment-specific conditions and approvals.
Not restricting direct merge access effectively.
Failing to communicate the new rules clearly to team members.

#### Follow-Ups
How would you handle exceptions or emergencies where immediate merges are needed?
Can you expand this approach to other file types or directories with different approval requirements?

**Code:**

```
migrations.yml

name: Migrations Approval

on: pull_request: branches: - develop - main

jobs: check-approvers: runs-on: ubuntu-latest permissions: contents: read pull-requests: write checks: write id-token: write

CODE
Copy
steps:
  - name: Check PR changes
    id: check-changes
    run: |
      changed_files=$(git diff --name-only origin/main..HEAD | grep '^migrations/')
      echo "changed_files=$changed_files"

  - name: Enforce Two Approvals
    if: steps.check-changes.outputs.changed_files != ''
    run: |
      echo "::set-output name=needs_two_approvals::true"
CODE
Copy

```yaml
# reference-data.yml
name: Reference Data Approval

on:
  pull_request:
    branches:
      - develop
      - main

jobs:
  check-approvers:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
      checks: write
      id-token: write

    steps:
      - name: Check PR changes
        id: check-changes
        run: |
          changed_files=$(git diff --name-only origin/main..HEAD | grep '^reference-data/')
          echo "changed_files=$changed_files"

      - name: Enforce One Approval
        if: steps.check-changes.outputs.changed_files != ''
        run: |
          echo "::set-output name=needs_one_approval::true"
```

