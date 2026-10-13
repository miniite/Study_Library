---
type: "practice"
difficulty: "Deep"
---

# Your org wants an internal shared GitHub Actions workflow that all repos call, but you must be able to update it centrally without breaking pinned versions. How do you version and consume reusable workflows safely?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reusable Workflows](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reusable_Workflows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of maintaining a central, version-controlled shared GitHub Actions workflow that can be updated without breaking existing pinned versions in repositories, we need to carefully plan our strategy around Git versioning and consumption patterns. The key is to ensure both flexibility and consistency across all repositories.

TL;DR:

Use semantic versioning for your workflows.
Create a separate repository for the shared workflows with a clear directory structure.

#### Cons
ume these workflows via uses actions in .github/workflows files, ensuring each repo references a specific version.
Update the central repository while maintaining compatibility.

#### Triaging the Question
The core of this question revolves around managing versions and dependencies of reusable GitHub Actions within an organization. The interviewer is probing your understanding of Git versioning strategies, dependency management, and best practices for shared resources in a distributed environment. Assumptions include that all repositories use GitHub Actions, and the workflow files are stored under .github/workflows.

#### First-Principles Solution

Version Workflow Files: Use semantic versioning (e.g., v1.0, v2.1) for your workflows to ensure clear understanding of the changes made in each update.
Centralized Repository Structure:
Create a repository named, e.g., shared-workflows.
Organize workflow files into directories that match their purpose or functionality (e.g., ci, deploy, etc.).
Updating and Consuming Workflows:
In your main repositories, reference the workflows via paths in your .github/workflows files.
Use tags to pin specific versions: For example, if a repository pins a version by specifying uses: shared-workflows/ci@v1.0, it will always use that exact version.
Automated Updates and Rollbacks:
Implement a mechanism for automated updates where new versions are deployed without breaking existing pinned references.
Ensure the central repository can maintain backward compatibility or provide clear deprecation paths.

Step-by-Step Solution:

Setup Central Workflow Repository:

BASH
git clone https://github.com/org/shared-workflows.git
cd shared-workflows

Create Versioned Workflows:

For example, a CI workflow might be structured as ci/v1.0/ci.yml.
Ensure each version has clear documentation and a changelog.

#### Cons
ume in Repositories:

YAML
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: org/shared-workflows@v1.0/ci
        with:
          token: ${{ secrets.GITHUB_TOKEN }}

Automate Version Management:

Use a CI/CD pipeline to automatically bump the version and update the changelog.
Implement a process for testing changes before deploying them centrally.

#### Closing Summary
By using semantic versioning, separating workflows into a central repository with clear directories, and pinning specific versions in repositories that consume these workflows, we ensure both flexibility and consistency. This approach minimizes disruptions during updates while maintaining compatibility.

#### Common Mistakes/Follow-Ups

Failing to use tags correctly can lead to breaking changes.
Not having a robust versioning strategy can cause confusion when multiple workflows are updated simultaneously.
Lack of automated testing and rollback mechanisms increases the risk of introducing bugs.

**Comparison:**

```
Workflow Version	Description
v1.0	Initial version
v1.1	Added feature X
v2.0	Major refactoring
```

**Code:**

```yaml
# Example .github/workflows/ci.yml in consumer repository
name: CI

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: org/shared-workflows@v1.0/ci
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
```

**Example input / output:**

```text
# Input: .github/workflows/ci.yml in a consumer repository
name: CI

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: org/shared-workflows@v1.0/ci
        with:
          token: ${{ secrets.GITHUB_TOKEN }}

# Output: Workflow execution in the consumer repository
## Workflow CI on main branch
- Run step 1 using actions/checkout@v3
- Run step 2 using org/shared-workflows@v1.0/ci
```

**Data:**

```yaml
# Example shared-workflows repository structure
shared-workflows/
├── ci/
│   ├── v1.0/
│   │   └── ci.yml
└── deploy/
    └── v2.0/
        └── deploy.yml
```

**Flow:**

1. Central Workflow Repository
2. Update Version?
3. Tag and Commit New Version
4. Notify Consumers
5. Manual Update or AutomatedRollout in Consumers
6. Monitor for Issues

