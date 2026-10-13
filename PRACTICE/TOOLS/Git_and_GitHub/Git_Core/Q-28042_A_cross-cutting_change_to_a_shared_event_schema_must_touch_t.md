---
type: "practice"
difficulty: "Expert"
---

# A cross-cutting change to a shared event schema must touch the ingestion job, the dbt staging model, and the downstream mart atomically, but each area has different CODEOWNERS. How do you balance one atomic Git commit against per-area review without deadlocking the merge?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Concurrency Groups and Merge Queues](<../../../../TOOLS/Git_and_GitHub/Git_Core/Concurrency_Groups_and_Merge_Queues.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a cross-cutting change to a shared event schema that affects multiple areas (ingestion job, dbt staging model, downstream mart), ensuring atomic Git commits while respecting different CODEOWNERS can be challenging. The key is to design and implement a strategy that minimizes merge conflicts and respects the review process without causing deadlocks. This solution involves careful coordination, clear communication, and leveraging GitHub's features.

#### Triage

The question asks how to balance an atomic Git commit against per-area reviews in a scenario where each area has its own CODEOWNERS. The core challenge is ensuring that changes are committed as one unit but still undergo separate reviews and approvals from different stakeholders. This involves understanding the workflow, communication channels, and leveraging GitHub's features like pull requests (PRs).

#### First-Principles Solution

Define a Clear Workflow:

Establish a clear process for cross-cutting changes that includes coordination among team members.
Use branches to isolate work in progress.

Coordinate with CODEOWNERS:

Before starting the change, notify the relevant CODEOWNERS and agree on a timeline.
Ensure all affected areas are covered by a single PR or multiple coordinated PRs if necessary.

Use GitHub's Pull Request Mechanism:

Create a single branch for the cross-cutting changes.
Push this branch to your repository and open a single PR that covers all affected areas.
Use GitHub Actions or GitOps tools to automate checks and notifications to relevant reviewers.

Ensure Atomic Commit:

Use Git staging area (.gitadd, git commit -a) to ensure changes are committed atomically.
Avoid committing partial changes; all modifications should be part of the same PR.

Communicate and Negotiate Deadlines:

Set realistic deadlines with all relevant stakeholders.
Use comments and discussions in GitHub PRs to clarify any doubts or concerns.

Leverage GitHub's Review Process:

Allow sufficient time for each area to review and provide feedback.
Merge the changes only after all areas have approved, possibly using merge conflicts as a last resort if necessary.

Documentation and Communication Channels:

Maintain detailed documentation of the change process.
Use Slack or other communication tools to keep everyone informed about progress.

#### Closing Summary

By coordinating with CODEOWNERS, leveraging GitHub's pull request mechanism, and ensuring atomic commits, you can balance the need for per-area reviews while maintaining a single commit. Clear communication and well-defined workflows are crucial in preventing deadlocks and ensuring smooth integration.

#### Common Mistakes/Follow-Ups
Not Informing Codeowners: Failing to notify relevant stakeholders can lead to delays and conflicts.
Partial Commits: Committing partial changes can cause merge conflicts and make the review process more complicated.
Overlooking Time Zones: Ensuring that all reviewers are available at a convenient time is important, especially in distributed teams.

#### Example
Workflow
Notify CODEOWNERS:
Notify @data-ingestion, @dbt-models, and @downstream-marts of the upcoming change.
Create PR:
BASH
git checkout -b cross-cutting-change
# Make necessary changes
git commit -a
git push origin cross-cutting-change
Open PR in GitHub:
Open a PR that touches all affected areas with clear descriptions.
Coordinate Reviews:
Use comments and discussions to clarify any concerns.

**Code:**

```bash
# Example Git commands
git checkout -b cross-cutting-change
git commit -a
git push origin cross-cutting-change
```

**Flow:**

1. Initiate Change
2. Notify CODEOWNERS
3. Create and Push Branch
4. Open PR in GitHub
5. Coordinate Reviews
6. Merge After All Approvals

