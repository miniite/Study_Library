---
type: "practice"
difficulty: "Intermediate"
---

# Your team's runbooks and IaC live in the same repo, and a runbook-only PR triggers a full infra plan and burns cost. How would you separate change classes so docs changes and infra changes have independent CI without splitting the repo?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reducing Actions Costs](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reducing_Actions_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of separating runbooks from infrastructure code in a single Git repository while ensuring that only relevant pipelines are triggered, we need to implement a more granular approach to our continuous integration (CI) process. The key idea is to leverage Git hooks and branches to distinguish between changes affecting documentation and those affecting infrastructure.

In essence, the question is about designing a CI/CD pipeline system where different types of code changes can be handled independently without requiring multiple repositories. By doing this, we aim to prevent unnecessary infrastructure deployments triggered by runbook-only pull requests (PRs).

Firstly, let's assume some basic setup:

Our repository contains both documentation and IaC.
We use GitHub Actions for our CI/CD needs.

Our goal is to ensure that:

Documentation changes do not trigger any infrastructure plan.
Infrastructure code changes only trigger the necessary plans without affecting other parts of the repository.

To achieve this, we can follow these steps:

Triaging the Problem

The core issue here is ensuring that the CI/CD pipeline recognizes different types of changes and triggers appropriate actions based on those changes. We need to implement a mechanism that can differentiate between runbooks (documentation) and infrastructure code in our Git repository.

#### Solution
from First Principles
Branch Naming Convention:
Use distinct branch naming conventions for documentation and IaC. For example, use docs/* for documentation branches.
Git Hooks:
Implement pre-commit hooks that can inspect the changes made in each commit to determine if they are related to runbooks or infrastructure code.
GitHub Actions Workflows:
Create separate GitHub Action workflows for different branch types.
Step-by-Step Implementation
1. Branch Naming Convention

We will use a naming convention where docs/* branches represent documentation changes, and the main development branches (like main, develop) are used for infrastructure code.

PLAINTEXT
Branches:
- docs/update-readme: Documentation change branch
- develop: Development branch with both runbooks and IaC
2. Pre-commit Hooks

We can create a pre-commit hook that checks the commit messages to determine if they contain keywords related to documentation or infrastructure changes.

BASH
#!/bin/bash

# Check for keywords in commit message
if echo $GIT_COMMIT_MESSAGE | grep -q 'docs'; then
    echo "Detected doc changes. Skipping infra plan."
else
    # Proceed with normal workflow
fi
3. GitHub Actions Workflows

We will create separate workflows to handle different types of branches:

YAML
# .github/workflows/docs.yml
name: Docs Workflow

on:
  push:
    branches:
      - docs/*

jobs:
  check-docs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Check documentation changes
        run: echo "Handling doc change"
YAML
# .github/workflows/main.yml
name: Main Workflow

on:
  push:
    branches:
      - main
      - develop

jobs:
  check-infra-changes:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Check infrastructure changes
        run: echo "Handling infra change"

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing the above strategy, we ensure that only relevant pipelines are triggered based on the type of changes in the repository. This not only prevents unnecessary costs but also streamlines our CI/CD process.

#### Common mistakes
include:

Not defining a clear branch naming convention.
Failing to implement appropriate Git hooks or GitHub Actions workflows.
Overcomplicating the solution with overly complex logic.

#### Follow-ups
could involve:

Adding more sophisticated logic for handling edge cases, such as mixed changes in one commit.
Ensuring that all team members are aware of the new branching strategy and how it impacts their workflow.

