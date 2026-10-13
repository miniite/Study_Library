---
type: "practice"
difficulty: "Deep"
---

# Design a governance model for who can approve prod merges across 40 teams — CODEOWNERS, branch protection, quorum reviews — that scales without becoming an approval bottleneck.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-tenant GitOps](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Repo_and_Environment_Layout/Multi-tenant_GitOps.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a governance model for approving production (prod) merges in a GitOps environment with 40 teams, we need to balance enforceability and flexibility while ensuring smooth operations. The core components are CODEOWNERS, branch protection rules, and quorum reviews. Let's break down the solution step-by-step from first principles.

#### Triaging the Question

The goal is to create a scalable governance model that prevents merge bottlenecks while maintaining high standards of code quality and security. We'll need to consider multiple layers: who can approve merges (via CODEOWNERS), branch protection rules, and quorum reviews for critical branches.

#### Assumptions
Each team owns specific repositories or parts thereof.
There are varying levels of criticality among the branches (e.g., feature branches vs. prod).
Teams have different SLAs for code merges.
The environment supports CI/CD pipelines and automated tests.

#### First-Principles Solution
Define Ownership with CODEOWNERS: Each repository should have a clear definition of who is responsible for the code using CODEOWNERS. This ensures that every piece of code has at least one approver.
Branch Protection Rules: Implement strict branch protection rules on critical branches to enforce merge approvals and ensure tests pass before merging into prod. For example, you can configure GitHub to require pull requests to be reviewed before merging.
Quorum Reviews for Critical Branches: For highly critical branches (e.g., main, prod), implement quorum reviews where multiple approvers are required to sign off on a merge request.

#### Implementation
Steps

Repository Mapping:

Define which teams own which parts of the codebase.
Assign CODEOWNERS files in each repository that specify team members responsible for different paths or entire repositories.

Branch Protection Configuration:

For non-critical branches (e.g., feature branches), set up less stringent rules such as requiring pull requests with at least one review and automated tests.
For critical branches, enforce stricter requirements like requiring multiple reviews from specific team members and passing all automated tests before merging.

Quorum Reviews:

Define a quorum for each branch that needs it (e.g., 2/4 reviews are required).
Ensure that the quorum includes members from different teams to avoid bottlenecks.
Use tools like GitHub Actions or GitLab CI to automate the review process and notify approvers.
Sample Workflow

Development Branch:

CODEOWNERS: Team A is responsible for feature/* branches.
Branch Protection: Requires at least one review from a member of Team A before merging into this branch.

Staging Branch:

CODEOWNERS: Team B is responsible for the staging environment.
Branch Protection: Requires automated tests to pass and one approval from a member of Team B before merging.

Prod Branch (main/production):

CODEOWNERS: Teams A, B, and C are jointly responsible for critical paths in this branch.
Quorum Review: Requires 2 approvals from different members across these teams to merge into prod.

#### Closing Summary

By combining CODEOWNERS, branch protection rules, and quorum reviews, we can ensure that merges are properly vetted while maintaining scalability. This approach prevents bottlenecks by distributing responsibility among multiple approvers and leveraging automated tools.

#### Common Mistakes
and Follow-Ups
Mistake 1: Over-reliance on manual approvals without automated checks.
Mistake 2: Not defining clear ownership with CODEOWNERS, leading to confusion about who is responsible for what.

#### Follow-Up
Discussing how to handle emergency merges outside the normal process and how to integrate these changes into the governance model.

**Comparison:**

| Repository | Owners                | Branch Protection Rules                     |
|------------|-----------------------|---------------------------------------------|
| main       | Team A, Team B         | Requires 2 approvals from different teams   |
| feature/*  | Team A                | At least one review required               |
| staging    | Team B                | Automated tests and 1 review required      |

**Code:**

```
Example CODEOWNERS file for a repository
Default rule, matches all paths

@team-a

CODE
Copy

```yaml
# Sample branch protection rules in GitHub

{
  "pattern": "main",
  "required_status_checks": {
    "strict": true,
    "contexts": ["ci/circleci:unit", "ci/circleci:integration"]
  },
  "required_pull_request_reviews": {
    "dismiss_stale_reviews": false,
    "require_code_owner_reviews": false,
    "required_approving_review_count": 2
  }
}
```

**Example input / output:**

```

Input:

Repo: 'main'
Teams owning the repo: ['Team A', 'Team B']

Output: Branch protection rules configured for 'main':

Require at least one review from a member of Team A or Team B.
Automated tests must pass before merge.
Quorum reviews set to 2 approvals, each from a different team.

Dry-run example (GitHub Actions):

CODE
Copy
# .github/workflows/codeql-analysis.yml
name: CodeQL Analysis

on:
  pull_request:
    branches: [main]

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v1
        with:
          languages: java

      - name: Perform code scanning
        uses: github/codeql-action/analyze@v1
```

**Data:**

```json
{
  "repos": [
    {
      "name": "repo-1",
      "owners": ["@team-a", "@team-b"]
    },
    {
      "name": "repo-2",
      "owners": ["@team-b", "@team-c"]
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Repository
4. Is it a critical branch?
5. Apply quorum reviews
6. Apply standard branchprotection rules
7. Require 2/3 approvals fromdifferent teams
8. Automated tests + at least onereview required
9. Assign CODEOWNERS files toteams responsible forbranches
10. Ensure `CODEOWNERS` iscorrectly defined andmaintained

