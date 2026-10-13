---
type: "practice"
difficulty: "Expert"
---

# You want a break-glass path where an on-call engineer can push an emergency alert-suppression to prod but every such override leaves an immutable, reviewed-after-the-fact audit trail in Git. Design the branch protection and reconciliation workflow, and how you'd audit those overrides monthly.

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Branch Protection Design](<../../../../TOOLS/Git_and_GitHub/Git_Core/Branch_Protection_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that emergency alert suppression can be pushed to production by on-call engineers while maintaining an immutable and auditable record of each override in Git, we need a robust branch protection strategy and a post-override reconciliation process. This solution will leverage GitHub's advanced branch protection features and regular audits.

#### Triaging the Question

The core question revolves around designing a mechanism for emergency overrides that are both quick to apply during critical situations but also rigorously auditable afterward. The key points being probed are:

Immediate Response: How to allow on-call engineers to push changes quickly.
Audit Trail: Ensuring every override is recorded and can be reviewed later.
Branch Protection: Configuring GitHub to enforce these rules effectively.

#### Assumptions
We assume the team has write access but requires approval for certain high-risk actions.
The on-call engineer needs a clear path to push changes without manual approval, only needing to follow the preconfigured Git flow.

#### First Principles Solution
Branch Protection Rules: Configure branch protection rules on the main or critical branches (e.g., main, staging) with strict conditions such as requiring pull requests for non-merge commits.
Emergency Override Branches: Create a separate branch, say alert-suppression-override, which allows direct pushes by on-call engineers but is protected to ensure changes are reviewed and merged afterward.
Step 1: Setting Up Branch Protection

In GitHub, go to the repository settings > Branches and set up rules for the main branches:

PLAINTEXT
Branch: main
Protection Rules:
- Require pull requests before merging
- Require approvals from certain team members

For emergency overrides, create a new branch alert-suppression-override with relaxed protection rules:

PLAINTEXT
Branch: alert-suppression-override
Protection Rules:
- Allow direct pushes by on-call engineers
- Enable code checks
- Require status checks to pass before merging
Step 2: Emergency Override Workflow

When an emergency override is needed, the on-call engineer should follow these steps:

Create a New Branch: git checkout -b alert-suppression-override
Make Changes: Commit the necessary changes.
Push Directly to Prod: git push origin alert-suppression-override

GitHub will automatically log this action, and the branch will be protected to prevent further modifications.

Step 3: Post-Override Reconciliation

Once the emergency is handled, a review process should be initiated:

Merge into Main Branch: Create a pull request from alert-suppression-override to main.
Review and Merge: Have approvers review and merge the changes.
Audit Trail: Store the details of the override in a dedicated audit log repository.
Monthly Audit Process

To ensure compliance, set up a monthly audit:

List All Overrides: Run a script to list all commits on alert-suppression-override branches.
Review Details: Manually review each commit for context and necessity.
Document Findings: Log the findings in a shared document or ticket.

#### Closing Summary

This approach ensures that emergency overrides can be pushed quickly by on-call engineers while maintaining an immutable audit trail. The branch protection rules and post-override reconciliation workflow provide the necessary controls to balance speed with accountability.

#### Common Mistakes/Follow-Ups
Lack of Documentation: Ensure detailed documentation is maintained for each override.
Overuse of Overrides: Monitor if overrides are being used unnecessarily, which could indicate broader issues in alerting systems.
Approval Process Skips: Verify that the approval process is followed every time, even during emergencies.

