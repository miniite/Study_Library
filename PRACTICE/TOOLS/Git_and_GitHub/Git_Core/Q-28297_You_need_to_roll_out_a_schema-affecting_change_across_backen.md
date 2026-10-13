---
type: "practice"
difficulty: "Deep"
---

# You need to roll out a schema-affecting change across backend + frontend in lockstep so neither deploys ahead of the other. How do you use Git branching, PR dependencies, and deploy gating to coordinate?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reverts in GitOps Rollouts](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reverts_in_GitOps_Rollouts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure a schema-affecting change is rolled out in lockstep between backend and frontend, we need to leverage Git branches, pull request (PR) dependencies, and deploy gating mechanisms. The core idea is to maintain synchronization across both codebases while preventing any single team from deploying without the other's approval.

First, let’s break down what needs to be coordinated:

Schema Changes: These must be in sync between backend and frontend.
Code Coordination: Both teams should merge their changes together at a consistent point in time.
Deploy Gating: Ensuring that one deployment does not proceed until the other is also ready.

Let's outline how we can achieve this step-by-step:

Branching Strategy:

Create a develop branch from the main or master branch for ongoing development and feature work.
Use separate branches for schema changes, like schema-backend-v1, schema-frontend-v1.
Merge these into the develop branch when ready.

Pull Request Dependencies:

Set up a mechanism where PRs from both teams depend on each other. For example, a frontend PR cannot be merged or deployed until there is an approved backend schema change.

Deploy Gating:

Use continuous integration and deployment (CI/CD) pipelines that enforce deploy gating.

Let’s go through these steps in more detail:

1. Branching Strategy

Backend:

BASH
git checkout develop
git branch schema-backend-v1

Make schema changes, commit, and push to schema-backend-v1.

Frontend:

BASH
git checkout develop
git branch schema-frontend-v1

Update the frontend to work with the new schema, commit, and push to schema-frontend-v1.

2. Pull Request Dependencies
Set up a dependency in GitHub Actions or GitLab CI/CD where PRs are dependent on each other.
YAML
# Example GitHub Actions workflow for backend
jobs:
  deploy-backend:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Build and test backend
        run: make build && make test
      - name: Ensure frontend PR is merged
        if: github.event.pull_request.head.ref == 'schema-frontend-v1'
        uses: GitHubActionsCommunity/check-pull-request-status@master
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          pr-ref: schema-backend-v1
          event-name: pull_request
3. Deploy Gating
Configure CI/CD pipelines to gate deployments.
YAML
# Example deployment strategy in AWS CodeDeploy
appspec.yml:
  version: 0.0
  os: linux
  files:
    - source: /path/to/backend/code/
      destination: /var/www/app/
  hooks:
    ApplicationStop:
      before_install:
        - file: /var/www/app/deploy_precheck.sh
          timeout: 15
deploy_precheck.sh:
  # Script to check if frontend deployment is ready
  if [ ! -d "/path/to/frontend/code/" ]; then
    echo "Frontend not deployed. Deployment blocked."
    exit 1
  fi

#### Closing Summary
and Common Mistakes/Follow-Ups
Ensure that both teams are aware of the schema change process.
Regularly communicate to avoid delays in deployments.
Monitor for any unexpected merge conflicts or issues due to untested changes.

Some common mistakes include:

Skipping PR dependencies, leading to out-of-sync deployments.
Not properly setting up deploy gating, allowing one team to proceed without the other’s approval.

For follow-ups, consider discussing additional tools like GitOps for more complex deployments and how to handle hotfixes or urgent updates.

**Comparison:**

```
Workflow Step	Backend Branch	Frontend Branch	Notes
Initial Setup	develop	develop	Create branches for schema changes
PR Creation	schema-backend-v1	schema-frontend-v1	Both teams create PRs simultaneously
PR Merging	Develop merged	Develop merged	Ensure both PRs are approved before merging into master
Deploy Check	deploy-backend	deploy-frontend	Use deploy gating to ensure backend only deploys if frontend is ready
```

**Code:**

```yaml
# Example GitHub Actions workflow
jobs:
  deploy-backend:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Build and test backend
        run: make build && make test
      - name: Ensure frontend PR is merged
        if: github.event.pull_request.head.ref == 'schema-frontend-v1'
        uses: GitHubActionsCommunity/check-pull-request-status@master
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          pr-ref: schema-backend-v1
          event-name: pull_request
```

**Example input / output:**

```text
# Example input-output for PR dependency check
GitHub Actions Workflow:
- Backend PR (schema-backend-v1) is created.
- Frontend PR (schema-frontend-v1) must be merged first before the backend can proceed.

Outcome:
- If frontend PR is not ready, deploy-backend step fails with a message: "Frontend not deployed. Deployment blocked."
```

**Data:**

```json
{
  "appspec.yml": {
    "version": "0.0",
    "os": "linux",
    "files": [
      { "source": "/path/to/backend/code/", "destination": "/var/www/app/" }
    ],
    "hooks": {
      "ApplicationStop": {
        "before_install": ["/var/www/app/deploy_precheck.sh"]
      }
    }
  },
  "deploy_precheck.sh": "#!/bin/bash\nif [ ! -d \"/path/to/frontend/code/\" ]; then\necho \"Frontend not deployed. Deployment blocked.\"\nexit 1\nfi"
}
```

**Flow:**

1. No
2. Yes
3. Backend PR Created
4. Frontend PR Created
5. Is Frontend PR Merged?
6. Deploy-Backend Blocked
7. Backend Deployed
8. Notify Backend to Wait
9. Frontend PR Dependent onBackend PR
10. Frontend PR Merged
11. Backend PR Merged
12. Both Deployed Simultaneously

