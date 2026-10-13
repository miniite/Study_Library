---
type: "practice"
difficulty: "Deep"
---

# Your AKS cluster's kubelet-managed nodes keep going NotReady under load spikes; walk me through how you'd diagnose whether it's node pressure, PLEG, or the Azure CNI IP exhaustion.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ AKS and Container Platform Standards](<../../../../TOOLS/Azure/Azure_Platform/Azure_Functions_and_Cosmos_DB_Tuning/AKS_and_Container_Platform_Standards.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with AKS cluster nodes that frequently go into a "NotReady" state during load spikes, diagnosing the issue requires a deep understanding of Kubernetes and Azure components. The key areas to investigate are node pressure, Pod Lifecycle Events (PLEG), and IP address exhaustion by the Azure CNI plugin. This involves a step-by-step analysis starting from the cluster's overall health to specific metrics and logs.

**Comparison:**

```
Metric	Description
CPU Utilization	Percentage of CPU used by the node's pods.
Memory Usage	Amount of memory consumed by running pods.
Disk I/O Operations	Number of read/write operations on the disk.
Network Throughput	Bandwidth usage and latency metrics.
```

**Code:**

```python
# Example Python code snippet for collecting Kubernetes API server metrics
import requests

def get_k8s_metrics(api_server_url):
    response = requests.get(f"{api_server_url}/metrics")
    if response.status_code == 200:
        return response.text
    else:
        raise Exception("Failed to fetch metrics")

# Example usage
api_server_url = "https://<your-aks-cluster>.eastus.aks.azure.com"
metrics = get_k8s_metrics(api_server_url)
print(metrics)
```

**Example input / output:**

```text
Input:
- AKS cluster experiencing frequent NotReady nodes during load spikes.

Output:
1. Check node CPU and memory usage via Kubernetes metrics API.
2. Review Pod events for any PLEG-related warnings or errors.
3. Monitor IP address utilization in Azure CNI logs and network interfaces.
4. Analyze node disk I/O and network latency using Azure monitoring tools.
```

**Data:**

```yaml
# Sample Kubernetes event log from Azure Monitor
events:
  - type: Warning
    reason: NodeUnreachable
    message: Node "<node-name>" is reporting unresponsive to the control plane.
    nodeName: <node-name>
    lastTimestamp: "2023-10-01T14:05:00Z"
```

**Flow:**

1. Check Node Metrics
2. Node CPU & Memory
3. Examine High Utilization
4. Review Pod Events for PLEGErrors
5. Monitor IP Address Exhaustion
6. Analyze Disk I/O and NetworkLatency

