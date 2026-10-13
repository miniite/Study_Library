---
type: "practice"
difficulty: "Deep"
---

# You need to migrate the default branch of a payments service from master to main across CI, protection rules, open PRs, deploy pipelines, and every clone, with zero broken deploys. Sequence the steps.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Automated Dependency Updates](<../../../../TOOLS/Git_and_GitHub/Git_Core/Automated_Dependency_Updates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When migrating a repository's default branch name from master to main, it’s crucial to ensure that no existing workflows are disrupted, especially in a critical system like a payments service where reliability and security are paramount. This process involves several steps, including updating configurations in CI/CD systems, modifying protection rules, resolving open pull requests (PRs), ensuring deploy pipelines are updated, and addressing every clone of the repository.

First, let’s clarify what we need to do:

Update CI/CD Configurations: Ensure that all build and deployment pipelines recognize main instead of master.
Modify Protection Rules: Adjust any branch protection rules to apply to main instead.
Resolve Open PRs: Rebase or merge any open PRs against the new default branch.
Update Repositories: Ensure that all clones and future clones use main as their default branch.

Now, let’s break down each step in detail:

Step 1: Update CI/CD Configurations

First, identify which CI/CD systems are being used (e.g., Jenkins, GitHub Actions, GitLab CI). For this example, we'll focus on GitHub Actions since it's the most common. Open your repository’s .github/workflows directory and review each YAML file.

YAML
# Example of a GitHub Action workflow that needs to be updated
name: CI

on:
  push:
    branches: [master]
  pull_request:
    branches: [master]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
        with:
          fetch-depth: 0
      - name: Build
        run: make build

Update the branches key in both the on and jobs sections to use main instead of master.

Step 2: Modify Protection Rules

Go to GitHub’s repository settings and navigate to the "Branches" section. Here, you can set up branch protection rules that apply to specific branches.

Update Default Branch: Change the default branch name from master to main.
Set Protection Rules for main: Ensure that the new main branch has appropriate protections such as requiring status checks to pass before merging or requiring pull requests to be signed off by certain team members.
Step 3: Resolve Open PRs

For open PRs, you need to either rebase them onto main if they are behind and need updates, or merge them into main.

Rebase/Resolve Conflicts: Use the command line or GitHub web UI to rebase each PR.
Merge Resolved PRs: Once conflicts are resolved, merge the PRs.
Step 4: Update Repositories

Ensure that all existing clones of the repository have their default branch set to main. If necessary, push a new commit with a trivial change (e.g., a comment saying "Default branch is now main") and force push it to update the remote repository’s default branch.

BASH
git checkout master  # Ensure you are on the current default branch
git commit --amend -m "Update default branch to main"
git push origin HEAD:main --force

#### Closing Summary

By following these steps, we ensure a seamless transition from master to main without breaking any existing deployments or workflows. The key is to be meticulous and test thoroughly at each stage.

#### Common Mistakes/Follow-Ups
Forgetting to Update CI/CD Configurations: This can lead to broken pipelines.
Not Resolving Open PRs Correctly: If not handled properly, it can result in merge conflicts or unresolved changes.
Not Updating Remote Repositories: Force pushing the correct default branch ensures that all clones are aligned.

