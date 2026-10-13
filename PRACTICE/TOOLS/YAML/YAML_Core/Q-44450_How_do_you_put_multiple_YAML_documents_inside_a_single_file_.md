---
type: "practice"
difficulty: "Intermediate"
---

# How do you put multiple YAML documents inside a single file and why is that useful for Kubernetes manifests?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Multi-document YAML for Kubernetes](<../../../../TOOLS/YAML/YAML_Core/Multi-document_YAML_for_Kubernetes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Kubernetes, it's essential to manage various resources like Deployments, Services, and ConfigMaps effectively. One common practice is to store multiple YAML documents in a single file, which can make your configurations more organized and easier to manage. This technique leverages the flexibility of YAML to encapsulate different resource definitions within a single document.

To understand why this is useful, let's break down the process from first principles and then discuss some practical examples.

#### Triage
What the Question Is Asking

The question is essentially asking how you can group multiple Kubernetes manifest files into one YAML file. It also seeks to understand the benefits of doing so. The key idea here involves leveraging YAML’s ability to define multiple documents within a single file, which is particularly useful in managing complex Kubernetes configurations.

#### First-Principles Solution

Kubernetes manifests are typically defined as YAML or JSON files, each representing a single resource like a Deployment, Service, or ConfigMap. However, you can combine these resources into one document by enclosing them with --- (triple dashes). This separator tells the Kubernetes API to parse everything up until it encounters another ---, treating each block as an independent manifest.

For example:

YAML
# First resource: Deployment.yaml
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
        image: myapp-image:latest

---

# Second resource: Service.yaml
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
    targetPort: 8080
  type: LoadBalancer

---

#### Closing Summary
+ Common Mistakes/Follow-Ups

Combining multiple Kubernetes manifests into one file using --- is a practical approach to manage and deploy complex configurations. This method is useful for organizing related resources, making it easier to version control and share configurations. However, it’s essential to ensure that each resource definition is correctly formatted and separated by the --- markers.

#### Common mistakes
include:

Forgetting to use --- as a separator between different resource definitions.
Mixing up the order or type of resources in the same file, leading to misconfiguration or deployment failures.
Overlooking validation checks during development, which can result in runtime errors.

In follow-ups, the interviewer might ask about best practices for organizing manifests, how to handle large-scale deployments, or alternative methods (like Helm charts) that offer more advanced templating and dependency management.

