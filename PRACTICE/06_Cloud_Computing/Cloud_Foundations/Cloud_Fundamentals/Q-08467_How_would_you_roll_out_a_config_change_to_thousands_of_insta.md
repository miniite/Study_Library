---
type: "practice"
difficulty: "Deep"
---

# How would you roll out a config change to thousands of instances without a big-bang restart?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Safe Config Rollouts](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When rolling out a configuration change to thousands of instances, we aim to minimize downtime and ensure smooth transitions. The core idea is to use a phased approach that leverages cloud-native tools and practices such as blue-green deployments or canary releases. This method ensures minimal disruption while allowing us to verify the new configuration before full rollout.

The question essentially asks how to implement a rolling update strategy in a cloud environment, which involves several key steps: identifying instances, planning the update sequence, ensuring minimal downtime, and validating changes. We need to make assumptions about the current infrastructure (e.g., whether it’s using containers or VMs) and available tools (like Kubernetes or AWS Elastic Beanstalk).

To achieve this, we can follow a step-by-step process:

Preparation: Assess the number of instances and their current state.
Deployment Strategy Selection: Choose between blue-green, canary, or rolling updates based on requirements.
Update Sequence Planning: Determine which instances will be updated first and last.
Rollout Execution: Gradually update instances while monitoring performance.
Post-Check: Validate the new configuration to ensure stability.

Let’s break this down further:

Preparation

Identify all running instances of the service, using a cloud provider's API or a database that tracks instance states.
Ensure you have a backup plan for data and configurations.

Deployment Strategy Selection

Blue-Green Deployment: Deploy new versions to a separate set of instances while keeping old ones live. Once verified, switch traffic to the new instances.
Canary Release: Gradually roll out the update to a small subset of users or instances, then expand if everything goes well.
Rolling Update: Gradually replace instances one by one.

Update Sequence Planning

For blue-green, ensure both sets are identical initially and then switch traffic.
For canary releases, define a percentage of the user base or instances that will receive the new version first.
For rolling updates, plan to update a small batch of instances at a time.

Rollout Execution

Use automation tools like Terraform or CloudFormation for blue-green deployments.
Implement canary releases via A/B testing frameworks or by manually routing traffic.
Use Kubernetes or similar orchestration tools for rolling updates, where you can configure the update strategy (e.g., 10% at a time).

Post-Check

Monitor logs and metrics to ensure no issues arise post-update.
Validate functionality and performance before fully switching over.

**Comparison:**

```
Deployment Strategy	Description	Example
Blue-Green	Deploy new version to a separate set of instances.	Kubernetes Ingress routing, AWS ALBs
Canary Release	Gradually roll out the update to a small subset of users or instances.	A/B testing frameworks, manual traffic routing
Rolling Update	Gradually replace instances one by one.	Kubernetes rolling updates
```

**Code:**

```python
# Example of Kubernetes Rolling Update Configuration
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-app
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: example-app
    spec:
      containers:
      - name: example-container
        image: new-image:latest
```

**Example input / output:**

```

Input: A list of instances to update.

JSON
Copy
{
  "instances": [
    { "id": 101, "status": "running", "config_version": "v1" },
    { "id": 102, "status": "running", "config_version": "v1" },
    { "id": 103, "status": "running", "config_version": "v1" }
  ]
}

Output: Updated instances with the new configuration.

JSON
Copy
{
  "instances": [
    { "id": 101, "status": "running", "config_version": "v2" },
    { "id": 102, "status": "running", "config_version": "v2" },
    { "id": 103, "status": "running", "config_version": "v2" }
  ]
}
```

**Data:**

```json
{
  "instances": [
    { "id": 1, "status": "running" },
    { "id": 2, "status": "running" },
    { "id": 3, "status": "running" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Identify Instances
6. Prepare Blueprint
7. Select Deployment Strategy
8. Is Blue-Green?
9. Deploy New Version toSeparate Set of Instances
10. Is Canary Release?
11. Gradually Roll Out the Updateto a Subset
12. Replace Instances Gradually
13. Monitor and Validate

