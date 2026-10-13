---
type: "practice"
difficulty: "Deep"
---

# Your Log Analytics bill is dominated by verbose container logs; how do you use Basic Logs, data collection rules, and transformations to cut cost without losing signal?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Azure Functions and Cosmos DB Tuning](<../../../../TOOLS/Azure/Azure_Platform/Azure_Functions_and_Cosmos_DB_Tuning/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with high costs due to verbose container logs in Azure Log Analytics, the key is to balance logging granularity with operational efficiency. This involves optimizing your data collection process, applying appropriate transformations, and leveraging Basic Logs to focus on necessary signals while minimizing redundant information.

The primary goal is to ensure you collect only the relevant log data for your application’s performance monitoring and troubleshooting needs. Let's break down how we can achieve this using Azure Log Analytics tools: data collection rules, Basic Logs, and log transformations.

#### Triage

First, it's essential to understand that the cost in this context comes from the amount of data being collected and processed by Azure Log Analytics. The question specifically mentions "verbose container logs," which suggests there is a lot of unneeded information being stored. The interviewer might be probing your understanding of how to fine-tune logging configurations to reduce unnecessary costs while maintaining valuable insights.

#### Assumptions
We are working within an existing Azure Log Analytics workspace.
Container logs are currently too verbose, leading to high costs.
The focus is on optimizing the log collection process without compromising critical operational data.

#### First-Principles Solution

Identify and Filter Unnecessary Logs

Use container orchestration tools (like Kubernetes) to configure logging policies that capture only necessary logs. For example, in a Kubernetes cluster, you can set up policies to collect only specific log levels or exclude certain fields from being logged.

Implement Data Collection Rules

In Azure Monitor for Containers, create data collection rules to specify which logs should be ingested into Log Analytics. These rules allow you to filter out unnecessary logs based on criteria such as severity level, namespace, or container name.

Use Basic Logs and Transformations

Leverage Basic Logs in the Log Analytics workspace to query only relevant log events. This allows for more focused querying rather than processing large volumes of data.
Apply transformations using Kusto Query Language (KQL) to further refine the logs, removing noise and extracting key information.

Cost Optimization through Monitoring

Use Azure Monitor's cost optimization features to monitor and manage costs associated with log storage. This includes setting up alerts for high-cost scenarios and reviewing the data retention policies.
Step-by-Step Solution

Review Current Logging Configuration

BASH
# Example: Reviewing Kubernetes cluster logging policy
kubectl edit cm -n <namespace> fluentbit-config

Update Data Collection Rules

JSON
{
  "rules": [
    {
      "ruleId": "<unique-id>",
      "name": "LogCollectionRule",
      "enabled": true,
      "condition": {
        "match": "sev_level > 1 AND namespace = 'my-namespace'"
      },
      "actions": [
        {
          "actionType": "log",
          "resourceId": "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.ContainerService/managedClusters/<cluster-name>",
          "targetLogAnalyticsWorkspaceResourceId": "<workspace-resource-id>"
        }
      ]
    }
  ]
}

Query and Transform Logs Using KQL

KQL
// Example: Querying for critical errors in a specific container
ContainerLogs
| where sev_level == 2 and namespace == 'my-namespace' and containerName == 'my-container'
| project time, level, message, containerId

Review Cost Metrics

BASH
# Example: Reviewing Azure Monitor cost metrics
az monitor metric list --resource <workspace-resource-id> --metric "Log Storage MB"

#### Closing Summary
and Common Mistakes

By implementing a more granular logging strategy, using data collection rules to filter logs effectively, and applying KQL transformations for precise querying, you can significantly reduce the cost of verbose container logs without losing critical operational signals. Common mistakes include over-filtering which could lead to missing important logs or under-filtering that results in excessive costs.

Always ensure a balance between logging verbosity and cost efficiency by regularly reviewing and refining your logging policies.

**Comparison:**

| Rule ID | Name        | Enabled | Condition                     | Action                                                                                       |
|---------|-------------|---------|------------------------------|---------------------------------------------------------------------------------------------|
| 001     | LogCollectionRule | True    | sev_level > 1 AND namespace = 'my-namespace' | { "logAnalytics": { "workspaceResourceId": "<workspace-id>" } }                              |

**Code:**

```bash
# Example: Updating Kubernetes cluster logging policy
kubectl edit cm -n my-namespace fluentbit-config

# Example: Creating data collection rule
az monitor log-analytics workspace data-collection-rule create --name LogCollectionRule --resource-group my-resource-group --workspace-name my-workspace --source Microsoft.ContainerService/MonitoredClusters --condition 'sev_level > 1 AND namespace = "my-namespace"' --action '{"logAnalytics": {"workspaceResourceId": "<workspace-id>"}}'
```

**Example input / output:**

```text
// Example input -> output for KQL query
Input:
ContainerLogs

Output:
| where sev_level == 2 and namespace == 'my-namespace' and containerName == 'my-container'
| project time, level, message, containerId
```

**Data:**

```json
{
  "rules": [
    {
      "ruleId": "log-rule-001",
      "name": "LogCollectionRule",
      "enabled": true,
      "condition": {
        "match": "sev_level > 1 AND namespace = 'my-namespace'"
      },
      "actions": [
        {
          "actionType": "log",
          "resourceId": "/subscriptions/sub-id/resourceGroups/res-group/providers/Microsoft.ContainerService/managedClusters/my-cluster",
          "targetLogAnalyticsWorkspaceResourceId": "<workspace-resource-id>"
        }
      ]
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Review Current Logging Config
4. Is logging too verbose?
5. Update Data Collection Rules
6. Apply KQL Transformations forPrecision
7. Monitor Cost Metrics andReview Policies
8. Keep Current Setup, MonitorCosts

