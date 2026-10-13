---
type: "practice"
difficulty: "Expert"
---

# How do you design a policy for AKS node pool consolidation and bin-packing across a fleet to reclaim 30% waste, and what reliability guardrails keep it from causing eviction storms?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ AKS and Container Platform Standards](<../../../../TOOLS/Azure/Azure_Platform/Azure_Functions_and_Cosmos_DB_Tuning/AKS_and_Container_Platform_Standards.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this DevOps Architect/Principal-level interview question, we're tasked with designing an Azure Kubernetes Service (AKS) policy that consolidates node pools to reclaim 30% of waste by implementing bin-packing strategies across a fleet. This involves understanding the trade-offs between resource allocation, reliability, and cost optimization. We'll start by breaking down the problem, then move through a detailed first-principles solution.

Firstly, we need to understand that AKS node pools are managed Kubernetes clusters with specific configurations for nodes. Bin-packing is an algorithm used to optimally pack items into bins or containers. In this context, it means efficiently distributing pods across the available nodes in such a way as to maximize resource utilization while minimizing waste.

The goal here is to reduce wastage by 30%, but we must ensure that our policy doesn't lead to eviction storms, where too many pods are evicted due to insufficient resources. Eviction storms can cause instability and downtime for applications. To achieve this, we'll leverage Azure Policy along with Kubernetes native capabilities.

#### Triaging the Question
Objective: Reduce 30% of waste in node pools by implementing bin-packing strategies.
Scope: Fleet-wide consolidation across multiple AKS clusters.

#### Cons
traints: Avoid eviction storms that could destabilize applications.

#### Assumptions
We have a mixed workload with varying resource demands.
The fleet has sufficient nodes to avoid overcommitment.

#### First-Principles Solution
Step 1: Define Resource Metrics and Utilization

We start by defining the key performance indicators (KPIs) that will guide our policy. These include:

CPU utilization
Memory usage
Disk I/O
Network I/O

Using these metrics, we can identify underutilized nodes and consolidate them effectively.

Step 2: Implement Bin-Packing with AKS Node Pools

To implement bin-packing across multiple node pools, we'll use the kubectl command-line tool in conjunction with Azure CLI. Specifically, we’ll employ the kubectl taint and kubectl label commands to manage node labels and taints.

Here’s a high-level example of how this might work:

SH
# Label nodes for bin-packing
kubectl label nodes <node-name> kubernetes.io/label=bin-packing

# Taint nodes if necessary
kubectl taint nodes <node-name> key=value:NoSchedule
Step 3: Configure Azure Policy

Azure Policy can enforce rules and configurations across your AKS clusters. We’ll create a custom policy definition that ensures proper bin-packing practices are followed.

JSON
{
    "if": {
        "allOf": [
            { "field": "type", "equals": "Microsoft.ContainerService/managedClusters/nodePools" },
            { "not": { "field": "tags.binpackingStatus", "exists": true } }
        ]
    },
    "then": {
        "effect": "audit",
        "details": {
            "message": "Node pool should have bin-packing status tagged."
        }
    }
}

This policy audits node pools that lack the binpackingStatus tag, ensuring they are properly configured for consolidation.

Step 4: Monitor and Eviction Guardrails

To prevent eviction storms, we need to implement robust monitoring and alerting. We can use Azure Monitor along with Kubernetes dashboard tools like Prometheus and Grafana to monitor resource utilization in real-time.

For example:

YAML
# Sample Prometheus Alert Rule
groups:
- name: "EvictionStormAlerts"
  rules:
  - alert: EvictionsPerMinuteHigh
    expr: kubernetes_pod_eviction_total[1m] / on(namespace) group_left(node) rate(kube_node_status_capacity_bytes[5m]) > 0.3
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Evictions per minute are high"

This rule alerts us when evictions exceed a certain threshold, allowing timely intervention.

Step 5: Rollout and Validate

Finally, we’ll rollout this policy across the fleet in stages to ensure stability. We can use Canary deployments to introduce the changes incrementally:

SH
# Introduce policies with canary approach
az policy assignment create --policy {policy-definition} --name "canary-policy"

#### Closing Summary
and Common Mistakes

By following these steps, we can effectively consolidate AKS node pools using bin-packing strategies while maintaining reliability. Key takeaways include:

Proper Metrics: Use comprehensive metrics to identify underutilized nodes.
Policy Enforcement: Leverage Azure Policy for consistent enforcement.
Monitoring & Alerts: Implement real-time monitoring and alerts to prevent eviction storms.

#### Common mistakes
include:

Overcommitting resources, leading to instability.
Failing to validate policies in a staging environment before full rollout.
Not setting up proper monitoring and alerting systems.

**Code:**

```

kubectl label nodes <node-name> kubernetes.io/label=bin-packing kubectl taint nodes <node-name> key=value:NoSchedule

CODE
Copy
```json
{
    "if": {
        "allOf": [
            { "field": "type", "equals": "Microsoft.ContainerService/managedClusters/nodePools" },
            { "not": { "field": "tags.binpackingStatus", "exists": true } }
        ]
    },
    "then": {
        "effect": "audit",
        "details": {
            "message": "Node pool should have bin-packing status tagged."
        }
    }
}
YAML
Copy
groups:
- name: "EvictionStormAlerts"
  rules:
  - alert: EvictionsPerMinuteHigh
    expr: kubernetes_pod_eviction_total[1m] / on(namespace) group_left(node) rate(kube_node_status_capacity_bytes[5m]) > 0.3
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Evictions per minute are high"
```

**Example input / output:**

```text
# Example input command to label a node pool
kubectl label nodes node-1 kubernetes.io/label=bin-packing

# Sample output from Azure Policy assignment creation
{
  "id": "/subscriptions/{subscription-id}/providers/Microsoft.Authorization/policyAssignments/canary-policy",
  "name": "canary-policy",
  "type": "Microsoft.Authorization/policyAssignments",
  "location": null,
  "properties": {
    "description": "",
    "policyDefinitionId": "/subscriptions/{subscription-id}/providers/Microsoft.Authorization/policyDefinitions/b1f3806924b74ebeab7d271a6a5a2ae9",
    "parameters": {}
  }
}
```

**Flow:**

1. Consolidation Initiation
2. Label Nodes for Bin-Packing
3. Taint Nodes if Necessary
4. Promote Policies to FullAssignment
5. Rollout in Stages with Canary
6. Monitor and Alert on EvictionStorms

