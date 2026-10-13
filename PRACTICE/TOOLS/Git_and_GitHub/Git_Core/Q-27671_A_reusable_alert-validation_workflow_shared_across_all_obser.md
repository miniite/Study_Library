---
type: "practice"
difficulty: "Deep"
---

# A reusable alert-validation workflow shared across all observability repos was updated and silently broke rule deploys in repos pinned to @main. How do you version reusable workflows so a central change can't break pinned consumers, and how do you roll a fix org-wide?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reusable Workflows](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reusable_Workflows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a reusable alert-validation workflow breaking rule deploys in observability repos that are pinned to @main, we need to ensure that any changes to shared workflows are versioned and can be rolled out without disrupting consumers. This involves creating a robust versioning strategy for reusable workflows and implementing a mechanism to manage these versions across the organization.

The core challenge is to decouple central updates from breaking local deployments while ensuring that all affected repos get the necessary fixes in a timely manner. We need to consider both technical and organizational aspects to achieve this goal effectively.

#### Triage
Question Analysis: The problem revolves around managing versioned reusable workflows to prevent changes from breaking pinned consumers, and implementing an efficient rollback strategy.

#### Assumptions
We are using Git/GitHub for source control management.
Observability repos follow a workflow-based structure where rules are defined via these workflows.
There's a central repository containing the reusable workflows.

#### First Principles Solution
Versioning Reusable Workflows: Introduce semantic versioning (e.g., v1, v2) for each update to the alert-validation workflow. This ensures that any changes are clearly labeled and can be tracked over time.
Branch Management: Use branches in Git to manage different versions of workflows. Create a branch main/v1 for version 1 of the workflow, and subsequent updates will create new branches like main/v2.
Deprecation Policy: Establish a deprecation policy that defines how long existing consumers can continue using old versions before they are forced to update.
Rollout Strategy:
Implement automated testing to validate changes in CI/CD pipelines.
Use release notes and changelogs to document updates, making it easier for stakeholders to understand the implications of a new version.
Create a deployment checklist that includes verifying that all dependent repos have been updated or are aware of the impending change.
Detailed Implementation

Semantic Versioning:

Release v1
Deploys v1
Stable period for consumers toupdate
Decide on deprecationtimeline
Create and deploy v2
Communicate changesthrough channels (email, Slack)

Branch Management:

SHELL
git checkout -b main/v1
# Make necessary changes
git commit -am "Update alert-validation workflow to v1"
git push origin main/v1

git checkout -b main/v2
# Make necessary changes
git commit -am "Update alert-validation workflow to v2"
git push origin main/v2

Automated Testing: Ensure that each version of the workflow is thoroughly tested in a CI/CD pipeline before being merged into main/v1 and subsequently deployed.

Deprecation Policy: Define clear rules for when to mark a version as deprecated, such as after 6 months of stable release or upon critical bug fixes.

SHELL
git tag -a v1.0.0 -m "Tagging v1.0.0 for deprecation"

Rollout Strategy: Notify all relevant repositories about the update and provide them with the necessary changesets to apply.

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Closing Summary
By implementing semantic versioning, branch management, automated testing, and a clear deprecation policy, we can manage updates to reusable workflows effectively. This ensures that central changes do not inadvertently break pinned consumers while providing mechanisms for rolling out fixes.

#### Common Mistakes
Failing to maintain a changelog or release notes.
Not having a consistent deprecation timeline.
Ignoring automated testing, leading to undetected bugs in production.
Lack of clear communication and rollback plans.

In follow-ups, we might need to discuss how to handle emergency fixes, the impact on CI/CD pipelines, and ensuring that all stakeholders are aware of version changes. Additionally, exploring alternative solutions like using a separate repository for workflows or employing more advanced branching strategies could also be beneficial.

**Comparison:**

| Version | Description | Deprecation Date |
| ------- | ----------- | ---------------- |
| v1      | Initial release | 2024-05-01 |
| v2      | Updated alert-validation workflow | TBA |
| Task | Status |
|------|--------|
| Automated testing passed. | ✔️ |
| Release notes created and distributed. | ✔️ |
| Communication channels notified of the update. | ✔️ |

**Code:**

```shell
# Example Git commands for creating branches and committing changes
git checkout -b main/v1
git commit -am "Update alert-validation workflow to v1"
git push origin main/v1

git checkout -b main/v2
git commit -am "Update alert-validation workflow to v2"
git push origin main/v2
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Update `main/v1` to `main/v2` and push changes. | Successfully updated workflow in `main/v2`. |
| Notify all repos pinned to `@main`. | Repositories received notification about the update. |

**Data:**

```json
{
  "reusable-workflow": {
    "version": "v1",
    "description": "Alert validation workflow for version 1"
  },
  "repository": {
    "name": "observability-repo-1",
    "branch": "main/v1",
    "last-synced-at": "2023-10-01T12:00:00Z"
  }
}
```

**Flow:**

1. Central Workflow Repository
2. Create v1 branch
3. Test v1 in CI/CD
4. Tag v1 for deprecation
5. Notify consumers of update
6. Publish v2 updates to all repos

