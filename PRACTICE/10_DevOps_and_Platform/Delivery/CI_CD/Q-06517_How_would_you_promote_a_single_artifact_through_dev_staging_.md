---
type: "practice"
difficulty: "Deep"
---

# How would you promote a single artifact through dev, staging, and prod without rebuilding?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Build Reproducibility and Artifact Promotion](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Build_and_Release_Metadata/Build_Reproducibility_and_Artifact_Promotion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To promote an artifact (e.g., a Docker image or a software build) seamlessly from development (dev), to staging, and finally to production (prod) without rebuilding, we need to leverage a robust CI/CD pipeline that is capable of managing the lifecycle of our artifacts across different environments. This process involves several steps: environment isolation, artifact management, deployment strategy, and continuous monitoring.

The core idea here is to ensure that the same exact version of your code and dependencies are deployed in each environment without any manual intervention or rebuilding, which can introduce human errors and inconsistencies. Let’s dive into the detailed solution.

**Comparison:**

```
Environment	Deployment Strategy
Dev	Manual or CI-triggered deployment
Staging	Automated smoke tests, security scans
Prod	Automated release with rollback plan

Key Points	Description
-------------	---------------------
Consistency	Ensuring the same artifact is used in all environments.
Security	Implementing proper access controls and monitoring for each environment.
```

**Code:**

```
Source Control
Build
Artifact Management
Test
Integration
Staging
Production

This flowchart outlines the typical CI/CD pipeline, where artifacts are built and deployed from source control through various stages.
```

**Example input / output:**

```

For a Kubernetes deployment, the image field in the YAML manifest specifies the exact version of the Docker image to be used for deployment.

YAML
Copy
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app-container
        image: my-app:v1.0.0 # This version tag is critical for consistency across environments.
        ports:
        - containerPort: 8080
```

**Data:**

```yaml
# Example of a YAML deployment manifest for Kubernetes

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app-container
        image: my-app:v1.0.0 # The version tag is critical here
        ports:
        - containerPort: 8080
```

**Flow:**

1. Source Control
2. Build
3. Artifact Management
4. Test
5. Integration
6. Staging
7. Production
8. This flowchart represents the CI/CD pipeline steps, ensuring that the same artifact is used across environments.

