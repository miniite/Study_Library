---
type: "practice"
difficulty: "Expert"
---

# Reconcile GitOps with two competing control planes — Terraform/Crossplane for cloud infra and Argo CD for workloads — where each legitimately owns part of the state; how do you draw the seam?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-cluster Ordering](<../../../../10_DevOps_and_Platform/Delivery/GitOps/GitOps_Fundamentals/Multi-cluster_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a GitOps environment, reconciling two competing control planes—Terraform/Crossplane for cloud infrastructure and Argo CD for workloads—requires a strategic approach to ensure both systems operate coherently without conflicts. The key is drawing the seam between these control planes in such a way that they can harmoniously manage their respective state while maintaining consistency.

The question is probing your understanding of GitOps principles, specifically how different tools and workflows can be integrated into a cohesive system. Key assumptions include:

Both Terraform/Crossplane and Argo CD have legitimate ownership over specific parts of the infrastructure.
The goal is to minimize manual intervention and ensure automated reconciliation and validation.

To solve this, we need to break down the problem from first principles:

Define Ownership: Identify which components are managed by Terraform/Crossplane (e.g., network configurations, cluster setup) and which by Argo CD (e.g., application deployments).

State Management: Ensure both systems can read and write state information without overlap. This might involve using separate repositories or namespaces for different layers of the infrastructure.

Reconciliation Strategy: Establish a strategy to reconcile changes between these systems, potentially through custom GitOps workflows or by leveraging existing tools like Flux CD.

Automation and Validation: Implement automated validation and error handling mechanisms to prevent conflicts and ensure data integrity.

Here’s how we can draw the seam:

Separate Ownership:

Terraform/Crossplane: Responsible for setting up and managing cloud infrastructure such as VPCs, subnets, load balancers, and Kubernetes clusters.
Argo CD: Manages application deployments within these clusters, ensuring that workloads are consistently deployed across environments.

State Management:

Use separate Git repositories or namespaces for different layers of the infrastructure. For example, one repository could manage cloud resources while another handles applications.
Ensure both systems can access and update their respective state files using appropriate permissions and workflows.

Reconciliation Strategy:

Implement a custom GitOps workflow that triggers Argo CD to reconcile changes made by Terraform/Crossplane. This might involve writing scripts or leveraging existing tools like Flux CD to act as an intermediary.
Use semantic versioning or timestamps in Git to ensure that updates are applied in the correct order and without conflicts.

Automation and Validation:

Set up automated validation checks using tools like kubectl or custom scripts to verify that changes made by one system do not conflict with those made by the other.
Implement CI/CD pipelines that validate state consistency before applying any changes, ensuring that both control planes are aligned.

By carefully defining ownership, managing state separately, implementing a robust reconciliation strategy, and automating validation, we can successfully integrate Terraform/Crossplane and Argo CD into a cohesive GitOps environment. This approach ensures that both systems operate independently but harmoniously, reducing the risk of conflicts and maintaining consistent infrastructure and workloads.

**Comparison:**

```
System	Ownership	State Management	Reconciliation Strategy
Terraform/Crossplane	Cloud Infrastructure	Separate repository	Custom GitOps workflow (Flux CD)
Argo CD	Workloads and Applications	Separate namespace	Triggered by state changes in Terraform
```

**Code:**

```
Example Terraform configuration for network setup

resource "aws_vpc" "main" { cidr_block = "10.0.0.0/16" }

resource "aws_subnet" "web" { vpc_id = aws_vpc.main.id cidr_block = "10.0.1.0/24" }

CODE
Copy

```yaml
# Example Argo CD configuration for application deployment
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: example-app
spec:
  source:
    repoURL: https://github.com/example/repo.git
  destination:
    server: https://kubernetes.default.svc
    namespace: default
```

**Example input / output:**

```text
- **Input**: A change request to update network configurations using Terraform.
- **Output**: Argo CD is notified of the state change, performs a reconciliation, and updates application deployments accordingly.
```

**Data:**

```json
{
  "cloud_infrastructure_repo": {
    "branch": "main",
    "path": "/infrastructure/cloud"
  },
  "workloads_repo": {
    "branch": "main",
    "path": "/applications/workloads"
  }
}
```

**Flow:**

1. Cloud Infrastructure
2. Terraform/Crossplane
3. Applications
4. Argo CD
5. State Change Notification
6. Reconciliation Trigger
7. Application DeploymentUpdate

