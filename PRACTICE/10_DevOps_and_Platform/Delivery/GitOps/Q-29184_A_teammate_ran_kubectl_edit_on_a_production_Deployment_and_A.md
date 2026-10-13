---
type: "practice"
difficulty: "Deep"
---

# A teammate ran kubectl edit on a production Deployment and Argo CD self-heal reverted it mid-incident, worsening the outage — how do you reconcile emergency changes with self-heal safely?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Scaling Argo CD](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Argo_CD_vs_Flux/Scaling_Argo_CD.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're dealing with an incident where a teammate made an emergency change to a production Deployment using kubectl edit, which was then reverted by Argo CD's self-heal mechanism. This situation highlights the tension between immediate operational needs and automated governance mechanisms. To safely reconcile these in GitOps, we need to understand both the nature of the changes and the goals of our GitOps tooling.

#### Triage
What’s Being Probed

The interviewer is probing your understanding of:

The core principles of GitOps.
The importance of immutability and version control in DevOps practices.
How to safely handle emergency changes without disrupting automated processes.
Strategies for ensuring that both human judgment and automation work cohesively.

#### Assumptions
Argo CD is being used as the primary GitOps tool.
The kubectl edit change was significant enough to cause an outage.
The self-heal mechanism of Argo CD is configured to revert any changes that deviate from the desired state stored in the repository.

#### First Principles Solution

To address this issue, we need to adopt a multi-layered approach:

Prevent Direct Edits: Ensure that direct kubectl edit operations on production resources are restricted or logged and reviewed.
Emergency Change Handling: Establish a process for handling emergency changes without disrupting the automated self-heal mechanism.
Automated Rollbacks with Grace Periods: Implement strategies to manage rollbacks while minimizing their impact.
Collaboration Between DevOps Teams: Foster communication between teams to ensure that critical changes are planned and coordinated.
Detailed Steps

Prevent Direct Edits:

Enforce a policy where direct kubectl edits on production clusters are disallowed unless explicitly approved by a specific team.
Implement RBAC policies in Kubernetes to restrict such actions, ensuring only authorized personnel can make such changes.

Emergency Change Handling:

Create an "emergency" namespace or label for resources that require rapid modifications without triggering the self-heal mechanism.
Use Argo CD's labels and selectors to exclude these resources from the automated reconciliation process during emergencies.

Automated Rollbacks with Grace Periods:

Configure Argo CD to allow for manual approval of rollbacks, ensuring that changes can be undone but only after thorough review.
Implement a grace period before automatic self-heal kicks in, giving human operators time to react and apply necessary fixes.

Collaboration Between DevOps Teams:

Establish a clear communication pipeline between the development and operations teams to ensure that critical changes are communicated promptly.
Use CI/CD pipelines to manage all production deployments, ensuring that emergency changes can be integrated seamlessly.

#### Closing Summary

To safely handle this scenario, we must enforce policies around direct edits, establish robust processes for handling emergencies, configure Argo CD with appropriate labels and selectors, and ensure ongoing communication between teams. By doing so, we can maintain the benefits of automated governance while still allowing for quick emergency responses when necessary.

#### Common Mistakes/Follow-Ups
Over-relying on Automation: Automated self-heal mechanisms should be used in conjunction with manual oversight to prevent unintended consequences.
Lack of Communication: Ensuring that all teams are aware of the emergency change process is crucial to avoid conflicts and misunderstandings.
Ignoring Emergency Change Logs: Keeping detailed logs of any emergency changes can help in post-incident analysis and improvement.

**Comparison:**

```
Change Type	Action	Effect
Direct Edit	Forbidden	Ensures no accidental changes.
Emergency Label	Applies to specific resources	Allows manual override for critical changes.
Rollback Grace Period	Configurable	Prevents immediate self-heal actions post-change.
```

**Code:**

```yaml
# Example YAML for Argo CD application with emergency labels
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: example-app
spec:
  source:
    repoURL: https://github.com/example/repository.git
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  project: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    exceptions:
      - group: apps
        kind: Deployment
        name: example-app
        labelSelector: "is-emergency=true"
```

**Example input / output:**

```text
Input:
- Emergency change applied to Deployment `example-app` in namespace `production` via `kubectl edit`.

Output:
- Argo CD self-heal mechanism is configured to exclude resources with label `is-emergency=true`.
- Manual approval process ensures that the emergency change can be rolled back if necessary.
```

**Data:**

```json
{
  "namespace": "production",
  "labelSelector": {
    "matchLabels": {
      "is-emergency": true
    }
  },
  "syncPolicyExceptions": [
    {
      "group": "apps",
      "kind": "Deployment",
      "name": "example-app"
    }
  ]
}
```

**Flow:**

1. Initiate Change
2. Apply Label'is-emergency=true'
3. Review and Approve
4. Apply Changes
5. Self-Heal Excluded
6. Monitor for Issues
7. Rollback if Necessary

