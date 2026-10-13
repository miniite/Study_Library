---
type: "practice"
difficulty: "Basic"
---

# What is a required status check on a PR and how does it enforce quality gates?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pull Request Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Pull_Request_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In the context of Continuous Integration/Continuous Deployment (CI/CD), a "status check" on a Pull Request (PR) ensures that certain conditions are met before merging code into the main branch. This is crucial to enforce quality gates and maintain code integrity, similar to how a traffic light system ensures all vehicles follow safety rules at intersections.

This question primarily probes your understanding of the role status checks play in CI/CD pipelines and their importance in maintaining code quality. The interviewer might also be interested in how you would implement such checks and potential alternatives or drawbacks.

Let’s break it down from first principles:

What is a Status Check on a PR?

A status check on a PR is a mechanism that automatically runs tests, builds, and other validations when changes are submitted for code review. These checks are designed to ensure the merged code meets specific criteria before being deployed, thereby preventing broken or suboptimal code from entering production.

Key Requirements of a Status Check
Code Quality: Ensuring the code adheres to coding standards.
Build Success: Confirming that the code builds without errors.
Test Passes: Running unit tests, integration tests, and other automated tests to validate functionality.
Security Scans: Checking for vulnerabilities in dependencies or code.
Documentation: Ensuring there is no lack of documentation where it’s necessary.
Implementing Status Checks

To implement these status checks effectively:

Integration with CI/CD Pipelines: Use tools like Jenkins, GitLab CI, GitHub Actions, or CircleCI to automate the process.
Define Check Rules: Configure rules in your CI tool to run specific tasks when a PR is opened or updated.
Notify Stakeholders: Send notifications if any check fails so that developers can address issues before merging.

#### Example
Workflow
A developer opens a PR with new code changes.
The CI/CD system automatically runs tests and builds the code.
If all checks pass, the PR is marked as "Ready for Review."
If any check fails (e.g., unit tests fail), the PR remains in a "Blocked" state until issues are resolved.

#### Common Mistakes
Overlooking Security Scans: Failing to include security checks can lead to vulnerabilities.
Inadequate Test Coverage: Only running basic tests may not catch all issues, leading to potential bugs in production.
Ignoring Code Quality: Ensuring coding standards and formatting are maintained is crucial but often overlooked.

#### Closing Summary

Status checks on PRs are essential for maintaining high code quality and preventing regressions. By configuring these checks effectively, teams can ensure that only robust, tested, and secure code makes it to the main branch.

**Code:**

```yaml
# Example GitHub Actions configuration
name: CI

on:
  pull_request:
    branches:
      - main

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
    - name: Check out repository code
      uses: actions/checkout@v2

    - name: Set up Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '14'

    - name: Install dependencies
      run: npm install

    - name: Run tests
      run: npm test

    - name: Build project
      run: npm run build

    - name: Check code style
      run: npm run lint
```

**Example input / output:**

```text
Input:
A PR with a new feature branch 'fix-login' is opened.

Output:
- Code Quality: success (all code follows best practices)
- Build Success: success (build completes without errors)
- Test Passes: success (unit tests and integration tests pass)
- Security Scans: blocked (vulnerabilities found in dependencies)

Result:
The PR remains in a "Blocked" state until the security issues are resolved.
```

**Data:**

```json
{
  "PR": {
    "id": 123,
    "title": "Fix login functionality",
    "branchName": "fix-login",
    "statusChecks": [
      { "name": "Code Quality", "result": "success" },
      { "name": "Build Success", "result": "success" },
      { "name": "Test Passes", "result": "success" },
      { "name": "Security Scans", "result": "blocked" }
    ]
  }
}
```

