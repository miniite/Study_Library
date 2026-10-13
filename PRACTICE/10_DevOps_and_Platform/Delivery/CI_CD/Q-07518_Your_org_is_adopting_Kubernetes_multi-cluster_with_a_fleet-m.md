---
type: "practice"
difficulty: "Expert"
---

# Your org is adopting Kubernetes multi-cluster with a fleet-management layer; how do you decide the split of responsibility between the fleet controller, the per-cluster GitOps agent, and the CD pipeline for who declares intended version, who reconciles, and who owns rollback?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Push vs Pull Deployments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Deployment_Observability/Push_vs_Pull_Deployments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a Kubernetes multi-cluster environment with a fleet-management layer, deciding the split of responsibility between the fleet controller, per-cluster GitOps agents, and CD pipelines is crucial for maintaining consistency, reliability, and ease of management. The goal is to ensure that each component has clear roles in declaring intended states, reconciling those states, and handling rollbacks.

The key idea here is to leverage the strengths of both central control and local operations to create a robust CI/CD pipeline tailored to multi-cluster Kubernetes environments. The fleet controller handles high-level orchestration, while per-cluster agents manage localized reconciliation. CD pipelines focus on continuous delivery, ensuring that changes are tested before being applied.

The question asks how to allocate responsibilities between different components in a multi-cluster Kubernetes setup with fleet management. Specifically, it seeks clarity on who should declare the desired state (intended version), who should perform reconciliation, and who should manage rollbacks. The key assumptions are that GitOps principles will be applied, and there is an existing CI/CD pipeline for deployment.

Fleet Controller: This component is responsible for high-level orchestration across multiple clusters. Its primary tasks include:

Defining the desired state of infrastructure resources.
Managing global policies and configurations.
Ensuring consistency in cluster deployments.

Per-Cluster GitOps Agents: These agents operate at a lower level, focusing on each individual cluster. Their main responsibilities are:

Reading the desired state from the fleet controller.
Applying this state to the local cluster infrastructure.
Monitoring and reporting discrepancies (reconciliation).

CD Pipelines: CD pipelines handle continuous integration and deployment tasks within clusters. They ensure that new versions or updates go through testing before being deployed. The responsibilities of CD pipelines include:

Building, testing, and deploying code changes.
Managing application lifecycle stages.
Ensuring compliance with local cluster policies.

Fleet Controller Declaration:

The fleet controller should declare the intended version or state for global resources that need uniformity across clusters (e.g., shared services, infrastructure components).
It acts as a central repository of truth and enforcer of global consistency.

Per-Cluster Reconciliation:

Per-cluster agents are responsible for applying the declared state to each individual cluster.
They perform reconciliation to ensure that local clusters match the desired state.
This ensures that each cluster remains consistent with the overall organizational policies.

CD Pipeline Rollback:

CD pipelines handle rolling back changes if necessary, but they do not initiate the rollback themselves.
They are responsible for testing and deploying code, ensuring that any issues can be rolled back within a cluster's scope without affecting others.
The fleet controller might orchestrate rollbacks on a larger scale, coordinating with per-cluster agents to ensure consistency.

By splitting responsibilities in this way, the system achieves a balance between centralized control and local autonomy. The fleet controller ensures global consistency, while per-cluster agents handle localized reconciliation. CD pipelines focus on continuous delivery and testing within clusters, enabling efficient and reliable deployment processes.

Over-centralizing decisions can lead to delays and reduced responsiveness.
Underestimating the need for local reconciliation can result in inconsistencies between clusters.
Ignoring the importance of rollback mechanisms can lead to cascading failures if a critical update fails.

The solution is correct and comprehensive, covering all aspects of responsibility allocation in a multi-cluster Kubernetes environment.

