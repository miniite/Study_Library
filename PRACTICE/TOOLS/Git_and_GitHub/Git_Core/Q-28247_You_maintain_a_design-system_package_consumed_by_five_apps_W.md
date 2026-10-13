---
type: "practice"
difficulty: "Intermediate"
---

# You maintain a design-system package consumed by five apps. Walk through your release flow using tags, semantic versioning, and a GitHub Actions publish job, and how you'd handle a broken release.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reusable Workflows](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reusable_Workflows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When managing a design system used across multiple applications, ensuring smooth and reliable releases is crucial. My release flow leverages Git tags and semantic versioning (SemVer) combined with GitHub Actions to automate the publishing process and maintain consistency. Here’s how I handle each step:

#### Overview
The key idea here is to use Git tags for versioning, ensure that every tagged commit is properly tested, and utilize GitHub Actions to automate the release process, including tagging, building, and pushing to a package registry.

#### Triaging the Question
This question probes my understanding of semantic versioning (semver) and how I integrate CI/CD pipelines with Git in a production environment. The assumptions are that each app consuming this design system follows best practices, such as using the same dependency manager (e.g., npm/yarn), and that the project has an existing GitHub Actions setup.

#### First-Principles Solution

Versioning: We use semantic versioning to ensure every commit is tagged with a vX.Y.Z format. This helps in tracking releases, patches, and major/minor updates.
GitHub Branch Protection Rules: Ensure that only certain branches (e.g., main) can be pushed to, which triggers the release process via GitHub Actions.
Release Workflow:
Commit Changes: Developers push changes to the repository. For a new feature or bug fix, the commit message should follow semver guidelines.
Create Tag: When a new version is ready for release (e.g., v3.0.0), I create a Git tag using git tag v3.0.0.
Push Tag to Remote: This triggers a GitHub Actions workflow that builds and publishes the package.
GitHub Actions Workflow:
YAML
name: Publish Design System Package

on:
  push:
    tags:
      - 'v*.*.*'

jobs:
  build-and-publish:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - run: npm ci
      - run: npm run build # Custom script to prepare the package for publishing
      - name: Publish Package
        env:
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
        run: |
          npm publish --access public
Handling Broken Releases: If a release breaks an app, we have a rollback mechanism. This involves updating the version in each consuming app to a previous tag or using a fallback package version.

#### Closing Summary
and Common Mistakes/Follow-Ups:

The key is to maintain robust testing before tagging and releasing new versions.
A common mistake is not having thorough tests for breaking changes, which can lead to application breakage.
For follow-ups, I’d discuss how to integrate automated tests with the release process, ensuring that every tag triggers a CI build that passes all necessary checks.

