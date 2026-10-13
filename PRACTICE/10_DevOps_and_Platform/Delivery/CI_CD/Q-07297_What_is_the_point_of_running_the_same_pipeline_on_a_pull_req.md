---
type: "practice"
difficulty: "Basic"
---

# What is the point of running the same pipeline on a pull request before it is merged?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Pull Request Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Pull_Request_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In the world of CI/CD, running a pipeline on a pull request (PR) before merging is crucial because it ensures that code changes are validated and tested in an isolated environment. This step acts as a safety net to prevent broken builds or tests from making their way into the main branch. Let's break down why this practice is so important.

**Code:**

```bash
# Example of a CI/CD pipeline configuration for GitHub Actions
on:
  pull_request:
    branches: [main]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Run tests
        run: make test
```

**Example input / output:**

```text
When a pull request is created, the CI/CD pipeline is triggered automatically. For example:
- **Input:** A developer opens a pull request from branch `feature-xyz` into the main branch.
- **Output:** The pipeline runs tests and checks for code quality before any changes are merged.
```

**Data:**

```json
{
  "pull_request_id": 12345,
  "branch_name": "feature-xyz",
  "author": "john_doe",
  "merge_commit_sha": "abc123def456"
}
```

