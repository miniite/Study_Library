---
type: "practice"
difficulty: "Deep"
---

# Design branch protection and required checks so a broken alert-config main is nearly impossible (main must always be safe to deploy to on-call), while still allowing an audited emergency break-glass override during a paging storm. How would you record and review those overrides?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Branch Protection Design](<../../../../TOOLS/Git_and_GitHub/Git_Core/Branch_Protection_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design branch protection and required checks in Git & GitHub, we need to ensure that changes to the main branch are robustly validated before deployment, especially when it comes to critical alert configurations. At the same time, we must allow emergency overrides during severe incidents (paging storms) while maintaining auditability of these overrides. This requires a combination of strict checks, automated pipelines, and controlled manual approvals.

First, let's triage the question: The interviewer is probing our understanding of Git workflows, branch protection rules, automated testing, continuous integration/continuous deployment (CI/CD), and security practices for critical environments. We need to ensure that main remains safe and only validated changes are merged into it. Additionally, we must handle emergency overrides in a controlled manner with detailed audit logs.

Step-by-Step Solution

Branch Protection Rules:

Ensure the main branch has strict protection rules: Require status checks (e.g., tests passing), require approvals from specific teams or individuals, and restrict direct push access.

Automated Testing Pipelines:

Implement CI/CD pipelines that run comprehensive tests on every pull request before merging into main. This includes unit tests, integration tests, and system-level validation.

Emergency Override Process:

Define a clear emergency override process with predefined conditions under which an override can be performed.
Use GitHub Actions or similar CI/CD tools to create workflows that handle these overrides while logging detailed information about the override.

Audit Logging:

Record all override operations with timestamps, details of who initiated the override, and the context in which it was done. Store this data securely for future audits.
Detailed Workflow
Branch Protection Rules Implementation:
Yes
No
Yes
No
On Pull Request to main
Is PR approved?
Run CI/CD Pipeline
Abort Merge
Are all tests passing?
Merge PR into main
Revert PR and NotifyMaintainers
Automated Testing Pipelines:

#### Example
YAML snippet for GitHub Actions:
YAML
# .github/workflows/main-validation.yml

name: Validate main Branch

on:
  pull_request_target:
    branches:
      - main

jobs:
  validate_changes:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Set up Node.js
        uses: actions/setup-node@v1
        with:
          node-version: '14'
      - name: Install dependencies
        run: npm install
      - name: Run tests
        run: npm test
Emergency Override Process:
Yes
No
Yes
No
On PagerDuty Alert
Is it a paging storm?
Initiate Emergency Override
Create Pull Request foremergency change
Run CI/CD Pipeline on PR
Are tests passing?
Abort and revert changes
Merge PR into main withmanual approval
Regular Workflow

#### Closing Summary

In summary, we have designed a robust system to protect the main branch by implementing strict branch protection rules, automated testing pipelines, and an emergency override process. The emergency override is managed through predefined conditions and documented changes, ensuring that critical configurations can be updated safely during severe incidents.

#### Common Mistakes
and Follow-Ups
Mistake: Overly permissive branch protection rules or lack of comprehensive tests.

#### Follow-Up
Discuss the importance of regular code reviews and security scans to prevent vulnerabilities from being introduced even with automated pipelines.
Overly Permissive Rules
Lack of Robust Tests
Manual Approval for CriticalChanges
Audit Trail for Overrides

**Comparison:**

| Scenario                | Input (Pull Request to main)                       | Output (Outcome of PR) |
|-------------------------|---------------------------------------------------|-----------------------|
| Normal Workflow         | PR with all tests passing                         | Merged into `main`    |
| Emergency Override      | PR initiated during paging storm                   | Manual approval       |

**Code:**

```yaml
# .github/workflows/main-validation.yml

name: Validate main Branch

on:
  pull_request_target:
    branches:
      - main

jobs:
  validate_changes:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Set up Node.js
        uses: actions/setup-node@v1
        with:
          node-version: '14'
      - name: Install dependencies
        run: npm install
      - name: Run tests
        run: npm test

# .github/workflows/emergency-override.yml

name: Emergency Override Process

on:
  pull_request_target:
    branches:
      - main

jobs:
  emergency_override:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Set up Node.js
        uses: actions/setup-node@v1
        with:
          node-version: '14'
      - name: Install dependencies
        run: npm install
      - name: Run tests
        run: npm test
```

**Example input / output:**

| Scenario                | Input (Pull Request to main)                       | Output (Outcome of PR) |
|-------------------------|---------------------------------------------------|-----------------------|
| Normal Workflow         | PR with all tests passing                         | Merged into `main`    |
| Emergency Override      | PR initiated during paging storm                   | Manual approval       |

**Data:**

```yaml
# Example Data for GitHub Actions Configuration

name: Validate main Branch

on:
  pull_request_target:
    branches:
      - main

jobs:
  validate_changes:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Set up Node.js
        uses: actions/setup-node@v1
        with:
          node-version: '14'
      - name: Install dependencies
        run: npm install
      - name: Run tests
        run: npm test

name: Emergency Override Process

on:
  pull_request_target:
    branches:
      - main

jobs:
  emergency_override:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Set up Node.js
        uses: actions/setup-node@v1
        with:
          node-version: '14'
      - name: Install dependencies
        run: npm install
      - name: Run tests
        run: npm test
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. On Pull Request to main
6. Is PR approved?
7. Run CI/CD Pipeline
8. Abort Merge
9. Are all tests passing?
10. Merge PR into main
11. Revert PR and NotifyMaintainers
12. Yes
13. No
14. Yes
15. No
16. On PagerDuty Alert
17. Is it a paging storm?
18. Initiate Emergency Override
19. Create Pull Request foremergency change
20. Run CI/CD Pipeline on PR
21. Are tests passing?
22. Abort and revert changes
23. Merge PR into main withmanual approval
24. Regular Workflow

