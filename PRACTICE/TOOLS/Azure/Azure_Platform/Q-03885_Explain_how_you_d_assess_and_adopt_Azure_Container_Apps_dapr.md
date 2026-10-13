---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd assess and adopt Azure Container Apps' dapr and KEDA capabilities as an org standard versus keeping everything on AKS; what workloads move and what stays?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ AKS and Container Platform Standards](<../../../../TOOLS/Azure/Azure_Platform/Azure_Functions_and_Cosmos_DB_Tuning/AKS_and_Container_Platform_Standards.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To assess and adopt Azure Container Apps with Dapr and Kubernetes Ecosystem Autoscaling (KEDA) capabilities, we need to evaluate various aspects of our current infrastructure, workload characteristics, and future requirements. The goal is to determine which workloads can be migrated to Azure Container Apps while keeping others on AKS (Azure Kubernetes Service), ensuring a smooth transition that aligns with organizational standards.

#### Triaging the Question

The question is essentially probing how we balance leveraging Azure's managed services like Container Apps and Dapr for their inherent benefits, such as reduced operational overhead and enhanced functionality, against the flexibility and control provided by AKS. Key factors include workload type, performance requirements, cost considerations, and existing infrastructure dependencies.

#### Assumptions
Current Workloads: We have a mix of containerized applications running on AKS.
Azure Container Apps: Managed service that simplifies deployment and management of stateless containers.
Dapr: A toolkit for building microservices with resilience, security, and observability features.
KEDA (Kubernetes Event-driven Autoscaling): A solution to automatically scale Kubernetes pods based on application events or metrics.
Step-by-Step Solution
1. Identify Workload Characteristics
Stateless vs Stateful: Stateless applications are better suited for Container Apps due to their managed nature.
High I/O Operations: Workloads with frequent database interactions might benefit from AKS where you have more control over storage and network configurations.
2. Evaluate Performance Requirements
Latency Sensitivity: Applications requiring low latency may stay on AKS as it provides lower-latency guarantees compared to managed services.
Resource Intensive: CPU/GPU-intensive applications can potentially move to Container Apps but require careful monitoring for performance degradation.
3. Consider Cost and Resource Management
Cost Optimization: Container Apps offer pay-as-you-go pricing, which can be more cost-effective for low-demand workloads.
Resource Allocation: KEDA allows dynamic scaling based on real-time metrics, which can help optimize resource usage in AKS.
4. Assess Existing Dependencies
Third-Party Integrations: Workloads that rely heavily on external services or custom integrations might be better left on AKS due to potential compatibility issues.
Custom Configurations: Applications with specific configurations and settings may require the flexibility of a self-managed cluster like AKS.
5. Migrate Step-by-Step
Pilot Migration: Select one or two small applications for a trial run in Container Apps to evaluate performance, stability, and ease of management.
Performance Monitoring: Use tools like Azure Monitor and KEDA to continuously monitor the application's health and resource utilization.
Refinement and Scaling: Based on pilot results, refine policies and configurations to ensure smooth operation at scale.

#### Closing Summary

In summary, we need a hybrid approach where stateless applications that do not require high I/O or strict latency guarantees can be migrated to Azure Container Apps with Dapr for reduced operational overhead and enhanced functionality. Meanwhile, resource-intensive and performance-critical workloads should remain on AKS due to its flexibility and control. Continuous monitoring and refinement will ensure optimal deployment strategies.

#### Common Mistakes
and Follow-Ups
Over-Migration: Moving too many applications without thorough testing can lead to unexpected downtime or performance issues.
Underutilization of Features: Not fully leveraging Dapr and KEDA features can result in suboptimal application behavior and resource usage.
Dependency Issues: Ignoring existing dependencies can cause compatibility problems, leading to migration delays or failures.

