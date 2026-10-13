---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a short GitHub Actions job that runs yamllint on your Airflow and dbt YAML files and fails the PR if formatting is wrong?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Secure Workflows for Forks](<../../../../TOOLS/Git_and_GitHub/Git_Core/Secure_Workflows_for_Forks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure consistent YAML formatting in our CI/CD pipeline, we'll create a GitHub Actions job to run yamllint on both Airflow and dbt YAML files. This job will be triggered by pull requests (PRs) and will fail the PR if any of the YAML files contain linting errors.

#### Triaging the Question

The core task is to set up a GitHub Actions workflow that uses yamllint to check the formatting of Airflow and dbt YAML files. The key points are:

Trigger: We need the job to run automatically on PRs.
Tools: yamllint for linting the YAML files.
Action: If any file is not formatted correctly, the action should fail the PR.
Step-by-Step Solution
Understand the Prerequisites:
We need a GitHub repository with Airflow and dbt projects.
yamllint must be installed in the CI/CD environment.
Create the GitHub Actions Workflow:
Define a .github/workflows/check-yaml-format.yml file.
Define the Steps:
Install dependencies (if any).
Run yamllint on Airflow and dbt YAML files.
Handling Failures:
If yamllint finds issues, the job should fail.

#### Example
Workflow

Here's a sample GitHub Actions workflow that accomplishes this:

YAML
# .github/workflows/check-yaml-format.yml

name: Check YAML Formatting

on:
  pull_request:
    branches:
      - main
      - develop

jobs:
  check-yaml-format:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Python (if needed)
      if: startsWith(github.ref, 'refs/heads/main')
      run: |
        python3 --version
        pip install yamllint

    - name: Run yamllint on Airflow and dbt YAML files
      run: |
        find . -name "*.yaml" -exec yamllint {} \;

#### Closing Summary
and Common Mistakes/Follow-Ups

This workflow ensures that any linting issues in your YAML files are caught early, preventing potential bugs or errors. Key points to note:

Dependencies: Ensure yamllint is installed.
File Paths: Adjust the file paths if your project structure differs.
Branches: The on.pull_request.branches part ensures this runs only on specified branches.

#### Common mistakes
include missing dependencies, incorrect file path patterns, or failing to handle different branch configurations. Follow-ups might involve adding more detailed logging or integrating with other CI/CD tools for a full stack approach.

**Code:**

```yaml
# .github/workflows/check-yaml-format.yml

name: Check YAML Formatting

on:
  pull_request:
    branches:
      - main
      - develop

jobs:
  check-yaml-format:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Python (if needed)
      if: startsWith(github.ref, 'refs/heads/main')
      run: |
        python3 --version
        pip install yamllint

    - name: Run yamllint on Airflow and dbt YAML files
      run: |
        find . -name "*.yaml" -exec yamllint {} \;
```

**Flow:**

1. If issues found
2. No issues found
3. Checkout code
4. Set up Python (if needed)
5. Run yamllint on Airflow anddbt YAML files
6. Fails PR
7. Passes CI/CD

