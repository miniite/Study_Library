---
type: "practice"
difficulty: "Deep"
---

# You maintain a GenAI SDK consumed by five internal apps. Walk through your release flow using annotated tags, semver, and a GitHub Actions publish job, and how you'd yank a broken release that emits unsafe output.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reusable Workflows](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reusable_Workflows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

When managing releases of an AI SDK used by multiple internal applications, using Git tags with semantic versioning (semver) is crucial to ensure traceability and clarity in version control. GitHub Actions can automate this process, ensuring that only validated versions are published. In the event of a release that emits unsafe output, we need a robust mechanism to yank or roll back the problematic version.

#### Triage

The key aspects of the question revolve around:

Semantic Versioning (SemVer): Understanding how to use semver effectively for tagging releases.
GitHub Actions: Automating the release process with GitHub Actions, ensuring versioned artifacts are published correctly.
Yanking a Release: Handling scenarios where a release needs to be revoked due to issues.

#### First Principles Solution

Semantic Versioning (SemVer):

We use semver tags like vX.Y.Z, where X is the major version, Y is the minor version, and Z is the patch version.
Each tag represents a distinct release with a specific set of changes.

GitHub Actions:

A GitHub Actions workflow can be configured to automatically publish artifacts tagged with semver versions.
The workflow ensures that only valid tags are published, maintaining integrity across all internal applications.

Yanking a Broken Release:

If an unsafe release is identified post-deployment, we need a way to yank or retract the problematic version while ensuring that affected apps can be updated without breaking existing integrations.
Step-by-Step Solution

Tagging Revisions with SemVer:

BASH
# Example: Tagging a new release
git tag -a v1.0.3 -m "Release 1.0.3"
git push origin --tags

GitHub Actions Workflow for Publishing Artifacts:

YAML
name: Publish SDK Release

on:
  push:
    tags:
      - 'v*.*.*'

jobs:
  build-and-publish:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: 3.8

      - name: Install dependencies
        run: |
          pip install -r requirements.txt

      - name: Build package
        run: |
          python setup.py sdist bdist_wheel

      - name: Publish to PyPI
        uses: pypa/gh-action-pypi-publish@release/v1
        with:
          password: ${{ secrets.PYPI_TOKEN }}

Handling a Yanked Release:

YAML
name: Revoke Unsafe SDK Version

on:
  issues:
    types: [labeled]

jobs:
  yank-release:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: 3.8

      - name: Revoke release version
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          # Example script to revoke the release
          echo "Revoking version v1.0.2"
          gh api repos/:owner/:repo/releases/tag/v1.0.2 --method DELETE

      - name: Update application dependencies
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          # Example script to update dependencies in applications
          echo "Updating dependencies in apps"

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, using semantic versioning (semver) alongside GitHub Actions ensures a structured release process. The Publish SDK Release workflow automates the publishing of tagged versions, while the Revoke Unsafe SDK Version workflow provides a mechanism to yank problematic releases, safeguarding internal applications.

#### Common mistakes
include:

Not adhering strictly to semver rules.
Failing to update application dependencies when revoking a release version.
Overlooking the need for thorough testing before tagging and publishing new versions.

#### Common Mistakes/Follow-Ups
Testing: Always thoroughly test each version in isolation and integration environments.
Communication: Ensure clear communication with stakeholders about release schedules and potential risks.
Documentation: Maintain comprehensive documentation of release procedures to aid future maintenance and troubleshooting.

**Code:**

```yaml
name: Publish SDK Release

on:
  push:
    tags:
      - 'v*.*.*

jobs:
  build-and-publish:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: 3.8

      - name: Install dependencies
        run: |
          pip install -r requirements.txt

      - name: Build package
        run: |
          python setup.py sdist bdist_wheel

      - name: Publish to PyPI
        uses: pypa/gh-action-pypi-publish@release/v1
        with:
          password: ${{ secrets.PYPI_TOKEN }}

name: Revoke Unsafe SDK Version

on:
  issues:
    types: [labeled]

jobs:
  yank-release:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: 3.8

      - name: Revoke release version
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          echo "Revoking version v1.0.2"
          gh api repos/:owner/:repo/releases/tag/v1.0.2 --method DELETE

      - name: Update application dependencies
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          echo "Updating dependencies in apps"
```

