---
type: "practice"
difficulty: "Deep"
---

# Explain how a single bad config or model-version flip could propagate instantly to your entire inference fleet and cause a global incident. How would you design staged rollout and blast-radius scoping for model and config changes?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Shared Config as Single Point of Failure](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Shared_Config_as_Single_Point_of_Failure.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a distributed system, a single bad configuration or an incorrect model version can indeed propagate quickly throughout your inference fleet, leading to unexpected failures that can cascade into global incidents. The key challenge is ensuring controlled and staged rollouts while minimizing the blast radius of any potential issues.

#### Triaging the Question

The question probes how to manage changes in configurations and model versions in a distributed system to prevent ripple effects. Specifically, we need to design a mechanism for rolling out new versions or configurations that ensures they are introduced in a controlled manner, with minimal disruption. We will focus on staged rollouts and scoping blast radius.

#### Assumptions
The inference fleet consists of multiple instances deployed across different environments (e.g., development, staging, production).
Each instance communicates with central control services to receive updates.
There is a need for rollback mechanisms in case the change leads to unexpected issues.

#### First Principles Solution
Version Control: Implement versioning for both models and configurations to ensure clear tracking and management of different states.
Staged Rollouts:
Canary Deployment: Introduce the new model or configuration to a small subset of instances first to monitor performance and stability.
Blue-Green Deployments: Maintain two identical environments (blue and green) where you can swap between them seamlessly without downtime.
Monitoring & Alerting: Set up robust monitoring for key metrics such as latency, error rates, and resource utilization. Establish alerting thresholds that trigger automatic or manual interventions if issues arise.
Rollback Mechanisms: Ensure the ability to revert quickly in case of failures. This could involve switching back to the previous version or rolling back configuration settings.
Automated Testing: Integrate automated tests (unit, integration, and end-to-end) that run on both new and existing configurations/models before deployment.

#### Example Scenario

#### Cons
ider a scenario where you have an inference fleet deployed across multiple regions, with each region having its own set of instances. Let's walk through the steps for a staged rollout:

Initial Setup:

Define version tags (e.g., v1, v2) for models and configurations.
Set up monitoring dashboards for key metrics like response time and error rates.

Pre-Deployment Checks:

Perform thorough testing on a local or staging environment to catch any issues early.
Ensure all dependencies are compatible with the new version.

Staged Rollout Process:

Canary Deployment: Start by deploying the new model (e.g., v2) to 10% of instances in one region (say, Region A). Monitor closely for any anomalies.
PLAINTEXT

#### Example
Canary Deployment Plan:
- Target: 100 instances (Region A)
- New Model Version: v2
- Monitoring Metrics: Response Time, Error Rate
If successful, gradually increase the rollout to more regions and a larger subset of instances.

Blue-Green Swap:

Once confident in the new model's stability, perform a blue-green swap across all instances.
PLAINTEXT

#### Example
Blue-Green Swap Plan:
- Target: 1000 instances (All Regions)
- Old Model Version: v1
- New Model Version: v2

Rollback Mechanism:

If issues are detected during the rollout, quickly switch back to the previous version.
PLAINTEXT

#### Example
Rollback Plan:
- Target: 800 instances (All Regions)
- Old Model Version: v1
- New Model Version: v2

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Closing Summary
By implementing a staged rollout strategy, monitoring systems, and robust testing pipelines, you can minimize the risk of global incidents caused by model or configuration changes. This approach ensures that any issues are detected early and addressed before they spread to the entire fleet.

#### Common Mistakes

Overlooking Monitoring: Failing to set up proper monitoring leads to late detection of issues.
Insufficient Testing: Not performing thorough testing in staging environments can result in unexpected failures during production rollouts.
Lack of Rollback Plan: Without a clear rollback mechanism, any issue can become critical.

#### Follow-Ups

Discuss the integration with continuous integration/continuous deployment (CI/CD) pipelines.
Explore the use of advanced analytics for proactive anomaly detection.

#### Cons
ider the impact on user experience and how to handle user complaints during rollouts.

**Comparison:**

```
Stage	Action	Metrics to Monitor
1	Identify target	Instance IDs
	instances
2	Apply new version	Response Time, Error Rate
	to subset of
	instances
3	Monitor for issues	Logs, Anomaly Detection
```

**Code:**

```python
# Example Canary Deployment Code Snippet
def canary_deployment(version_tag, target_instances):
    # Step 1: Identify target instances
    target_instances = get_instances_in_region('A')

    # Step 2: Apply new version to a subset of instances
    apply_version_to_subset(target_instances[:int(len(target_instances) * 0.1)], version_tag)

    # Step 3: Monitor for issues
    monitor_instances_for_issues(target_instances[:int(len(target_instances) * 0.1)])
```

**Example input / output:**

```text
Input:
- Version tag: v2
- Target instances: 10% of Region A (10 instances)

Output:
- Instances [1, 2, ..., 9] updated to version v2
- Monitoring dashboard shows no issues for the first hour
```

**Data:**

```json
{
  "target_instances": [
    {"id": 1, "region": "A"},
    {"id": 2, "region": "A"},
    ...
    {"id": 99, "region": "A"}
  ],
  "version_tag": "v2"
}
```

**Flow:**

1. Identify Target Instances
2. Select Subset
3. Apply New Version
4. Monitor for Issues
5. Rollback if Issues Detected
6. Notify Team
7. Continue Monitoring
8. Rollout to All Instances if NoIssues

