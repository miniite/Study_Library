---
type: "practice"
difficulty: "Deep"
---

# A shared GitOps repo is edited by 8 teams and merge queue throughput is now the bottleneck for all deploys. How would you decompose the single repo or the merge queue so one team's slow check doesn't block everyone's deploys?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Concurrency Groups and Merge Queues](<../../../../TOOLS/Git_and_GitHub/Git_Core/Concurrency_Groups_and_Merge_Queues.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a shared GitOps repository being a bottleneck due to multiple teams merging changes, we need to decompose the single repo or the merge queue in a way that minimizes blocking while ensuring consistency and collaboration. This involves understanding the current workflow, identifying potential bottlenecks, and proposing an optimal solution.

First, let's understand what the problem is: With 8 teams contributing to a single GitOps repository, the frequent merges are causing delays, which block everyone else from deploying their changes until conflicts are resolved. The goal is to streamline this process without sacrificing the benefits of centralized GitOps practices.

To tackle this, we can explore several strategies:

Decompose the Single Repo: Splitting the single large repo into smaller, more manageable sub-repos can help reduce merge conflicts and improve throughput.
Optimize Merge Queue: Improving the efficiency of the merge queue can also help by reducing the time teams spend resolving conflicts.

We'll start with a triage to understand which approach is more appropriate for our current situation.

#### Triage
Current Setup: A single large GitOps repository shared among 8 teams.
Bottleneck: Frequent merges due to high team activity, leading to delays in deployments.

#### Assumptions
Teams have different areas of responsibility and work on distinct features or services.
The current merge queue is a bottleneck because it requires manual intervention to resolve conflicts.

#### First Principles Solution

Decompose the Single Repo:

Split the large repo into smaller, logically related sub-repos based on feature or service boundaries.
Ensure that each sub-repo has clear ownership and responsibility within teams.
Use GitOps principles like GitOps pipelines to automate deployments from these sub-repos.

Optimize Merge Queue:

Implement CI/CD pipelines with automated testing and linting to catch issues early, reducing merge conflicts.
Utilize GitHub Actions or similar CI/CD tools to create pre-merge checks that automatically resolve simple conflicts.
Establish a dedicated team for managing the merge queue, which can handle complex conflicts more efficiently.
Decomposing the Single Repo

To decompose the single repo, we need to identify logical boundaries:

Identify Service Boundaries:

Determine which services or features are independent and could be placed in their own repos.
For example, if you have a monorepo containing both front-end and back-end code, consider splitting them into separate repos.

Determine Ownership and Collaboration:

Each sub-repo should have clear ownership and responsibility.
Teams that work on closely related services can collaborate more easily in the same sub-repo but still benefit from reduced merge conflicts due to smaller scope.

Set Up GitOps Pipelines:

For each sub-repo, set up a GitOps pipeline that automates deployments and validation steps.
Use tools like Flux or Argo CD for managing infrastructure as code (IaC) across the sub-repos.
Optimizing Merge Queue

To optimize the merge queue:

Automated Testing:

Introduce CI/CD pipelines with automated tests to catch issues early in the development process.
Use tools like GitHub Actions to run unit tests, integration tests, and linting before merging pull requests (PRs).

Conflict Resolution Tools:

Implement conflict resolution tools that can automatically resolve simple conflicts based on predefined rules.
For complex conflicts, use automated merge strategies or have a dedicated team handle them.

Merge Queue Management:

Create a dedicated team or role for managing the merge queue.
Use tools like GitHub’s merge queues to prioritize and manage PRs more efficiently.

#### Closing Summary

By decomposing the single repo into smaller, manageable sub-repos and optimizing the merge queue through automation and specialized roles, we can significantly reduce merge conflicts and improve overall deployment throughput. This approach ensures that each team has clear ownership while leveraging GitOps principles for efficient collaboration and deployment.

#### Common Mistakes

Overcomplicating the repository structure without a clear logical boundary.
Failing to establish strong CI/CD practices that catch issues early.
Neglecting to involve teams in the transition process, leading to resistance or misalignment.

**Comparison:**

| Strategy               | Description                             |
|------------------------|-----------------------------------------|
| Decompose Repo         | Split monorepo into smaller sub-repos    |
| Automated Testing      | Use CI/CD tools to catch issues early     |
| Conflict Resolution    | Implement automated conflict resolution  |

**Code:**

```yaml
# Example GitHub Actions workflow for automated testing
name: Test and Deploy

on:
  push:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Run tests
        run: npm install && npm test
```

**Example input / output:**

| Input (Existing Setup) | Output (Decomposed Repositories) |
|------------------------|---------------------------------|
| Team A creates PR in main repo | Team A creates PR in backend-api repo |
| Team B merges into main repo causing conflicts | Team B and relevant teams resolve conflicts in backend-api repo before merging |

**Data:**

```json
{
  "repos": [
    {
      "name": "webapp",
      "description": "Front-end web application"
    },
    {
      "name": "backend-api",
      "description": "API services for the web app"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Initiate PR in Sub-Repo
4. Is Conflict?
5. Resolve Conflict
6. Merge into Main Repo
7. Automatically Merge orApprove PR
8. Deploy to Production

