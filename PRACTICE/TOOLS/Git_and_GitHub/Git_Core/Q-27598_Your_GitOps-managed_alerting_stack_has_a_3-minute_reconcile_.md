---
type: "practice"
difficulty: "Expert"
---

# Your GitOps-managed alerting stack has a 3-minute reconcile interval, so a git revert of a bad routing change isn't fast enough during a paging flood. How do you reconcile the Git-as-source-of-truth invariant with the need for a sub-minute suppression during an active incident?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reverts in GitOps Rollouts](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reverts_in_GitOps_Rollouts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To address the challenge of reconciling the Git-as-source-of-truth (GitOps) invariant with the need for rapid sub-minute suppression during an active incident, we can implement a multi-tiered approach that leverages GitOps principles while allowing for immediate action during critical incidents. This involves creating separate staging environments and using custom workflows to bypass or suppress changes temporarily.

#### Triaging the Question

The question is probing how to maintain the integrity of your Git-based configuration management system (GitOps) when real-time suppression of a bad routing change is needed, especially in scenarios like paging floods where speed is critical. Key assumptions include:

The current reconcile interval is 3 minutes.
You are using Git as the source of truth for all configuration.
There is an urgent need to suppress changes during active incidents.

#### First Principles Solution
Separate Staging Environment: Create a dedicated staging environment where changes can be temporarily applied or suppressed without affecting the production codebase.
Custom Workflows: Develop custom GitOps workflows that allow for immediate suppression of problematic changes, bypassing normal commit and reconcile intervals.
Automated Alerts & Notifications: Implement automated alerts and notifications to notify operators immediately when a change might impact service stability, allowing quick manual intervention.
Step-by-Step Solution

Design the Staging Environment:

Set up a staging environment where changes can be tested without affecting production.
Ensure this environment mirrors your production setup closely enough for accurate testing but is isolated to prevent accidental production deployment.

Custom Workflow Development:

Develop a custom workflow that allows operators to manually trigger a rollback or suppression of the problematic change.
This could involve creating a specific Git branch, labeling changes as "suppressed," and triggering an immediate reconciliation in the staging environment before applying to production.

Automated Alerts & Notifications:

Configure monitoring tools to send real-time alerts when certain thresholds are breached (e.g., high error rates, increased latency).
Integrate these alerts with your incident management system so operators can quickly identify and address issues.

Manual Override Mechanism:

Provide a manual override mechanism that allows operators to bypass normal commit intervals if necessary.
For example, allow operators to manually git revert changes directly in the staging environment without waiting for the 3-minute reconcile interval.

#### Closing Summary

By implementing separate staging environments and custom workflows, you can maintain GitOps principles while ensuring rapid response during critical incidents. This approach balances the need for real-time suppression with the integrity of your source-of-truth system, providing a robust solution that is both flexible and reliable.

#### Common Mistakes
& Follow-Ups
Mistake 1: Failing to isolate staging environments properly, leading to accidental production deployments.
Mistake 2: Overlooking automated alerts and notifications, resulting in delayed responses to incidents.

#### Follow-Up
Discuss potential integration with other CI/CD tools and how they can be used to further streamline the workflow.

**Comparison:**

```
Workflow Stage	Action	Time Interval
Pre-incident	Test changes	3 minutes
Incident	Manual rollback	sub-minute
```

**Code:**

```bash
# Example script for manual override mechanism

if [ "$ACTION" == "rollback" ]; then
    git checkout staging
    git revert --no-commit BAD_CHANGE_COMMIT_HASH # Revert problematic change without commiting
    git add .
    git commit -m "Rollback of bad routing change"
    git push origin staging
    ./scripts/reconcile.sh # Trigger immediate reconciliation in the staging environment
fi
```

**Example input / output:**

```

Input: A bad routing change (BAD_CHANGE_COMMIT_HASH) is causing issues in production.

Output:

Operators manually identify the problematic commit.
They run a custom script to revert and push the changes to staging.
The reconcile.sh script triggers an immediate reconciliation in the staging environment.
Once verified, operators manually apply the changes to production or keep them suppressed as needed.

Dry-Run:

CODE
Copy
$ git checkout staging
$ git revert --no-commit BAD_CHANGE_COMMIT_HASH
$ git add .
$ git commit -m "Rollback of bad routing change"
$ git push origin staging
$ ./scripts/reconcile.sh
```

**Data:**

```yaml
# Example configuration for manual override
staging:
  branches:
    - main
  overrides:
    - if: "BAD_CHANGE_COMMIT_HASH"
      action: rollback
      interval: sub-minute
```

**Flow:**

1. Identify Problematic Change
2. Manual Revert
3. Push to Staging
4. Immediate Reconciliation inStaging
5. Verify Changes
6. Apply to Production or KeepSuppressed

