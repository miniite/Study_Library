---
type: "practice"
difficulty: "Intermediate"
---

# Why would you set up a GitHub Actions workflow to run your ML unit tests and data-validation checks on every pull request?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Secure Workflows for Forks](<../../../../TOOLS/Git_and_GitHub/Git_Core/Secure_Workflows_for_Forks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why setting up a GitHub Actions workflow is beneficial, let's first clarify what we're trying to achieve with this setup. The primary goal is to automate the execution of ML unit tests and data-validation checks whenever changes are proposed through pull requests (PRs). This ensures that any new code or dataset modifications don't break existing functionality or introduce inconsistencies.

#### Triage
Understanding the Question

The key aspects being probed here include:

Automated Testing: Ensuring continuous integration and deployment.
Code Quality & Consistency: Maintaining high standards by validating data and code quality before merging changes into the main branch.
Early Error Detection: Catching issues early in the development cycle to avoid downstream problems.

#### Assumptions
We are working with a version-controlled project using Git, specifically hosted on GitHub.
There is an existing Python or R script for running unit tests and data-validation checks.
The team uses pull requests for code review before merging changes into the main branch.

#### First-Principles Solution
Automation of Testing: By setting up a GitHub Actions workflow, we can automate the execution of our ML unit tests and data-validation checks. This ensures that every PR is thoroughly vetted without requiring manual intervention.

#### Cons
istent Environment: GitHub Actions runs in a consistent environment managed by GitHub, which helps ensure that the same conditions are met as in the development environment.
Quick Feedback Loop: Immediate feedback is provided to developers about potential issues early on, reducing the likelihood of introducing bugs into the main codebase.
Step-by-Step Solution

Define the Workflow:

Create a .github/workflows directory in your repository if it doesn't already exist.
Inside this directory, create a YAML file for your workflow (e.g., ml-unit-tests.yml).

Configure the Workflow:

YAML
name: ML Unit Tests and Data Validation

on:
  pull_request:
    branches:
      - main  # Specify which branch or branches to trigger the workflow

jobs:
  run-tests-and-validation:
    runs-on: ubuntu-latest  # Use a specific runner type

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v2

      - name: Set up Python environment
        uses: actions/setup-python@v2
        with:
          python-version: '3.8'

      - name: Install dependencies
        run: |
          pip install -r requirements.txt  # Assuming a `requirements.txt` file exists

      - name: Run ML Unit Tests
        run: |
          pytest tests/  # Running the unit test suite in the `tests/` directory

      - name: Validate Data
        run: |
          python validate_data.py  # Running data validation script

Integrate with Pull Requests:

Once the workflow is set up, it will automatically trigger whenever a PR is opened or updated.
The actions defined in the YAML file (e.g., running tests and validating data) will be executed.

#### Closing Summary
& Common Mistakes
Summary: Setting up GitHub Actions for ML unit tests and data-validation checks on pull requests ensures automated, consistent testing, providing quick feedback to developers and maintaining code quality.

#### Common Mistakes
Not defining the on keyword properly: This can prevent the workflow from running at all.
Running out-of-date or incorrect scripts: Ensure that the paths and commands in your YAML file match your actual files and directory structure.

#### Example
Workflow Execution

Let's walk through a hypothetical input scenario:

Input: A developer opens a PR for changes made to both code and data.
Output: GitHub Actions runs the specified tests and validation checks. If everything passes, it continues the PR process; if not, it blocks until issues are resolved.
Final Confirmation

The workflow setup ensures automated testing and data validation, improving code quality and reducing integration issues.

**Code:**

```yaml
name: ML Unit Tests and Data Validation

on:
  pull_request:
    branches:
      - main

jobs:
  run-tests-and-validation:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v2

      - name: Set up Python environment
        uses: actions/setup-python@v2
        with:
          python-version: '3.8'

      - name: Install dependencies
        run: |
          pip install -r requirements.txt

      - name: Run ML Unit Tests
        run: |
          pytest tests/

      - name: Validate Data
        run: |
          python validate_data.py
```

**Example input / output:**

```text
- **Input**: PR opened with changes to both code and data.
- **Output**: GitHub Actions runs the defined steps, providing test results and data validation feedback.
```

