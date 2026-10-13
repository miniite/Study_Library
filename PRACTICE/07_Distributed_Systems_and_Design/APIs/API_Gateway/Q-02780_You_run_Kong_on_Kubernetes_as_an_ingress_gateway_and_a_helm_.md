---
type: "practice"
difficulty: "Intermediate"
---

# You run Kong on Kubernetes as an ingress gateway and a helm upgrade of the chart rolled a new proxy version that quietly changed a default; walk through how you pin, test, and canary gateway version bumps so a chart upgrade never surprises prod.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Gateway Config Testing and Rollout](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Config_Testing_and_Rollout.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure smooth upgrades and prevent production disruptions when rolling out new versions of an API Gateway like Kong, we need to implement a robust strategy that includes pinning, thorough testing, and canary deployments. This approach helps us identify any issues early on without affecting the entire production environment.

Firstly, let's understand what needs to be done. When upgrading a Helm chart for Kong, it's crucial to handle version bumps carefully. We need to ensure that changes do not introduce regressions or unexpected behavior. The steps involve pinning the current version, performing extensive testing in staging environments, and then rolling out the new version through canary deployments before fully switching over.

**Comparison:**

```
Step	Action
1	Pin current version using Helm upgrade command.
2	Test in staging environment.
3	Gradually roll out to production with canary strategy.
4	Monitor and validate new version before full deployment.
```

**Code:**

```bash
# Example of Helm upgrade command with version pinning
helm upgrade --install kong stable/kong --version 1.35.0
```

**Example input / output:**

```text
Input:
- Current Kong installation: `kong-1.34.0`
- New version to be upgraded to: `kong-1.35.0`

Output:
1. Pin the current version for rollback if needed.
2. Test the new version in a staging environment.
3. Roll out the new version with canary deployment strategy.

Steps:
a) Upgrade Kong using Helm with pinned version.
b) Validate Kong's configuration and functionalities.
c) Migrate data if necessary.
d) Test APIs and services in staging.
e) Gradually increase traffic to the new version (canary).
f) Monitor for any issues or regressions.
g) Fully switch over after successful canary period.
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[Pin Current Version] --> B[Validate Configuration]
4. B --> C[Test in Staging]
5. C --> D[Migrate Data (if necessary)]
6. D --> E[Canary Deploy to Production]
7. E --> F[Monitor for Issues]
8. F --> G(Full Rollout) | Fully switch over after successful canary period
9. F --> H[Rollback] | If issues arise during monitoring

