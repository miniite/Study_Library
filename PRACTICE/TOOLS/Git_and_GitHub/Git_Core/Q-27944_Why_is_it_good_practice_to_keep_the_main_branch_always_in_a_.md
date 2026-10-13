---
type: "practice"
difficulty: "Intermediate"
---

# Why is it good practice to keep the main branch always in a 'buildable pipeline' state, and how do PR checks protect that for a data team?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Pull Requests](<../../../../TOOLS/Git_and_GitHub/Git_Core/Pull_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure reliable and reproducible builds in any software project, including those handled by data engineers, it is crucial to maintain the main branch always in a 'buildable pipeline' state. This means that at all times, the main branch should be stable, tested, and ready for deployment. Keeping the main branch in such a state helps prevent issues from reaching production and ensures that code changes are validated before they integrate into the mainline.

GitHub Pull Requests (PRs) play a vital role in protecting this buildable pipeline state by enforcing checks and validations through automated tests, linting, and other quality gateways. In essence, PRs act as safety nets to ensure that any proposed changes are thoroughly vetted before merging them into the main branch. This process helps maintain code quality and reduces potential integration issues.

#### Triage

The question asks about two aspects: (1) why it's important to keep the main branch in a 'buildable pipeline' state, and (2) how PR checks protect that for a data team. The key idea is that maintaining a stable main branch ensures reliability, while PR checks help prevent integration issues.

#### First-Principles Solution
Why Keep Main Branch Buildable?
Stability and Reliability: A buildable main branch signifies that the codebase can be deployed at any time without causing failures or breaking changes.
Reproducibility: Ensuring the main branch is always in a good state allows for easy debugging, testing, and deployment of new features or bug fixes.
Continuous Integration (CI): A buildable pipeline supports continuous integration by ensuring that every commit to the main branch triggers automated builds and tests.
How PR Checks Protect Main Branch
Automated Tests: PRs are required to pass all necessary automated tests before merging into the main branch. These can include unit tests, integration tests, and regression tests.
Code Linting: Linters ensure that code adheres to coding standards and best practices, reducing potential runtime errors.
Security Scans: Security tools check for vulnerabilities in dependencies or code, helping prevent security breaches.
Documentation Checks: Ensure that documentation is updated alongside the code changes.
Step-by-Step Solution

Setup a CI Pipeline:

Define and configure automated build and test jobs to run on each commit to the main branch.
Use tools like Jenkins, GitLab CI, or GitHub Actions for this purpose.

Create PRs:

Every time someone wants to merge code into the main branch, they must create a PR first.
The PR triggers all defined checks and tests automatically.

Implement Checks in PRs:

Add steps in your CI/CD pipeline to run the necessary checks as part of the PR process.
Ensure that only passing PRs can be merged into the main branch.

#### Closing Summary

In summary, maintaining a buildable main branch ensures stability and reliability for data pipelines. PR checks through automated tests, linters, security scans, and documentation checks help protect this state by ensuring every change is thoroughly vetted before it reaches production.

#### Common Mistakes/Follow-Ups
Skipping Tests in PRs: Failing to run all necessary tests can lead to undetected bugs reaching the main branch.
Manual Merging: Allowing direct merges without creating a PR bypasses automated checks and can introduce issues.
Outdated Checks: Ensuring that the checks are up-to-date and comprehensive is crucial; otherwise, they may not catch all potential problems.

**Comparison:**

```
PR Status	Description
Buildable	Main branch is always in a good state for deployment.
Automated Tests Pass	Ensures no bugs are introduced.
Linting Passes	Code adheres to coding standards and best practices.
```

**Code:**

```yaml
# Example GitHub Actions configuration for CI/CD pipeline

name: Data Pipeline CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.8'
    - run: pip install -r requirements.txt
    - name: Run tests
      run: pytest --cov=.
    - name: Check code style
      run: flake8 .
    - name: Secure scanning
      uses: snyk/trivy@v0.21.3
      with:
        image: python:3.8
```

**Example input / output:**

```

Input:

CODE
Copy
Commit changes to main branch

Output:

CODE
Copy
Triggered PR #457 with automated tests and linters. Checks passed, merging into main.
```

**Data:**

```json
{
  "branch": "main",
  "status": "buildable",
  "last_commit_hash": "abc123",
  "automated_tests_passed": true,
  "linting_passes": true,
  "security_scans": {
    "vulnerabilities_found": false,
    "details": []
  },
  "pr_number": 456
}
```

**Flow:**

1. Code Commits
2. Trigger PR
3. Automated Tests
4. Linting
5. Security Scans
6. Merge into Main Branch
7. Automated Tests Pass?
8. Yes, Merge
9. No, Revert Changes

