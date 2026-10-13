---
type: "practice"
difficulty: "Deep"
---

# With Azure CNI you ran out of pod IPs mid-scale; how do you decide between Azure CNI Overlay and kubenet, and what do you lose with each?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Azure Networking and Global Traffic](<../../../../TOOLS/Azure/Azure_Platform/Azure_DevOps_and_Artifact_Governance/Azure_Networking_and_Global_Traffic.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of running out of pod IPs in Azure Container Network Interface (CNI) mid-scale, we need to compare Azure CNI Overlay with kubenet. The decision hinges on understanding both solutions' strengths and trade-offs.

TL;DR

Azure CNI Overlay is a managed solution that uses an overlay network to allocate IP addresses across multiple subnets or VNETs. It provides more flexibility but comes with additional complexity and cost. On the other hand, kubenet leverages the underlying virtual network's CIDR blocks directly, making it simpler but potentially limiting in terms of scalability.

#### Triaging the Question

The core question is about choosing between Azure CNI Overlay and kubenet, given the constraint of running out of pod IPs. The interviewer is probing your understanding of networking principles, Azure CNI implementations, and the specific trade-offs involved in different network configurations within a Kubernetes cluster.

#### Assumptions
We are working with an Azure Kubernetes Service (AKS) environment.
kubenet uses the default Azure virtual network's CIDR block for pod IPs.
Azure CNI Overlay allows for IP allocation across multiple subnets or VNETs using VXLAN tunneling.
Step-by-Step Solution
1. Understanding kubenet
Default Behavior: kubenet is a lightweight, simple CNI plugin that uses the host's network stack.
IP Allocation: It operates within the default Azure virtual network's CIDR block (e.g., 10.240.0.0/16).
Limitation: If the virtual network runs out of IPs or you need to scale beyond the default CIDR, kubenet may not suffice.
2. Azure CNI Overlay
Overlay Network: Azure CNI Overlay uses VXLAN tunneling to overlay multiple subnets or VNETs.
IP Allocation: Pods can be allocated from different IP ranges across multiple subnets, allowing for more scalability.
Complexity and Cost: It introduces additional complexity in managing multiple network segments and potentially higher costs due to the managed nature of the service.
3. Making the Decision
Scalability Needs: If you anticipate scaling beyond the default CIDR block or need to spread pods across multiple subnets for better isolation, Azure CNI Overlay is a better choice.
Simplicity and Cost: If your cluster size remains within manageable limits and there's no strong need for IP address flexibility or better network segmentation, kubenet can be simpler and more cost-effective.

#### Closing Summary

Choosing between Azure CNI Overlay and kubenet depends on the specific requirements of your Kubernetes deployment. Azure CNI Overlay offers greater scalability but at the cost of complexity and potentially higher costs, while kubenet is simpler and cheaper but may run into limitations with larger deployments or complex network configurations.

#### Common Mistakes
and Follow-Ups
Common Mistake: Failing to consider the underlying virtual network's CIDR block limits when choosing kubenet.

#### Follow-Up
Questions: The interviewer might ask about specific scenarios where one option would be preferred over the other, such as handling network policies or managing different teams' deployments across multiple subnets.
Triaging Summary

In summary, Azure CNI Overlay and kubenet each have their own set of trade-offs. Understanding these will help in making an informed decision based on the specific needs of your Kubernetes cluster.

**Comparison:**

| Network Plugin | IP Allocation | Complexity | Cost |
|----------------|---------------|------------|------|
| kubenet        | Default CIDR  | Low        | Low  |
| Azure CNI      | VXLAN Overlay | High       | High |

**Code:**

```bash
# Example of deploying with kubenet (default)
az aks create --resource-group myResourceGroup --name myAKSCluster

# Example of configuring CNI Overlay
az aks update --resource-group myResourceGroup --name myAKSCluster \
  --kubernetes-version "1.20.7" \
  --network-plugin azure
```

**Example input / output:**

```

Input: AKS cluster with kubenet network plugin, running out of pod IPs in the default CIDR block. Output: Scaling issues resolved by switching to Azure CNI Overlay.

Request: How do you implement Azure CNI Overlay? Response:

Update the AKS cluster using Azure CLI commands:

CODE
Copy
az aks update --resource-group myResourceGroup --name myAKSCluster \
  --kubernetes-version "1.20.7" \
  --network-plugin azure

Monitor network usage and ensure pods are correctly allocated.
```

**Data:**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-container
        image: my-app-image
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

**Flow:**

1. B1
2. kubenet
3. B2
4. Azure CNI Overlay
5. Simple, Cost-Efficient
6. Complexity, Managed Network
7. No Scaling Issues
8. Higher Costs, Better Scalability

