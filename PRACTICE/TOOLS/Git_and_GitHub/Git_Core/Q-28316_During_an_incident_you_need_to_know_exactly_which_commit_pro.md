---
type: "practice"
difficulty: "Deep"
---

# During an incident you need to know exactly which commit produced the running binary of a stateless service across 200 pods. How do you bake, expose, and later query that provenance, and why is a mutable tag insufficient?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reproducible Builds from Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reproducible_Builds_from_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To track down the exact commit producing the running binary of a stateless service across multiple pods during an incident, we need to bake in a reliable way to capture this provenance, expose it appropriately, and later query it effectively. The key challenge here is that relying solely on mutable tags (like latest, v1.23.4) can lead to issues, as they are prone to changes and lack the necessary granular detail.

In essence, the solution involves:

Baking in a reliable way to capture the commit hash.
Exposing this information through Git metadata or environment variables.
Querying this provenance later using tools like kubectl or custom scripts.

Let's delve into each step from first principles.

**Comparison:**

```
Key	Description
COMMIT_HASH	The commit hash of the running binary
annotations	Git metadata annotations used to store the commit hash
image	Docker image name with placeholder for COMMIT_HASH
```

**Code:**

```
SH
Copy
# Example of baking in the commit hash via a Git tag and setting an env var in deployment YAML

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-stateless-service
spec:
  replicas: 200
  template:
    metadata:
      labels:
        app: my-stateless-service
    spec:
      containers:
      - name: my-container
        image: <IMAGE_NAME>:<COMMIT_HASH>
        env:
        - name: COMMIT_HASH
          valueFrom:
            fieldRef:
              fieldPath: metadata.annotations.commitHash
```

**Example input / output:**

```text
# Example of using kubectl to query the commit hash from a pod

$ kubectl get pods -l app=my-stateless-service -o json | jq '.items[].metadata.annotations.commitHash'
abc123def456
```

**Data:**

```yaml
# Sample deployment YAML with baked-in commit hash via annotations

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-stateless-service
spec:
  replicas: 200
  template:
    metadata:
      labels:
        app: my-stateless-service
      annotations:
        commitHash: "abc123def456"
    spec:
      containers:
      - name: my-container
        image: myregistry/my-service:<COMMIT_HASH>
        env:
        - name: COMMIT_HASH
          valueFrom:
            fieldRef:
              fieldPath: metadata.annotations.commitHash
```

**Flow:**

1. Repository
2. Tag commit
3. Push code
4. Deployment YAML
5. Pods
6. Query Commit Hash
7. Annotations
8. Env Var COMMIT_HASH
9. Kubernetes Metadata

