---
type: "practice"
difficulty: "Intermediate"
---

# A frontend PR needs different CI than a backend PR in the same monorepo. How do you structure workflows with path filters and reusable workflows so each app only runs its relevant checks?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Monorepo CI Optimization](<../../../../TOOLS/Git_and_GitHub/Git_Core/Monorepo_CI_Optimization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to ensure that frontend pull requests (PRs) trigger different CI pipelines than backend PRs in a monorepo. By leveraging GitHub Actions' path filters and reusable workflows, we can achieve this with minimal configuration overhead. The key is to define specific paths for each application type within our monorepo and use these paths as triggers for the respective CI jobs.

First, let’s break down what the interviewer might be probing:

Understanding of GitHub Actions workflow structure.
Knowledge of path filters in CI/CD pipelines.
Experience with reusable workflows.
Ability to design a scalable solution that minimizes redundancy.

#### Assumptions
include:

The monorepo contains both frontend and backend applications.
Each application has its own set of CI steps (e.g., linting, testing).
We want to avoid unnecessary checks on the other type of code when making changes.
Step-by-Step Solution
1. Define Reusable Workflows

We start by creating reusable workflows for common CI tasks such as linting and running tests. These can be stored in a dedicated directory within our monorepo or in an external repository if they are shared across multiple projects.

YAML
# .github/workflows/lint.yml
name: Lint

on:
  pull_request:
    branches: [main]
    paths:
      - 'frontend/**'
      - '!**/backend/**'

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      # Steps to run linter for frontend code
YAML
# .github/workflows/test.yml
name: Test

on:
  pull_request:
    branches: [main]
    paths:
      - 'backend/**'
      - '!**/frontend/**'

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      # Steps to run tests for backend code
2. Define Specific Workflows

Next, we define specific workflows that utilize the reusable tasks based on the path of the modified files.

YAML
# .github/workflows/frontend-tests.yml
name: Frontend CI

on:
  pull_request:
    branches: [main]
    paths:
      - 'frontend/**'
      - '!**/backend/**'

jobs:
  build_and_test_frontend:
    runs-on: ubuntu-latest
    steps:
      # Steps to run frontend-specific checks like lint and test
YAML
# .github/workflows/backend-tests.yml
name: Backend CI

on:
  pull_request:
    branches: [main]
    paths:
      - 'backend/**'
      - '!**/frontend/**'

jobs:
  build_and_test_backend:
    runs-on: ubuntu-latest
    steps:
      # Steps to run backend-specific checks like lint and test
3. Ensuring Clarity and Scalability

Each workflow is designed to be clear and self-contained, making it easy for other team members to understand the intent without having to dig into complex configurations. This approach also ensures that future changes or additions can be made more easily.

#### Closing Summary

By using path filters in GitHub Actions workflows, we ensure that each PR only triggers the relevant checks based on the modified file paths. This not only saves time but also prevents false positives and negatives, leading to a more reliable CI process. The use of reusable workflows further streamlines the setup and maintenance of these CI pipelines.

#### Common Mistakes/Follow-Ups
Misconfiguration of Path Filters: Always double-check path patterns to ensure they match the expected file structure.
Overlooking Edge Cases: Ensure that edge cases, like mixed frontend/backend files in a single PR, are handled appropriately.
Not Utilizing Reusability: Sometimes developers might duplicate steps instead of using reusable workflows, leading to unnecessary complexity.

Would you like me to elaborate on any specific part of the solution or provide additional examples?

**Code:**

```yaml
# Example frontend CI workflow
name: Frontend CI

on:
  pull_request:
    branches: [main]
    paths:
      - 'frontend/**'
      - '!**/backend/**'

jobs:
  build_and_test_frontend:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Run linting
        run: npm run lint
      - name: Run tests
        run: npm run test

# Example backend CI workflow
name: Backend CI

on:
  pull_request:
    branches: [main]
    paths:
      - 'backend/**'
      - '!**/frontend/**'

jobs:
  build_and_test_backend:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Run linting
        run: pipenv run flake8
      - name: Run tests
        run: pipenv run pytest

# Reusable workflow for linting and testing (simplified)
name: Lint and Test

on:
  pull_request:

jobs:
  lint_and_test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Run linting
        run: npm run lint
      - name: Run tests
        run: npm run test

# Path filter in action
name: Lint Frontend

on:
  pull_request:
    branches: [main]
    paths:
      - 'frontend/**'
      - '!**/backend/**'

jobs:
  build_and_test_frontend:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Run linting
        run: npm run lint
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Pull Request
6. Is Path in Frontend?
7. Lint and Run Tests (Frontend)
8. Is Path in Backend?
9. Lint and Run Tests (Backend)
10. Skip Checks

