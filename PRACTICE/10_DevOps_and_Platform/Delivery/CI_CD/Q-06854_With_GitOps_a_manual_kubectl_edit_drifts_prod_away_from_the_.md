---
type: "practice"
difficulty: "Expert"
---

# With GitOps, a manual kubectl edit drifts prod away from the repo; how does the reconciliation loop react, and how do you make drift visible and auditable?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Push vs Pull Deployments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Deployment_Observability/Push_vs_Pull_Deployments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are dealing with the challenge of maintaining consistency between our production Kubernetes environment and our GitOps repository using a reconciliation loop. When manual kubectl edits introduce changes in production that diverge from the desired state defined in our repo, the reconciliation mechanism must detect these discrepancies and ensure they are resolved while making them visible and auditable.

#### Triaging the Question

The core of this question revolves around understanding how GitOps systems handle drifts (discrepancies) in a Kubernetes cluster. The key aspects being probed include:

Detection Mechanisms: How does the system identify that there is a drift?
Reconciliation Logic: What steps are taken to reconcile these discrepancies and bring the production environment back into alignment with the GitOps repo?
Auditing and Visibility: How can we ensure that any changes made through manual kubectl edits are logged, tracked, and audited?

#### Assumptions
We assume a GitOps setup is in place.
The production Kubernetes cluster uses a reconciliation loop to apply desired states from the GitOps repo.
Manual kubectl commands may introduce drift by modifying resources directly.

#### First Principles Solution
Detection of Drift

To detect drift, we rely on two key mechanisms:

Resource Comparison: The system compares the actual state (current state) of Kubernetes resources with their desired states as defined in the GitOps repository.
Event Logging: Any changes made through kubectl commands are logged and recorded for audit purposes.
Reconciliation Logic

The reconciliation loop typically works through these steps:

Polling or Webhook Triggers: The system periodically checks (via polling) or uses webhooks to detect any updates in the GitOps repository.
Resource Comparison: It compares the current state of resources with their desired states from the repo.
Patch Application: For any discrepancies found, it applies patches or full replacements as needed to bring the cluster back into sync.
Validation and Rollback: After applying changes, the system validates the new state and rolls back if necessary due to validation failures.
Auditing and Visibility

To ensure drift is visible and auditable:

Event Logs: All manual kubectl commands are logged with timestamps, details of changes made, and their authors.
Audit Traces: These logs can be correlated with the GitOps repository and reconciliation events to trace any drifts back to specific commits or actions.
Automated Notifications: Alerts and notifications can be set up for significant drifts detected through automated tools.

#### Closing Summary

In summary, while manual kubectl edits introduce a risk of drift in our production Kubernetes cluster, GitOps systems equipped with robust reconciliation loops and comprehensive logging mechanisms can detect these discrepancies, apply necessary fixes to correct the state, and ensure all changes are auditable. Key steps include regular resource comparisons, event logging, and automated validation processes.

#### Common Mistakes/Follow-Ups
Over-reliance on Polling: While effective for simplicity, polling can introduce delays in detecting drifts if not done frequently enough.
Incomplete Logging: Failing to log all relevant details of kubectl commands can make it challenging to trace and audit changes effectively.
Manual Workarounds: Relying too heavily on manual workarounds instead of integrating GitOps practices can lead to inconsistencies.

To mitigate these issues, consider implementing continuous integration/continuous delivery (CI/CD) pipelines that integrate with the Kubernetes cluster and enforce strict policies around direct modifications.

**Comparison:**

```
Drift Detection Mechanism	Description
Resource Comparison	Compares current state with desired state from GitOps repo
Event Logging	Logs all kubectl commands for audit purposes
Validation Rollback	Rolls back changes if validation fails
```

**Code:**

```bash
# Example script for detecting drift using kubectl and comparing states
kubectl get pods -o json > current_state.json
git checkout <branch> && git pull  # Update local repo state
kubectl get pods -o json > desired_state.json
diff current_state.json desired_state.json  # Check for differences
```

**Example input / output:**

```text
# Input: Manual kubectl edit causing pod to crash
kubectl edit deployment mysql

# Output: Drift detected and logged
[2023-10-06T14:57:32Z] User "john.doe" modified the deployment "mysql"
[2023-10-06T14:58:00Z] Drift detected between current state and desired state
```

**Data:**

```json
{
  "currentState": {
    "pods": [
      {"name": "nginx-65b49d7bf8-pq7l5", "status": "Running"},
      {"name": "mysql-864c5f957d-zkzv2", "status": "CrashLoopBackOff"}
    ]
  },
  "desiredState": {
    "pods": [
      {"name": "nginx-65b49d7bf8-pq7l5", "status": "Running"},
      {"name": "mysql-864c5f957d-zkzv2", "status": "Running"}
    ]
  }
}
```

**Flow:**

1. No
2. Yes
3. Pass
4. Fail
5. Manual kubectl edit
6. Detect Drift?
7. Monitor Loop Continues
8. Log and Alert
9. Apply Patches/Replacements
10. Validate State
11. Cluster Synced
12. Rollback Changes

