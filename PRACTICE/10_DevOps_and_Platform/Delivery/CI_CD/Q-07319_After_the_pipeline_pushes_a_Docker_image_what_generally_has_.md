---
type: "practice"
difficulty: "Intermediate"
---

# After the pipeline pushes a Docker image, what generally has to happen next for users to see the change?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Container Images and Registries](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Container_Images_and_Registries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a Docker image is pushed to a container registry as part of a CI/CD pipeline, several steps need to be taken before users can see and use the new changes. This involves understanding the entire process from the push event to the deployment in production environments.

#### Triage

The question is essentially asking about the post-pipeline steps required for a Docker image to become available to end-users. The interviewer might be probing your knowledge of container orchestration, deployment strategies, and ensuring that new changes are correctly integrated into the infrastructure. Key assumptions include:

You understand CI/CD pipelines.
You know how Docker images are used in deployments.

#### First Principles Solution
Tagging and Pushing: Ensure the Docker image is properly tagged with a version or tag before pushing it to the registry.
Image Availability: Verify that the new Docker image has been successfully pushed to the container registry.
Deployment Strategy:
Manual Deployment: If manual deployment is used, trigger the deployment process from your CI/CD pipeline.
Automated Deployment: Set up an automated deployment strategy using a tool like Kubernetes or Docker Swarm.
Service Restart: Ensure that old services are gracefully shut down and new containers with updated images are started.
Health Checks: Implement health checks to ensure the new service is running correctly before marking it as active.
Rollback Plan: Establish a rollback mechanism in case anything goes wrong.
Full Solution
Step-by-Step Process

Tagging and Pushing:

Tags help identify different versions of your Docker image. Use Git commit hashes, build numbers, or semantic versioning to tag the image.
BASH
docker tag <local-image-name>:<tag> <registry-url>/<image-name>:<tag>
Push the tagged image to the container registry.
BASH
docker push <registry-url>/<image-name>:<tag>

Image Availability:

Use a tool like curl or a browser to verify that the Docker image is available in the registry.
BASH
curl http://<registry-url>/v2/<image-name>/tags/list

Deployment Strategy:

Manual Deployment: Trigger the deployment manually from your CI/CD pipeline using commands like kubectl apply.
YAML
# Example Kubernetes deployment manifest
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
        image: <registry-url>/<image-name>:<tag>
        ports:
        - containerPort: 80
Automated Deployment: Use a CI/CD pipeline to automatically trigger deployment when the Docker image is pushed. This can be done using Kubernetes Helm, GitOps tools like Argo CD, or simple shell scripts.

Service Restart:

Ensure that old containers are stopped before starting new ones.
BASH
kubectl rollout restart deployment/my-app-deployment

Health Checks:

Implement liveness and readiness probes to ensure the new service is healthy before marking it as active.
YAML
spec:
  template:
    metadata:
      labels: {}
    spec:
      containers:
      - name: my-app-container
        image: <registry-url>/<image-name>:<tag>
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /healthz
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10

Rollback Plan:

Have a clear rollback strategy in place. This can be done by rolling back to a previous version of the Docker image or using a blue-green deployment strategy.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, after pushing a Docker image to the registry, you need to ensure it is tagged correctly, verify its availability, and then deploy it through your chosen method. Automated deployments are generally preferred for efficiency and reliability. Common mistakes include forgetting health checks, not having a rollback plan, or failing to update service configurations.

#### Follow-Ups
How do you handle versioning in Docker images?
What security measures should be considered during the deployment process?

**Code:**

```bash
# Example tagging and pushing command
docker tag local-image-name:latest registry-url/image-name:1.0.2
docker push registry-url/image-name:1.0.2

# Example Kubernetes deployment manifest
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
        image: registry-url/image-name:1.0.2
        ports:
        - containerPort: 80
```

**Example input / output:**

```text
# Input to verify Docker image availability
curl http://registry-url/v2/image-name/tags/list

# Output example
{
  "name": "image-name",
  "tags": ["1.0.2", "latest"]
}
```

**Data:**

```yaml
# Example Kubernetes deployment manifest
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
        image: registry-url/image-name:1.0.2
        ports:
        - containerPort: 80
```

**Flow:**

1. Tag and Push Docker Image
2. Verify Image Availability
3. Automate Deployment UsingCI/CD
4. Deploy New Containers withUpdated Images
5. Implement Health Checks
6. Rollback Plan in Place

