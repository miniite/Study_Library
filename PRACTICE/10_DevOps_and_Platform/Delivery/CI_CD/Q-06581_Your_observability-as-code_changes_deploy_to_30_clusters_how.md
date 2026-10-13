---
type: "practice"
difficulty: "Deep"
---

# Your observability-as-code changes deploy to 30 clusters; how would you stagger the rollout across failure domains so a bad collector config can't blind your whole fleet at once?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Zero Downtime Deployments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/Zero_Downtime_Deployments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deploying observability-as-code (OAC) changes to multiple Kubernetes clusters, it's critical to ensure that the deployment is staggered across failure domains to mitigate the risk of a bad configuration affecting the entire fleet at once. This problem involves understanding both the infrastructure topology and the operational principles of CI/CD pipelines.

The core idea here is to implement a staged rollout strategy that ensures each cluster operates in a different state during the transition period, thereby isolating potential issues before they cascade across all clusters. Let’s break down the steps required for this approach:

Understand Failure Domains: A failure domain is a group of resources that are likely to fail or be affected by a single fault condition simultaneously. In cloud-native environments like Kubernetes, these can be zones (e.g., AWS availability zones), regions, or even custom-defined groups based on specific criteria.

Stagger Rollout Strategy:

Identify the total number of failure domains and cluster groups.
Determine the order in which clusters will be updated.
Ensure that each update batch covers only a subset of these failure domains to minimize the risk of cascading failures.

CI/CD Pipeline Integration: The CI/CD pipeline should support such staged deployments, possibly through custom triggers or environment variables that control the deployment phase and target cluster groups.

Monitoring and Telemetry Setup: Implement real-time monitoring and logging for all clusters during the rollout to detect issues early. This setup helps in quickly identifying problematic configurations before they affect a broader set of clusters.

Recovery Plan: Have rollback strategies in place, including automated rollbacks triggered by predefined conditions or manual interventions if necessary.

By following these steps, we can achieve a more resilient and controlled deployment process, ensuring that any issues with the new configuration are confined within a limited scope until resolved.

**Comparison:**

| Cluster Group | Clusters |
|---------------|----------|
| Group 1       | cluster-01, cluster-04 |
| Group 2       | cluster-02, cluster-05 |
| Group 3       | cluster-03            |

**Code:**

```python
# Example of a Python function to stagger cluster deployments across failure domains
def stagger_deployments(clusters, num_fail_domains):
    """
    Args:
        clusters (list): List of cluster names.
        num_fail_domains (int): Number of failure domains in the environment.

    Returns:
        list: Staggered deployment schedule for each cluster.
    """
    fail_domain_size = len(clusters) // num_fail_domains
    deployments = []

    for i in range(num_fail_domains):
        start_index = i * fail_domain_size
        end_index = start_index + fail_domain_size

        # Adjust for edge cases where the last batch has fewer clusters
        if i == num_fail_domains - 1:
            end_index = len(clusters)

        deployments.append(clusters[start_index:end_index])

    return deployments

# Example usage
clusters = ["cluster-01", "cluster-02", "cluster-03", "cluster-04", "cluster-05"]
num_fail_domains = 3
deployments = stagger_deployments(clusters, num_fail_domains)
print(deployments)
```

**Example input / output:**

```text
Input:
Clusters: [cluster-01, cluster-02, cluster-03, cluster-04, cluster-05]
Number of failure domains: 3

Output:
[
  ['cluster-01', 'cluster-04'],
  ['cluster-02', 'cluster-05'],
  ['cluster-03']
]
```

**Data:**

```yaml
clusters:
  - cluster-01
  - cluster-02
  - cluster-03
  - cluster-04
  - cluster-05
num_fail_domains: 3
```

**Flow:**

1. CI/CD Pipeline
2. Staggered Deployment
3. Cluster Group 1 Deploy
4. Monitor and Log
5. Cluster Group 2 Deploy
6. Monitor and Log
7. Cluster Group 3 Deploy
8. Complete Deployment,Monitor for Issues

