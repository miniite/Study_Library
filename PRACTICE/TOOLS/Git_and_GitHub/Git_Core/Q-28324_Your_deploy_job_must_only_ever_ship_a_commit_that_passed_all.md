---
type: "practice"
difficulty: "Expert"
---

# Your deploy job must only ever ship a commit that passed all required checks on main, never a re-run against a stale ref. Wire GitHub environments, deployment protection rules, and ref pinning to enforce that end to end.

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Protected Branches and Required Checks](<../../../../TOOLS/Git_and_GitHub/Git_Core/Protected_Branches_and_Required_Checks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to ensure that our deploy job only deploys commits from main branch that have passed all required checks. This involves using GitHub's environment protection and deployment protection features along with appropriate ref pinning.

Firstly, let's summarize the key components:

GitHub Environments: These are used for setting up complex environments where your application will run.
Deployment Protection Rules: These rules control when a pull request can be deployed to production.
Ref Pinning: Ensures that only commits from a specific reference (branch) are allowed.

#### Triage

The core of the question is about ensuring robustness in our deployment pipeline. We need to enforce strict rules using GitHub's features and ensure these rules cannot be bypassed. Specifically, we want to avoid deploying based on stale references or commits that haven't passed all required checks.

#### Assumptions
The main branch will always have a green (passed) status.
No commit can pass the build and tests without being merged into main.
We need to ensure that any deployment from our CI/CD pipeline only comes from the latest main.
Step-by-Step Solution

Set Up GitHub Environments:

Create an environment for production.
Configure the environment to require a pull request merge before deploying.

Deployment Protection Rules:

Set up rules that enforce that only merges into main can trigger deployments.
Ensure these rules are strict, not allowing deployments from any other branch or ref.

Ref Pinning:

Pin the main branch in your CI/CD pipeline to ensure it always uses the latest commit from this branch.
Detailed Steps

Environment Setup:

YAML
# Example of a GitHub Actions workflow for setting up environments
on:
  push:
    branches:
      - main
jobs:
  setup-environments:
    runs-on: ubuntu-latest
    steps:
      - name: Set up environment variables
        run: echo "PROD_ENVIRONMENT=true" >> $GITHUB_ENV

Deployment Protection Rules:

Go to your repository settings and navigate to the Environments section.
Create a new environment for production, with rules that require manual approval or automatic approvals based on pull request merges.

Ref Pinning in CI/CD Pipeline:

YAML
# Example GitHub Actions workflow for deploying from main branch
name: Deploy to Production
on:
  push:
    branches:
      - main
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: ${{ secrets.PRODUCTION_URL }}
    steps:
      - uses: actions/checkout@v3
      - run: echo "Ref is pinned to `main`"
      - run: yarn install
      - run: yarn build
      - run: npm run deploy

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, by setting up proper GitHub environments, deployment protection rules, and ref pinning in our CI/CD pipeline, we can ensure that deployments are only triggered from the latest main branch with all required checks passing.

#### Common mistakes
include:

Not using ref pinning to enforce strict source control.
Allowing merges into branches other than main to trigger deployments.
Failing to set up proper protection rules in GitHub environments.

#### Follow-up
questions could involve discussing how to handle emergency updates, rollback procedures, and ensuring the integrity of the production environment.

