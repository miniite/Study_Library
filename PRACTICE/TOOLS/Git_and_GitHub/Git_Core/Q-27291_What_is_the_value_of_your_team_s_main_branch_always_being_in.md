---
type: "practice"
difficulty: "Basic"
---

# What is the value of your team's main branch always being in a 'runnable demo' state, and how do PR checks help protect that for an LLM app?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Branching Strategies](<../../../../TOOLS/Git_and_GitHub/Git_Core/Branching_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a software development context, particularly for a Large Language Model (LLM) application, ensuring that your team's main branch always remains in a 'runnable demo' state is of utmost importance. This means that any feature or fix merged into the main branch should result in an application that functions as intended, providing value to end-users without issues. This concept is crucial for maintaining user trust and ensuring that continuous delivery can be sustained.

#### Triaging the Question

The question essentially asks about the significance of keeping the main branch always runnable and how pull request (PR) checks can support this goal. The interviewer is likely probing your understanding of version control, code quality, and Continuous Integration/Continuous Deployment (CI/CD) practices in a software development workflow.

#### Assumptions
Team Workflow: Assume that there are multiple developers contributing to the main branch.
Branching Strategy: Use a typical branching model where feature branches are created from the main branch for new work and merged back into it once complete.
CI/CD Pipeline: The pipeline includes automated tests, linters, and other checks before code is merged.
Step-by-Step Solution
1. Keeping the Main Branch Runnable

When a team’s main branch is always in a 'runnable demo' state, it means that any developer can check out the latest commit on main, run the application locally or deploy it to an environment (like staging), and have a fully functioning version of the app without encountering errors. This provides several benefits:

User Trust: End-users rely on knowing that they can use the latest features or fixes when available.
Continuous Delivery: Ensures that new features can be released to production quickly with minimal risk.
Ease of Debugging: Developers have a consistent environment to test and debug their changes.
2. Role of PR Checks

Pull Request (PR) checks are automated processes that run before any code is merged into the main branch. These checks help ensure that only high-quality, stable code makes it into main. Common PR checks include:

Static Code Analysis: Linters and formatters to maintain consistent coding standards.
Unit Tests: Ensuring individual components work as expected.
Integration Tests: Testing how different parts of the application interact with each other.
End-to-end (E2E) Tests: Running full scenarios that mimic real user interactions.
Security Scans: Detecting vulnerabilities or potential security risks.

These checks help protect the main branch by:

Reducing Bugs: Catching and fixing issues early in the development process.
Maintaining Code Quality: Ensuring that only well-tested, stable code is merged into main.
Preventing Accidents: Avoiding accidental merges of broken or incomplete features.
3. Example Scenario

Imagine a team working on an LLM app where developers are creating new features in feature branches. Each PR will undergo several checks:

BASH
# Sample PR Workflow
git checkout -b feature/new-feature main
# Develop and test the feature
# Push to remote repository
git push origin feature/new-feature

When a developer opens a PR, GitHub Actions (or another CI/CD service) automatically runs tests. If all tests pass:

BASH
# Assuming success
# Merge into main branch
git checkout main
git pull origin main
git merge --ff-only feature/new-feature
git push origin main

If any test fails, the PR is marked as failed and remains open until the issues are resolved.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Keeping the main branch always in a 'runnable demo' state through rigorous PR checks is vital for maintaining code quality and ensuring that new features can be released reliably. PR checks like static analysis, unit testing, integration tests, E2E tests, and security scans are essential tools to safeguard the integrity of main. Common mistakes include skipping checks or merging broken code into main, which can lead to regression issues and frustrated users.

When asked follow-ups, you might be probed on:

Specific tools or services used for PR checks.
How often PRs need to be reviewed in practice.
Strategies for handling conflicts when multiple PRs are merged simultaneously.

**Comparison:**

```
Check Type	Description
Static Analysis	Ensures code adheres to coding standards and linters.
Unit Tests	Verifies individual components or modules work as expected.
Integration Tests	Confirms that different parts of the application interact correctly.
End-to-end Tests	Simulates full user scenarios from start to finish.
```

**Code:**

```bash
# Example PR Workflow script snippet using GitHub Actions

name: Run Tests

on:
  pull_request:

jobs:
  test-and-lint:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [3.8, 3.9]
    steps:
      - uses: actions/checkout@v2
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v2
        with:
          python-version: ${{ matrix.python-version }}
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
      - name: Run tests
        run: |
          pytest
      - name: Run linter
        run: |
          flake8 .
```

**Example input / output:**

```text
Input:
- Feature branch 'new-feature' created from main.
- Developer pushes code to remote repository.

Output:
- GitHub Actions runs PR checks.
- Checks include Code style, Unit tests, Integration tests.
- If all checks pass, PR is merged into main. If any check fails, PR remains open for revisions.
```

**Data:**

```json
{
  "branch": "main",
  "PR": {
    "title": "Add new feature for user authentication",
    "state": "open",
    "checks": [
      { "name": "Code style", "status": "success" },
      { "name": "Unit tests", "status": "success" },
      { "name": "Integration tests", "status": "failure" }
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. All Pass
4. Any Fail
5. Feature Branch Created
6. Merge into Main?
7. Merge to Main
8. Fix Issues and Retry PR
9. Run All Checks
10. Merged Successfully
11. PR Rejected, Fix Issues
12. Continuous DeploymentTriggers
13. Retry PR or Open New One

