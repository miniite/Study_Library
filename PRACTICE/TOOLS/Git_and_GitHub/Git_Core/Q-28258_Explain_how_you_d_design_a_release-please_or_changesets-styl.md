---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd design a release-please or changesets-style automated release pipeline that derives the version bump and changelog from commit messages, and what convention it forces on the team.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Conventional Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Conventional_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an automated release pipeline using Git & GitHub, we'll focus on creating a system that derives the version bump and changelog from commit messages. This approach not only streamlines the release process but also enforces clear and consistent documentation practices within the team. The key is to leverage Git hooks and GitHub Actions for automation.

#### Triaging the Question

The question asks us to design an automated release pipeline that extracts version bumps and changelogs based on commit messages. The interviewer wants to probe our understanding of:

Commit Message Conventions: How we define and enforce commit message formats.
Automated Tools & Platforms: Which tools and platforms (Git hooks, GitHub Actions) would be utilized.
Release Workflow: The steps involved in the release process, including versioning, changelog generation, and deployment.

#### Assumptions
We are using Git for version control.
GitHub is used as the code repository.
The team follows a standard commit message format.
We have a clear understanding of the release cycle (e.g., major, minor, patch versions).
Step-by-Step Solution
1. Define Commit Message Conventions

To ensure that our pipeline can reliably extract version bumps and changelogs, we need to establish strict commit message conventions. A common format is:

CODE
<type>(<scope>): <subject>
<body>
<footer>

Where:

Type: feat, fix, refactor, etc.
Scope: Optional context (e.g., "feat(auth)").
Subject: Brief description of the change.
Body: Additional information.
Footer: Metadata like issue references, breaking changes.
2. Set Up Git Hooks

We'll use pre-commit and post-commit hooks to enforce these conventions:

pre-commit Hook: Ensures that commit messages follow the required format before commits are allowed.
post-commit Hook: Logs or sends alerts if any dev departs from the convention after committing but before pushing.
3. Create a GitHub Action

GitHub Actions will automate the release process based on branch names and push events:

Stages:

Changelog Generation: Extracts relevant commit messages into a changelog.
Version Bumping: Updates the version in package.json or equivalent.
Deployment: Deploys the new version to staging and production environments.

Workflow YAML:

YAML
name: Release Pipeline

on:
  push:
    branches:
      - main

jobs:
  build-and-release:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Repository
      uses: actions/checkout@v2

    - name: Set up Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '14'

    - name: Install dependencies
      run: npm install

    - name: Generate Changelog
      id: changelog
      run: |
        git fetch --tags
        npx lerna version --no-commit-hooks --no-push | tee newVersion
        echo "::set-output name=new_version::${newVersion}"

    - name: Bump Version
      env:
        NEW_VERSION: ${{ steps.changelog.outputs.new_version }}
      run: |
        sed -i "s/\(version\": \).*/\1\"$NEW_VERSION\"/" package.json

    - name: Build and Test
      run: npm run build && npm test

    - name: Deploy to Staging
      if: github.ref == 'refs/heads/main'
      run: |
        npm publish --tag next
4. Changelog Generation with Lerna

We'll use a tool like lerna or commitizen to parse commit messages and generate the changelog:

BASH
npx lerna version --no-commit-hooks --no-push | tee newVersion

This command generates a new version number, which is then used for bumping the package.

5. Deployment Strategy

For deployment, we'll use environment-specific configurations in package.json and automated deployment tools like Jenkins or Kubernetes.

#### Closing Summary

In summary, designing an automated release pipeline that derives versions and changelogs from commit messages involves:

Establishing clear commit message conventions.
Using Git hooks to enforce these conventions.
Creating a GitHub Action workflow for versioning and deployment.
Ensuring continuous integration and delivery.

#### Common Mistakes/Follow-Ups
Lack of Commit Message Standards: Poorly defined or enforced commit messages can lead to confusion.
Over-reliance on Automation: Manual steps in the release process that are not well-documented can cause issues.
Complexity in Rollbacks: Lack of rollback strategies can complicate post-release debugging.

By addressing these points, we ensure a robust and maintainable release pipeline.

**Comparison:**

```
Commit Message	Type	Scope	Subject
feat(auth)	feat	auth	Add login functionality with OAuth2 support
```

**Code:**

```yaml
name: Release Pipeline

on:
  push:
    branches:
      - main

jobs:
  build-and-release:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Repository
      uses: actions/checkout@v2

    - name: Set up Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '14'

    - name: Install dependencies
      run: npm install

    - name: Generate Changelog
      id: changelog
      run: |
        git fetch --tags
        npx lerna version --no-commit-hooks --no-push | tee newVersion
        echo "::set-output name=new_version::${newVersion}"

    - name: Bump Version
      env:
        NEW_VERSION: ${{ steps.changelog.outputs.new_version }}
      run: |
        sed -i "s/\(version\": \).*/\1\"$NEW_VERSION\"/" package.json

    - name: Build and Test
      run: npm run build && npm test

    - name: Deploy to Staging
      if: github.ref == 'refs/heads/main'
      run: |
        npm publish --tag next
```

**Example input / output:**

```text
Input:
- Commit message: "feat(auth): Add login functionality with OAuth2 support"

Output:
- New version number: 1.0.1 (assuming the previous version was 1.0.0)
- Changelog entry:
  - feat(auth): Add login functionality with OAuth2 support
```

**Data:**

```json
{
  "type": "feat",
  "scope": "auth",
  "subject": "Add login functionality with OAuth2 support",
  "body": "This commit adds the ability to authenticate using OAuth2 providers. The implementation includes error handling and logging.",
  "footer": ""
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. Commit
5. Pre-commit Hook
6. Valid Commit?
7. Git Fetch Tags
8. Lerna Version
9. New Version?
10. Sed Package.json
11. NPM Publish
12. Post-commit Hook

