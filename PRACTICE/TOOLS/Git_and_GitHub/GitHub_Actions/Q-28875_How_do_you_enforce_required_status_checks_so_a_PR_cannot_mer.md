---
type: "practice"
difficulty: "Intermediate"
---

# How do you enforce required status checks so a PR cannot merge until CI, security scan, and a manual QA sign-off all pass?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reusable Workflows](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Reusable_Workflows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce required status checks in GitHub Actions to prevent a Pull Request (PR) from merging unless CI builds successfully, a security scan completes without issues, and a manual QA sign-off is provided, we need to configure specific actions and workflows. The key idea here is that these required checks must all pass before the PR can be merged.

#### Triage

The question asks for a method to ensure certain conditions are met before merging a Pull Request (PR) in GitHub. Specifically, it requires CI builds, security scans, and manual QA sign-offs. These requirements need to be enforced through GitHub Actions workflows. The interviewer is probing your understanding of GitHub Actions, their configuration, and the use of required status checks.

Step-by-Step Solution
Define CI Checks: Set up a GitHub Action that runs tests and builds using a CI tool like Jenkins, GitLab CI/CD, or GitHub's own actions.
Security Scans: Integrate security tools like Snyk, Dependabot, or Trivy to perform static code analysis as part of the PR process.
Manual QA Sign-Off: Implement a manual approval step using GitHub's pull request approvals feature.
Workflow Setup

Let’s walk through setting up these required status checks in GitHub Actions:

1. CI Checks

Create a ci.yml workflow file under .github/workflows/ to run your CI tasks.

YAML
# .github/workflows/ci.yml
name: Continuous Integration

on:
  pull_request:

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Node.js environment
        uses: actions/setup-node@v2
        with:
          node-version: '14'

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test
2. Security Scans

Add a security scan workflow to check for vulnerabilities.

YAML
# .github/workflows/security.yml
name: Security Scan

on:
  pull_request:

jobs:
  static-analysis:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Node.js environment
        uses: actions/setup-node@v2
        with:
          node-version: '14'

      - name: Run security scan
        run: npx snyk test --fail-on-warning
3. Manual QA Sign-Off

Integrate manual approval using GitHub’s pull request approvals feature.

Go to your repository settings and under “Code review,” enable pull request reviews, requiring multiple approving reviewers or a specific reviewer.
Required Status Checks Configuration

In the Settings tab of your repository, navigate to the Status checks section:

Add ci.yml, security.yml, and any other relevant workflows.
Ensure these are set as required status checks before merging PRs.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By configuring these GitHub Actions workflows, we ensure that a Pull Request cannot be merged until all necessary steps—CI builds, security scans, and manual QA sign-offs—are completed successfully. This helps maintain code quality, security, and compliance standards.

#### Common mistakes
include:

Not setting the status checks as required.
Missing steps in your CI or security scan workflows.
Failing to enable pull request reviews properly.

#### Follow-up
questions could involve discussing how to handle false positives from security scans, integrating more complex automated tests, or handling urgent PRs that require exceptions to these rules.

**Comparison:**

```
Workflow	Description
ci.yml	Runs automated tests and builds.
security.yml	Performs static code analysis for security issues.
```

**Code:**

```
.github/workflows/ci.yml

name: Continuous Integration

on: pull_request:

jobs: build-and-test: runs-on: ubuntu-latest

CODE
Copy
steps:
  - name: Checkout code
    uses: actions/checkout@v2

  - name: Set up Node.js environment
    uses: actions/setup-node@v2
    with:
      node-version: '14'

  - name: Install dependencies
    run: npm install

  - name: Run tests
    run: npm test
CODE
Copy

```yaml
# .github/workflows/security.yml
name: Security Scan

on:
  pull_request:

jobs:
  static-analysis:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Node.js environment
        uses: actions/setup-node@v2
        with:
          node-version: '14'

      - name: Run security scan
        run: npx snyk test --fail-on-warning
```

**Example input / output:**

```text
- **Input**: A Pull Request with changes to a repository.
- **Output**: The PR cannot be merged until:
  - `ci.yml` checks pass successfully.
  - `security.yml` shows no critical vulnerabilities.
  - Manual QA sign-off is provided by an approved reviewer.
```

**Data:**

```json
{
  "pull_request_reviews_required": true,
  "required_pull_request_reviews.approving_review.required_approving_reviewer": {
    "user": "alice",
    "user": "bob"
  }
}
```

**Flow:**

1. Checkout Code
2. Set Up Node.js Environment
3. Install Dependencies
4. Run Tests
5. Checkout Code
6. Set Up Node.js Environment
7. Run Security Scan

