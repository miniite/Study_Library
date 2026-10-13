---
type: "practice"
difficulty: "Intermediate"
---

# A capacity spike needs more AKS nodes than your quota allows and it's a prod incident; how do you handle quota increases and burst capacity in the moment?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Azure Functions and Cosmos DB Tuning](<../../../../TOOLS/Azure/Azure_Platform/Azure_Functions_and_Cosmos_DB_Tuning/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a capacity spike during production, ensuring that you can quickly and effectively handle additional AKS nodes while respecting Azure quotas is crucial. As an experienced DevOps engineer, we need to balance immediate needs against long-term planning.

Firstly, the question asks how to handle quota increases and burst capacity in the moment. This involves understanding the current Azure quota limits, identifying the shortfall, and then implementing a strategy that ensures availability without causing potential issues down the line.

#### Triage
Understanding the Question

The core of this question is about managing dynamic resource demands during an incident while staying within organizational constraints. The interviewer might be probing your ability to navigate resource limitations, understand quota management in Azure, and implement effective capacity planning strategies.

#### First-Principles Solution
Assess Current Quota: Check the current AKS node quota for the subscription.
Identify Shortfall: Determine how many additional nodes are required beyond the current quota.
Request Quota Increase: Submit a request to your Azure administrator or the appropriate team for an increased quota.
Implement Autoscaling: Use Kubernetes autoscaling policies to dynamically adjust node counts based on demand.
Monitor and Optimize: Continuously monitor resource usage and optimize the configuration to balance costs with performance.
Detailed Steps

Check Current Quota

Log in to the Azure portal or use az account commands via Azure CLI to check the current quota limits for AKS nodes.

Identify Shortfall

Estimate the required number of additional nodes based on current load and expected demand.

Request Quota Increase

Prepare a detailed request explaining the business need, the estimated duration, and the impact if the request is denied.
Work with your Azure administrator or relevant team to escalate the request for an increased quota.

Implement Autoscaling

Configure horizontal pod autoscaling (HPA) in Kubernetes to automatically scale out nodes based on CPU or memory usage metrics.
Set appropriate scaling thresholds and minimum/maximum node limits to ensure balanced resource utilization.

Monitor and Optimize

Implement logging and monitoring using tools like Prometheus and Grafana to track resource consumption.
Regularly review the HPA configurations and make adjustments as needed to optimize performance while reducing costs.

#### Closing Summary

In summary, handling a capacity spike in AKS requires a combination of immediate actions (such as requesting increased quota) and long-term strategies (like implementing autoscaling). It's essential to balance resource demands with organizational constraints to ensure high availability and efficient use of resources.

#### Common Mistakes/Follow-Ups
Over-requesting Quota: Requesting more than necessary can lead to unnecessary costs. Always justify the request based on specific needs.
Ignoring Long-Term Planning: Focusing solely on short-term fixes without planning for future capacity needs can result in ongoing issues.
Not Utilizing Autoscaling: Relying only on manual scaling can cause performance issues and resource wastage.

**Code:**

```
Check current AKS node quota using Azure CLI

az account list-quotas --namespace Microsoft.ContainerService --query "[].{Name:name,Value:value}"

CODE
Copy

```yaml
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      targetAverageUtilization: 50
```

**Flow:**

1. Yes
2. No
3. Check Current Quota
4. Is Quota Sufficient?
5. Implement Autoscaling
6. Request Increased Quota
7. Monitor and Optimize
8. Continuously Monitor Usage

