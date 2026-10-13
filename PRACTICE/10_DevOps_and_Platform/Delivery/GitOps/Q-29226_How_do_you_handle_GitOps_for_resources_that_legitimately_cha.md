---
type: "practice"
difficulty: "Deep"
---

# How do you handle GitOps for resources that legitimately change at runtime, like HPA-scaled replica counts, without perpetual drift?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Multi-cluster Ordering](<../../../../10_DevOps_and_Platform/Delivery/GitOps/GitOps_Fundamentals/Multi-cluster_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

In a GitOps environment, managing resources that scale dynamically (like horizontal pod autoscalers, or HPAs) can introduce drift between what is defined in code and the current state of the cluster. This issue is critical because it can lead to misconfigurations, performance issues, and security vulnerabilities. The key solution involves implementing a robust feedback loop using tools like kubectl and kustomize, ensuring that desired states are always applied from Git while maintaining the ability to handle dynamic changes.

#### Triaging the Question

The question revolves around how to manage runtime changes (like HPA adjustments) in a GitOps setup without causing drift. The interviewer is probing your understanding of GitOps principles, Kubernetes resource management, and how to balance automated configuration with dynamic scaling mechanisms.

#### Assumptions
You are familiar with GitOps practices.
You understand the basics of Kubernetes and its resource objects.
You have experience with kubectl and kustomize.
The focus is on maintaining consistency between the desired state (as defined in code) and the actual running state.

#### First Principles Solution

Understand Desired vs Actual State:

In GitOps, the desired state of your infrastructure is codified in your Git repository.
The actual state is tracked by monitoring tools that report back to your CI/CD pipeline or an observability layer.

Implement a Feedback Loop:

Use kubectl and kustomize to apply the desired configuration from Git.
Monitor Kubernetes resources like HPAs, which scale based on observed metrics (e.g., CPU usage).
Integrate with tools like FluxCD or Argo CD that can automatically reconcile any drift between the current state and the desired state.

Automate Reconciliation:

Configure your GitOps tooling to apply the latest configuration from your Git repository.
Set up periodic reconciliations, such as using a Helm or Kustomize manifest that includes the HPA settings.
Ensure that any changes made by runtime scaling (e.g., HPAs) are periodically re-applied.

Handling Dynamic Changes:

Use kubectl apply to enforce the desired state and handle dynamic changes.
Implement a custom operator or controller to intercept and manage these changes, ensuring they align with your GitOps policies.
For example, you could create a Kubernetes custom resource definition (CRD) that allows for conditional scaling based on specific criteria.

Audit and Log:

Maintain detailed logs of all changes, both automated and manual.
Implement auditing mechanisms to ensure that any drift is detected early and resolved promptly.

#### Closing Summary

By implementing a robust feedback loop with tools like kubectl, kustomize, and GitOps orchestrators such as FluxCD or Argo CD, you can effectively manage runtime scaling while ensuring consistency between the desired state and the actual running state. This approach involves periodically reconciling dynamic changes with your Git repository to avoid drift.

#### Common Mistakes/Follow-Ups

Mistake: Ignoring runtime changes without proper reconciliation mechanisms.

#### Follow-Up
Discuss how ignoring these changes can lead to inconsistent states, performance issues, and security risks.

Mistake: Failing to use a GitOps tooling that supports custom operators or controllers for dynamic resource management.

#### Follow-Up
Explain the importance of using such tools to ensure compliance with your policies.

Mistake: Overly complex reconciliation logic leading to unnecessary overhead.

#### Follow-Up
Highlight the need for simplicity and efficiency in your GitOps workflows, while still ensuring robustness.

**Code:**

```yaml
# Example kustomization.yaml for HPA management
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- deployment.yaml

configMapGenerator:
- name: hpa-config
  env:
    - name: TARGET_CPU_UTILIZATION
      value: "70"

patchesStrategicMerge:
- patch_hpa.yaml

# Example patch_hpa.yaml to adjust HPA settings based on target utilization
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deployment
spec:
  template:
    spec:
      containers:
        - name: example-container
          resources:
            limits:
              cpu: "2"
            requests:
              cpu: "1"
```

**Flow:**

1. Git Repository
2. CI/CD Pipeline
3. Flux CD / Argo CD
4. Monitoring Tools
5. HPA Scaling
6. kubectl apply to Desired State
7. Periodic Reconciliation
8. Audit Logs & Alerts

