---
type: "practice"
difficulty: "Intermediate"
---

# How would you set up branch protection so your alert-config repo requires a passing CI lint check before anyone merges to main?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Signed Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Signed_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that any merge into main in an AIOps alert-config repository passes a CI lint check, we need to set up branch protection rules in GitHub. These rules help maintain the quality and consistency of our codebase by enforcing certain conditions before a pull request (PR) can be merged. The key idea is to configure a pre-commit or pre-merge hook that runs a CI pipeline to validate changes before allowing them into main.

#### Triage

The question asks us to set up branch protection rules in GitHub for the alert-config repository. Specifically, we need to ensure that any merge into the main branch requires a passing CI lint check. The interviewer is probing our understanding of:

Branch Protection Rules: How to configure these rules to enforce specific conditions.
CI/CD Pipelines: The integration between GitHub and CI tools like Jenkins or GitHub Actions.
Linting: How to use linters to catch common coding mistakes.

#### First-Principles Solution
Step 1: Define the Lint Check

First, we need a clear definition of what constitutes a passing lint check. For example, if you are using Python, this might mean running flake8 or black and ensuring no errors are found.

Define Lint Check
Implement Linter
Step 2: Configure GitHub Actions Workflow

Next, we need to set up a GitHub Actions workflow that runs the lint check as part of a CI pipeline. This involves creating a .github/workflows directory in our repository and adding a YAML file for the lint check.

YAML
# .github/workflows/lint.yml
name: Lint Check

on:
  pull_request:
    branches: [ main ]
  push:
    branches: [ main ]

jobs:
  linter:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.x'

    - name: Install dependencies
      run: |
        pip install flake8

    - name: Run Linter
      run: |
        flake8 .
Step 3: Set Up Branch Protection Rules

Finally, we configure the branch protection rules to enforce that a pull request (PR) must pass the lint check before merging. This involves going to the Settings tab of your repository in GitHub and navigating to Branches.

Add a rule for the main branch.
Enable "Require status checks to pass before merging".
Specify that only "Lint Check" status checks are required.
Configure Branch ProtectionRules
Enable Status Checks

#### Closing Summary
and Common Mistakes/Follow-Ups

By setting up these branch protection rules, we ensure that any change to the alert-config repository must first pass a lint check. This helps maintain code quality and consistency.

#### Common mistakes
include:

Failing to define clear linting rules.
Not properly configuring the GitHub Actions workflow or the branch protection settings.
Not testing the setup thoroughly before relying on it in production.

#### Follow-ups
might involve discussing how to handle exceptions, such as when specific changes require manual approval despite failing a lint check.

**Code:**

```yaml
# .github/workflows/lint.yml
name: Lint Check

on:
  pull_request:
    branches: [ main ]
  push:
    branches: [ main ]

jobs:
  linter:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.x'

    - name: Install dependencies
      run: |
        pip install flake8

    - name: Run Linter
      run: |
        flake8 .
```

