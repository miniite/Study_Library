---
type: "practice"
difficulty: "Expert"
---

# Your service is deployed by GitOps but an urgent incident needs a change faster than the reconcile loop and PR flow allow; how do you design a safe break-glass path that stays auditable and reconciles back?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Push vs Pull Deployments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Deployment_Observability/Push_vs_Pull_Deployments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are dealing with a service deployed using GitOps principles where an urgent change is needed faster than the standard reconcile loop or PR flow can accommodate. The key challenge here is to design a safe "break-glass" path that allows for immediate changes without compromising auditability and eventually reconciling back into the GitOps pipeline.

The question is really probing our understanding of:

How to handle urgent changes in a system designed for automated and repeatable deployments.
Maintaining robust auditing and logging practices.
Ensuring eventual consistency with the GitOps workflow.

We need to make several assumptions to reason through this problem:

The service is part of a larger infrastructure managed by Kubernetes or a similar orchestration tool.
We are using tools like FluxCD for GitOps deployment.
There may be specific incident response procedures in place that we need to follow.
We have access to logging and monitoring tools.

Let's break down the solution step-by-step:

Step 1: Define the Break-Glass Path

The first step is to establish a dedicated path or mechanism for making urgent changes. This could involve:

Introducing a "break-glass" command in Kubernetes that can manually update a configuration file.
Creating a separate namespace or cluster specifically for handling such incidents.
Step 2: Implement Auditing and Logging

To ensure the change is auditable, we need to:

Log every action taken via the break-glass path.
Record the time, user, and details of the changes made.
Store these logs securely in a centralized logging system (e.g., ELK Stack).
Step 3: Reconciliation Mechanism

Once the urgent change is made, we need to ensure it eventually reconciles back into the GitOps pipeline. This can be achieved by:

Writing a custom script that automatically runs after making changes via the break-glass path.
The script should commit the changes back to the Git repository and trigger the FluxCD process.
Step 4: Incident Response Procedure

Define clear steps for responding to urgent incidents, including:

Notifying stakeholders (e.g., on-call engineers).
Documenting all actions taken during the incident response.
Ensuring compliance with any relevant security or regulatory requirements.

#### Closing Summary
and Common Mistakes

The key takeaway is that while a break-glass path can be necessary for rapid responses to urgent incidents, it must not compromise the overall integrity of the GitOps process. Common mistakes include neglecting proper auditing and logging, failing to document changes effectively, and not having a clear reconciliation strategy.

**Comparison:**

| Step | Action                    | Tool/Procedure           |
|------|---------------------------|--------------------------|
| 1    | Define break-glass path   | Kubernetes `kubectl`      |
| 2    | Implement auditing       | ELK Stack                |
| 3    | Reconciliation mechanism | Custom script, FluxCD    |
| 4    | Document incident         | Incident response plan   |

**Code:**

```bash
# Example custom script for reconciling break-glass changes
#!/bin/bash

# Commit any manual changes back to the Git repository
git add .
git commit -m "Manually applied urgent fix"
git push origin main

# Trigger FluxCD reconcile process
kubectl apply -f /path/to/deployment.yaml
```

**Example input / output:**

```text
Input: Incident report for urgent fix to service A

Output:
- Break-glass command executed manually
- Changes logged in centralized logging system
- Custom script committed changes back to Git repository and triggered FluxCD reconcile process
- Service A is now reconciled into the standard GitOps pipeline
```

**Data:**

```json
{
  "logEntry": {
    "timestamp": "2023-10-01T12:00Z",
    "user": "admin",
    "action": "manually applied urgent fix for service A",
    "details": [
      { "component": "serviceA", "change": "updated configuration" },
      { "component": "serviceB", "change": "fixed critical bug" }
    ]
  }
}
```

**Flow:**

1. Logging
2. Reconciliation
3. No
4. Identify Urgent Change
5. Is GitOps PR/Reconcile LoopSufficient?
6. Use Break-Glass Path
7. Log Changes
8. E
9. Commit to Git Repository
10. G
11. Trigger Reconciliation withFluxCD
12. Document Incident Response

