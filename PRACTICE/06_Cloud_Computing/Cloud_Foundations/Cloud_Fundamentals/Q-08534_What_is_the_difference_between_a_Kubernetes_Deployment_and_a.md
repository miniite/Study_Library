---
type: "practice"
difficulty: "Deep"
---

# What is the difference between a Kubernetes Deployment and a StatefulSet, and when do you need the latter?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Stateless by Default](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Stateless_by_Default/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In cloud environments, managing stateless and stateful workloads efficiently requires understanding different Kubernetes resources like Deployments and StatefulSets. A Deployment is ideal for stateless applications where each pod in a deployment can be replaced with no data loss or dependency on previous states. In contrast, a StatefulSet manages stateful applications where every pod has a unique identity, ensuring persistent storage and network identifiers.

**Code:**

```yaml
# Example of a Deployment manifest
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: example-app
  template:
    metadata:
      labels:
        app: example-app
    spec:
      containers:
      - name: example-container
        image: example-image
```

