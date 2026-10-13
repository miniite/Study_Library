---
type: "practice"
difficulty: "Basic"
---

# What role does the 'desired state' play in GitOps?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Repo and Environment Layout](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Repo_and_Environment_Layout/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In GitOps, the "desired state" plays a pivotal role as it represents the desired configuration of your infrastructure and applications defined in code. This concept ensures that all changes are managed through version control systems like Git, making it easier to track, review, and revert changes. Let's break down how this works from first principles.

**Comparison:**

```
Component	Description
Desired State	Configuration stored in Git, representing the intended state of infrastructure and applications.
Actual State	Current live configuration of your cluster.
```

**Code:**

```yaml
# Example Desired State Configuration for Kubernetes Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3 # Set according to desired state
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

**Flow:**

1. Desired State
2. Git Repository
3. CI/CD Pipeline
4. Cluster API
5. Actual State

