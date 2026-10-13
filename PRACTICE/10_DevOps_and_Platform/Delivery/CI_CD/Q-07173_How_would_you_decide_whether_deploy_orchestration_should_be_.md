---
type: "practice"
difficulty: "Expert"
---

# How would you decide whether deploy orchestration should be pull-based (GitOps reconciler) or push-based for a fleet that spans managed Kubernetes, VMs, and serverless simultaneously, and can one model actually serve all three?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Push vs Pull Deployments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Deployment_Observability/Push_vs_Pull_Deployments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide between pull-based (GitOps) and push-based deploy orchestration for a mixed environment of managed Kubernetes, VMs, and serverless workloads, we need to consider several factors including consistency, scalability, security, and operational complexity. The GitOps approach inherently ensures that infrastructure and application configurations are version-controlled and can be reconciled with desired states, making it well-suited for environments where multiple teams or services are involved. However, a push-based model might simplify deployment in certain scenarios.

#### Triaging the Question

The question is fundamentally about choosing between two CI/CD strategies based on the characteristics of the target environment. GitOps relies on declarative configuration and continuous reconciliation to match actual states with desired ones, while push-based models typically involve manual or automated scripts that trigger deployments through APIs or webhooks. The key factors include:

#### Cons
istency across environments: Ensuring that all components are in sync.
Scalability of the deployment process.
Security and compliance requirements.
Operational complexity and maintenance overhead.

#### First-Principles Solution

To determine which model is best, we need to evaluate each strategy against the specific needs of our fleet. Here’s a step-by-step breakdown:

#### Cons
istency: GitOps ensures that all environments (Kubernetes clusters, VMs, serverless functions) are driven by the same source of truth in code repositories. This makes it easier to manage changes across multiple systems and maintain compliance.
Scalability: GitOps can be highly scalable with tools like Flux or ArgoCD managing many clusters from a single repository. Push-based models might require custom scripts or tools for each cluster, which can become cumbersome as the number of environments grows.
Security: With GitOps, there's a well-defined audit trail and version control that helps in securing changes through pull requests and approvals. Push-based models may lack this layer of security unless additional layers are implemented.
Operational Complexity: GitOps requires setting up continuous integration pipelines and potentially managing multiple reconciliation loops. However, once set up, it can significantly reduce the operational overhead during deployment.
Detailed Solution

To decide between these models, we should evaluate the following:

Managed Kubernetes Clusters:

GitOps: Leverage tools like Flux or ArgoCD to sync cluster states with Git repositories.
Push-based: Use APIs for manual or automated deployments. This might be simpler but less consistent.

VMs:

GitOps: While possible, setting up and maintaining GitOps in VM environments can be more challenging due to the lack of built-in Kubernetes features.
Push-based: Custom scripts or tools that trigger VM changes through APIs or configuration management tools like Ansible or Terraform.

Serverless Functions:

GitOps: Use CI/CD pipelines integrated with serverless platforms (e.g., AWS Lambda, Azure Functions) to manage deployments and configurations.
Push-based: Deploy functions via platform-specific APIs or through custom scripts. This is more straightforward but lacks the benefits of centralized management.

#### Closing Summary

In summary, GitOps offers a unified approach that ensures consistency across all environments while reducing operational complexity. However, its implementation requires careful planning and might not be feasible in all cases due to the differences between managed Kubernetes, VMs, and serverless functions. A hybrid approach could also be considered, where push-based models are used for simpler tasks, and GitOps handles more critical or complex deployments.

#### Common Mistakes
Overlooking the security benefits of GitOps.
Assuming that a push-based model will naturally scale without additional effort.
Ignoring the operational complexity introduced by setting up and maintaining multiple GitOps pipelines.

**Comparison:**

```
Environment Type	GitOps	Push-based
Managed Kubernetes	Flux, ArgoCD	Custom scripts, API triggers
VMs	Custom implementation	Ansible/Terraform, API triggers
Serverless Functions	CI/CD pipelines	Platform-specific APIs
```

**Code:**

```
MERMAID
Copy
graph TD
    A[Managed Kubernetes] --> B1(GitOps)
    A --> B2(Push-based)

    C[VMs] --> D1(GitOps)
    C --> D2(Push-based)

    E[Serverless Functions] --> F1(GitOps)
    E --> F2(Push-based)
```

**Example input / output:**

```

Input:

Environment type: Managed Kubernetes
Deployment type: GitOps or Push-based

Output:

If choosing GitOps:
Deploy using Flux/ArgoCD to sync cluster state with Git repository.
If choosing Push-based:
Implement custom scripts or use API triggers for deployment.

Input:

Environment type: VMs
Deployment type: GitOps or Push-based

Output:

If choosing GitOps:
Consider setting up a custom GitOps implementation that works with VM management tools.
If choosing Push-based:
Use Ansible, Terraform, or similar tools to manage VM configurations.

Input:

Environment type: Serverless Functions
Deployment type: GitOps or Push-based

Output:

If choosing GitOps:
Integrate CI/CD pipelines with serverless platforms for seamless deployments.
If choosing Push-based:
Use platform-specific APIs to deploy and manage functions.
```

**Data:**

```json
{
  "environments": {
    "kubernetes": ["GitOps", "Push-based"],
    "vm": ["GitOps", "Push-based"],
    "serverless": ["GitOps", "Push-based"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Environment
8. Is Managed Kubernetes?
9. GitOps
10. Push-based
11. Deploy using Flux/ArgoCD
12. Implement custom scripts/APItriggers
13. VMs
14. Is Managed Kubernetes?
15. GitOps
16. Push-based
17. Custom implementation forVMs
18. Use Ansible/Terraform, APItriggers
19. Serverless Functions
20. Is Managed Kubernetes?
21. GitOps
22. Push-based
23. Integrate CI/CD pipelines
24. Use platform-specific APIs fordeployment

