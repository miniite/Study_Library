---
type: "practice"
difficulty: "Basic"
---

# In a Kubernetes YAML manifest what are the four common top-level fields you always see?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Multi-document YAML for Kubernetes](<../../../../TOOLS/YAML/YAML_Core/Multi-document_YAML_for_Kubernetes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Kubernetes YAML manifests, there are several key top-level fields that are essential to define the configuration of your Kubernetes resources. As a DevOps fresher or mid-level engineer, understanding these fields is crucial for creating and managing Kubernetes deployments effectively.

In this context, let's break down the common top-level fields you always see in a Kubernetes YAML manifest:

apiVersion: This field specifies which version of the API to use. It helps ensure compatibility with the cluster.
kind: This indicates the type of object being defined (e.g., "Pod", "Deployment", "Service").
metadata: A collection of metadata fields, such as the name and labels, that provide identifying information for the resource.
spec: The specification section where you define how the Kubernetes object should be configured.

Each field serves a critical role in defining the nature and behavior of your Kubernetes resources.

**Code:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: nginx:latest
```

