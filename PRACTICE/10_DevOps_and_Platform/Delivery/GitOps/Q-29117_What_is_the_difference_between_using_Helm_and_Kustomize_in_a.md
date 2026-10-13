---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between using Helm and Kustomize in a GitOps repo?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ GitOps Fundamentals](<../../../../10_DevOps_and_Platform/Delivery/GitOps/GitOps_Fundamentals/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a GitOps setup, both Helm and Kustomize are used to manage Kubernetes resources declaratively, but they serve different purposes and have distinct features that make them suitable for different use cases.

#### Triaging the Question

The core of this question is understanding how Helm and Kustomize differ in managing and deploying Kubernetes applications. The interviewer might be probing your knowledge of GitOps practices, your ability to compare tools, and your understanding of their respective strengths and weaknesses. Assumptions include familiarity with Kubernetes resources, GitOps workflow, and the concept of declarative configuration.

#### First-Principles Solution
Helm

Helm is a package manager for Kubernetes that simplifies deploying complex applications by packaging them into chart files. A Helm chart defines the desired state of your application as a set of Kubernetes manifests (YAML files) along with additional metadata like dependencies and values. Here’s how Helm works in GitOps:

Chart Structure: Helmi charts have a specific directory structure, including templates, values.yaml, and other files.
Values File: Allows customization of application configurations via variables.
Package Management: Uses Tiller (now deprecated) or the Helm server for package management.
Kustomize

Kustomize is a tool for customizing Kubernetes manifests. It allows you to layer changes on top of base configuration files without modifying them directly, making it easier to manage small differences across environments like development and production.

Base Configuration: You start with a set of base YAMLs that define the default state.
Layered Customizations: Use overlays (e.g., kustomization.yaml) to add or modify resources without altering the original files.

#### Cons
istency: Ensures consistency across environments by maintaining a single source of truth.

#### Example Scenario

Imagine you have a GitOps workflow where your team needs to deploy and manage multiple applications on different Kubernetes clusters, each with varying configurations like environment variables, service accounts, and resource limits.

Using Helm

You might use Helm charts for complex applications requiring dependencies or extensive configuration:

MARKDOWN
# Chart Directory Structure
charts/
  my-app/
    Chart.yaml
    values.yaml
    templates/
      deployment.yaml
      service.yaml

A values.yaml file might look like this:

YAML
replicaCount: 3

image:
  repository: example/myapp
  tag: v1.0.0
  pullPolicy: IfNotPresent

env:
  - name: ENVIRONMENT
    value: development
Using Kustomize

For simpler, more lightweight configurations:

YAML
# base/
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app

---
apiVersion: v1
kind: Service
metadata:
  name: my-app-service

In the kustomization.yaml, you can add overlays:

YAML
resources:
- base/

patchesStrategicMerge:
- myapp-patch.yaml
- service-patch.yaml

#### Closing Summary
and Common Mistakes

Using Helm is ideal when you need to manage complex applications with many dependencies, while Kustomize excels in managing simpler configurations across multiple environments. Both tools are essential in a GitOps workflow but serve different needs.

#### Common Mistakes

Overusing Helm for simple deployments where Kustomize would be sufficient.
Not understanding the difference between patches and overlays in Kustomize, leading to incorrect or inconsistent configurations.
Final Thoughts

In practice, it’s common to use both tools together. For example, you might use a Helm chart to package your application's dependencies and then apply Kustomize overlays to customize it for different environments.

