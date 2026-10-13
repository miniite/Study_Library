---
type: "practice"
difficulty: "Deep"
---

# Your team keeps a prompt-registry pattern where prompts are Git-tracked and promoted dev to staging to prod by copying directories, and drift crept in. How would you guarantee prod prompts can only receive changes that already passed staging in Git?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reviewing Changes with Diff](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reviewing_Changes_with_Diff.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that production (prod) prompts only receive changes that have already been reviewed and approved through staging, we need to enforce a strict workflow where changes pass through multiple stages before reaching prod. This involves using Git branches and tags, implementing semantic versioning, and leveraging GitHub actions for automated reviews and deployments.

#### Triage

The core issue is drift in the prompt-registry pattern, which occurs when changes are directly copied between environments without proper review or validation. We need to ensure that all changes go through a staging environment before reaching prod. Specifically, we will use Git branches to manage different stages of deployment, enforce code reviews via GitHub actions, and use tags for versioning.

#### First-Principles Solution
Git Workflow: Implement a Git workflow with separate branches for development (dev), staging (stg), and production (prod).
Branch Management: Ensure that only changes approved in the dev environment are merged into the stg branch.
Automated Reviews: Use GitHub actions to trigger reviews, tests, and linting before allowing merge requests from dev to stg.
Tagging for Versioning: Utilize Git tags to mark specific versions that pass staging and can be deployed to prod.
Step-by-Step Solution

Branch Structure:

main: Base version of the repository, protected against direct pushes and only updated via merges from staging.
dev: Development branch where new features or changes are made.
staging: Integration branch where changes from dev are merged after passing reviews and tests.
prod: Production branch that reflects the current state of prod.

Enforcing Workflow:

Merge Policy: Only allow merges from dev to staging. This can be enforced using GitHub’s merge button or via a Git hook if you are not using GitHub Actions.
Automated Tests and Linting: Set up GitHub actions for continuous integration (CI) that runs tests, lints code, and triggers reviews when changes are pushed to the dev branch. Once these checks pass, developers can request to merge into staging.

Tagging:

When a change passes through staging successfully, tag it with an appropriate semantic version in staging. For example, if a change is tagged as v1.2.3, this version should be deployed to prod after manual review and approval.

Deployment Pipeline:

Manual Approval: Set up a GitHub action that automatically opens a pull request from staging to prod when a tag is created.
Post-Approval Deployment: Once the pull request is manually approved, deploy the tagged version to prod using another GitHub action or your CI/CD pipeline.

#### Closing Summary

By implementing this structured workflow, we can ensure that only changes that have passed through thorough reviews and tests in staging make it to production. This approach leverages Git branches for managing different stages of deployment and GitHub actions for automating the review process.

#### Common Mistakes/Follow-Ups
Merging Directly: Avoid merging directly from dev to prod without going through staging.
Manual Bypasses: Ensure that there are no manual bypasses around automated tests or reviews, as these can introduce drift.
Tagging Consistency: Maintain consistent tagging practices across the team to avoid confusion and ensure accurate versioning.

**Comparison:**

| Branch  | Purpose          |
|---------|------------------|
| main    | Base code        |
| dev     | Development      |
| stg     | Integration, Test |
| prod    | Production       |
| Tagging Scheme   | Description                     |
|------------------|---------------------------------|
| v1.2.3           | Version tag after staging approval |

**Code:**

```
Example GitHub Actions Workflow for Promoting Changes

.github/workflows/ci.yml (Continuous Integration)

YAML
Copy
name: CI

on:
  push:
    branches:
      - dev

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - name: Run Tests
        run: pytest
      - name: Lint Code
        run: flake8

.github/workflows/pr-checks.yml (Pull Request Checks)

YAML
Copy
name: PR Check

on:
  pull_request_target:
    branches:
      - staging

jobs:
  check-pr:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - name: Run Tests
        run: pytest
      - name: Lint Code
        run: flake8

.github/workflows/deploy.yml (Deploy from Staging to Prod)

YAML
Copy
name: Deploy

on:
  push:
    tags:
      - v*

jobs:
  deploy-prod:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - name: Setup Environment
        run: echo "Production environment setup"
      - name: Deploy to Prod
        run: |
          # Deployment logic here
          echo "Deployed version $(git describe --tags)"
```

**Example input / output:**

```text
# Example Dry-Run

## Input:
- Dev branch has new prompt changes.
- New PR from dev to stg is created and merged after passing CI/CD checks.

## Output:
- Staging environment receives the new changes, undergoes tests.
- After successful testing, v1.2.3 tag is applied to staging.
- Manually approved pull request opens from staging to prod.
- Prod environment is updated with version v1.2.3 after manual review and approval.
```

**Data:**

```json
{
  "branch-structure": {
    "main": "Base code",
    "dev": "Development branch for new features",
    "staging": "Integration and testing branch",
    "prod": "Production environment"
  },
  "tagging-policy": {
    "v1.2.3": "Version tag after successful staging deployment"
  }
}
```

**Flow:**

1. Dev
2. Submit PR to Staging
3. Staging Integration
4. Pass CI/CD Checks
5. Tag v1.2.3 in Staging
6. Manual PR Opened fromStaging to Prod
7. Prod Update after Review

