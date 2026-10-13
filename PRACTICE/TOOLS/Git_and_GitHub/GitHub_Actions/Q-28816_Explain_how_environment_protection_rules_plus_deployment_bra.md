---
type: "practice"
difficulty: "Deep"
---

# Explain how environment protection rules plus deployment branch policies together prevent a non-main branch from ever hitting prod.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reusable Workflows](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Reusable_Workflows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing GitHub Actions and deployment strategies, it's crucial to understand how environment protection rules and deployment branch policies work in tandem to ensure that only changes on the main (or protected) branches can be deployed to production environments. These mechanisms are designed to prevent unintended or unauthorized deployments from reaching a live environment.

First, let’s break down what each of these concepts entails:

Environment Protection Rules: These rules govern when and how actions can be executed in specific environments (like production, staging, etc.). They ensure that certain conditions must be met before an action is allowed to run.
Deployment Branch Policies: These policies define which branches are eligible for deployment. Typically, only branches with the required permissions or labels can trigger a production deployment.

Here’s how these rules work together:

Branch Protection Rules:

Ensure that only specific branches (like main or master) can be pushed to.
Require pull requests before changes are merged into protected branches.
Enforce signing and verification for commits.

Deployment Branch Policies:

Specify which branches are allowed to trigger deployments.
Typically, the main branch is the only one that can deploy directly to production without requiring a manual approval or additional steps.
Step-by-Step Solution
1. Environment Protection Rules

Environment protection rules define when and how actions (like deployments) are permitted in different environments. For example:

YAML
# Example GitHub Actions configuration for environment protection
on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  deploy-prod:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy to production
        run: echo "Deploying to prod"

In this configuration, the deploy-prod job can only be triggered when pushing directly to the main branch. If someone tries to deploy from another branch, the job will not run.

2. Deployment Branch Policies

Deployment branch policies ensure that only specific branches are allowed to trigger deployments. For instance:

Main Branch Policy: The main branch is protected so it can't be pushed to directly, requiring pull requests for changes.
Allowed Deployment Branches: Only the main branch should have the ability to deploy to production.
Full Example Configuration

Here’s a complete example of how these rules might be set up in GitHub Actions:

YAML
# GitHub Actions configuration
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  workflow_dispatch:

env:
  PROD_BRANCH: "main"

jobs:
  # Development Environment
  test-deploy-dev:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/dev'
    steps:
      - name: Test deploy to dev environment
        run: echo "Deploying to dev environment"

  # Production Environment
  deploy-prod:
    runs-on: ubuntu-latest
    if: github.ref == env.PROD_BRANCH
    steps:
      - name: Deploy to production
        run: echo "Deploying to prod"

In this setup, the deploy-prod job can only be executed when pushing directly to the main branch. Any other branch (like develop) will not trigger this job.

#### Closing Summary
and Common Mistakes

The key takeaway is that environment protection rules and deployment branch policies work in concert to ensure that production deployments are controlled and restricted to only specific branches, typically the main branch. This helps prevent accidental or unauthorized deployments into production environments.

#### Common mistakes
include:

Not having strict branch protection on the main branch.
Allowing any non-main branch to trigger production deployments.
Failing to enforce environment-specific rules for actions like deployments.

