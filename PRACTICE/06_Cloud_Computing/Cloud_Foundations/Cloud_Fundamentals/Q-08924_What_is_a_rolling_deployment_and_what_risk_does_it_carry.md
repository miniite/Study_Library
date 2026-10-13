---
type: "practice"
difficulty: "Deep"
---

# What is a rolling deployment and what risk does it carry?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Safe Config Rollouts](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Rolling deployments are a critical strategy in cloud environments, allowing updates to be applied gradually rather than all at once. This approach ensures that services remain available during software updates, mitigating downtime and reducing the risk of catastrophic failures. The core idea is to deploy new versions of an application to a subset of instances first, verify their stability, and then roll out the change incrementally.

#### Triage
What's Being Probed

The question is probing your understanding of rolling deployments as a cloud strategy and the specific risks associated with them. You need to articulate the concept from first principles, discuss its benefits, and delve into potential pitfalls. Assumptions include that you are familiar with cloud services, continuous deployment pipelines, and basic application architecture.

Detailed Solution

Definition: A rolling deployment is a type of deployment strategy where new software versions or configurations are gradually rolled out to an increasing number of instances in the production environment. This approach ensures that only a portion of the user base experiences downtime as each instance gets updated.

Process:

Initiate Deployment: The update starts with deploying the new version to a small, isolated subset of servers or containers.
Monitor Performance: Carefully monitor these instances for any issues such as bugs or performance degradation. Use logging and monitoring tools like Prometheus, Grafana, or CloudWatch.
Gradual Rollout: Once no issues are detected, gradually increase the number of instances running the new version until the entire cluster is updated.
Switch Off Old Version: After all instances have been updated, shut down the old version.

Benefits:

Minimal Downtime: Users experience only brief periods of downtime as updates are applied incrementally.
Risk Mitigation: Lower risk of rolling back changes if issues arise during deployment, as they affect a smaller subset initially.
Flexibility: Allows for more complex deployments and easier rollback strategies.

Risks:

Potential Rollback Issues: If the new version introduces bugs or performance issues that are not detected early in the rollout, rolling back can be challenging if it affects multiple instances simultaneously.
Complexity: Managing partial rollouts requires careful orchestration of deployment pipelines and monitoring.
Resource Utilization: More resources may be needed for initial testing phases to ensure smooth transition.

#### Closing Summary

Rolling deployments are a robust strategy in cloud environments, offering controlled and manageable updates. However, they come with their own set of challenges and risks that need careful management. Understanding these trade-offs is crucial for effective deployment planning.

#### Common Mistakes/Follow-Ups
Ignoring Monitoring: Failing to properly monitor new versions can lead to unnoticed issues.
Insufficient Testing: Not testing thoroughly enough in pre-production environments before initiating a roll-out.
Overcomplicating Deployment Pipelines: Adding unnecessary complexity without clear benefits.
Underestimating Rollback Complexity: Assuming that rolling back will be straightforward, especially if the new version impacts multiple instances.

**Comparison:**

```
Rollout Phase	Percentage	Steps
Initiate Deployment	5-10%	Deploy to subset
Monitor Performance	N/A	Log issues
Gradual Rollout	Incremental	Increase by 20%
Switch Off Old Version	90-95%	Shutdown remaining instances
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Initiate Deployment] --> B[Deploy to a Subset];
    B --> C[Monitor Performance];
    C --> D[Gradually Rollout];
    D --> E[Switch Off Old Version];
    E --> F[Update Complete];
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Deploy new application version to 10% of running instances. | Monitor for 24 hours, then increase deployment to 30%. Repeat until full rollout is complete. |

**Data:**

```json
{
  "rollingDeploymentSteps": [
    {"step": "Initiate Deployment", "description": "Deploy new version to a small subset of instances."},
    {"step": "Monitor Performance", "description": "Observe and log performance metrics for detected issues."},
    {"step": "Gradually Rollout", "description": "Increase the number of updated instances incrementally."},
    {"step": "Switch Off Old Version", "description": "Shut down old version after all instances are updated."}
  ]
}
```

**Flow:**

1. Initiate Deployment
2. Deploy to a Subset
3. Monitor Performance
4. Gradually Rollout
5. Switch Off Old Version
6. Update Complete

