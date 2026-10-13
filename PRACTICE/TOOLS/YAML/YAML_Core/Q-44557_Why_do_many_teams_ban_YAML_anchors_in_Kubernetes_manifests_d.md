---
type: "practice"
difficulty: "Deep"
---

# Why do many teams ban YAML anchors in Kubernetes manifests destined for GitOps, and what are the trade-offs of anchors versus Kustomize or Helm for DRY config?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Kustomize Patching](<../../../../TOOLS/YAML/YAML_Core/Kustomize_Patching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing why some DevOps teams avoid using YAML anchors in Kubernetes manifests for GitOps, it's essential to understand that this decision is rooted in a trade-off between simplicity and flexibility. Anchors in YAML allow for sharing common configurations across multiple documents, promoting code reuse and reducing redundancy (DRY principle). However, there are several reasons why some teams prefer alternatives like Kustomize or Helm.

To grasp the full picture, let's start by defining what anchors are and how they work, followed by a comparison with Kustomize and Helm. We'll also discuss the trade-offs of each approach in the context of GitOps.

Anchors allow you to define reusable parts of YAML manifests that can be included in multiple documents using & (alias) or * (reference). For example:

YAML
apiVersion: v1
kind: Namespace
metadata:
  name: &namespace k8s-testing

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  template:
    metadata:
      labels:
        app.kubernetes.io/instance: *namespace

In this example, the Namespace and Deployment share the same namespace via an anchor.

#### Triage

The core of the question revolves around understanding why teams might avoid YAML anchors in favor of Kustomize or Helm. There are several trade-offs to consider:

Simplicity vs. Flexibility: Anchors simplify manifest creation by reducing redundancy, but they can make manifests harder to understand and modify.
GitOps Considerations: GitOps requires consistent and predictable updates to the infrastructure. YAML anchors might introduce complexity during deployments or rollback operations.
Tooling Support: Kustomize and Helm provide extensive tooling that supports GitOps workflows more effectively than raw YAML.

#### First-Principles Solution

Simplicity of YAML Anchors:

#### Pros
Easier to write, less redundancy in configurations.

#### Cons
Increased complexity when modifying manifests; harder to reason about the impact of changes.

Kustomize:

#### Pros
Lightweight and easy to use for GitOps workflows; retains the benefits of YAML anchors but with better tooling support for GitOps operations like rolling back or updating configurations.

#### Cons
Less flexible compared to Helm, as it primarily focuses on overlays rather than full templating.

Helm:

#### Pros
Full-featured and widely adopted; supports more complex templating and dependency management.

#### Cons
Steeper learning curve; requires more advanced GitOps practices like templating and chart lifecycle management.

#### Trade-offs
DRY Principle vs. Simplicity: Anchors promote DRY but can lead to more complex manifests, especially when dealing with edge cases or large configurations.
GitOps Workflows: Kustomize provides a better fit for GitOps by offering tools that help manage changes and rollbacks in a predictable manner. Helm, while powerful, requires more GitOps-specific practices.
Learning Curve: Both Kustomize and Helm have lower learning curves compared to manually managing YAML anchors.

#### Closing Summary

In summary, teams might ban YAML anchors due to the increased complexity they introduce in large-scale configurations or during GitOps operations. Instead, tools like Kustomize offer a balanced approach that retains many benefits of DRY principles while providing better support for GitOps workflows. Helm is more powerful but requires a deeper understanding and commitment to advanced GitOps practices.

#### Common Mistakes/Follow-Ups
Misunderstanding YAML Anchors: Not recognizing the benefits of anchors in small-scale projects but overlooking their complexity in large-scale deployments.
Overlooking Tooling Support: Ignoring the extensive tooling support provided by Kustomize and Helm, which can simplify GitOps operations.
Closing

Understanding the trade-offs between YAML anchors and tools like Kustomize or Helm is crucial for making informed decisions about Kubernetes manifest management. The choice ultimately depends on the specific needs of the project and the level of complexity required.

**Code:**

```yaml
# Example with YAML anchors
apiVersion: v1
kind: Namespace
metadata:
  name: k8s-testing

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  template:
    metadata:
      labels:
        app.kubernetes.io/instance: &namespace k8s-testing
```

**Flow:**

1. DRY Principle
2. Anchors
3. Kustomize
4. Helm
5. Simplicity of Manifests
6. Complexity in Large-ScaleConfigurations
7. GitOps Workflow Support
8. Lightweight and Easy to Use
9. Full-Featured and WidelyAdopted
10. Steeper Learning Curve

