---
type: "practice"
difficulty: "Deep"
---

# Your deploy tool reports success the instant the orchestrator accepts the manifest, before pods are actually serving; how would you make 'deploy done' mean 'traffic verified healthy' without adding a long fixed sleep?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Deployment Verification](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Validating_Pipeline_Changes/Deployment_Verification.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that our CI/CD pipeline accurately signals "deploy done" only after traffic has been verified as healthy, we need to change the current approach where deployment success is reported based on orchestrator acceptance alone. Instead, we must introduce a mechanism that waits for and confirms the application's health before marking the deployment complete. This requires modifying both our deployment process and potentially introducing additional monitoring steps.

#### Triage
Understanding the Problem

The question probes how to synchronize the CI/CD pipeline with the actual state of deployed applications in production. The key challenge is ensuring that the "deploy done" signal aligns with the application being ready for traffic, rather than just the manifest being accepted by the orchestrator. This involves monitoring the health of the deployed pods and verifying they are serving requests correctly.

#### First-Principles Solution

To solve this problem, we need to integrate health checks into our deployment process. Here’s a step-by-step approach:

Modify Deployment Process: Ensure that deployments include a hook or script to run post-deployment health checks.
Health Checks: Implement service-level and application-level health checks using tools like liveness probes, readiness probes, or custom scripts.
Wait for Health: Use the results of these health checks to determine when the deployment is complete.

#### Implementation
Steps

Update Deployment Manifest:

Add a livenessProbe and readinessProbe in Kubernetes manifests if not already present.
These probes can be configured with appropriate HTTP GET/POST requests or command-line checks to verify application health.

Health Check Script: Develop a script that runs after deployment, performing end-to-end testing such as hitting API endpoints, checking database connectivity, and ensuring no errors are logged in the application logs.

Post-Deployment Hook:

Introduce a post-deployment hook or use a custom operator to wait for health checks to pass before marking the deployment complete.
This can be implemented using Kubernetes Operator Framework or a simple bash script running on a sidecar container.

#### Example
Code and Workflow

Let's walk through an example with some sample code:

BASH
# Step 1: Update Deployment Manifest (example in YAML)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
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
        image: my-app-image
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10

# Step 2: Post-Deployment Health Check Script (bash)
#!/bin/bash
APP_URL=http://my-app-service:8080/healthz
HEALTH_THRESHOLD=3
CHECK_COUNT=0

while true; do
    RESPONSE=$(curl -s -o /dev/null -w "%{http_code}" $APP_URL)
    if [ "$RESPONSE" -eq 200 ]; then
        echo "Health check passed after $(($CHECK_COUNT * 5)) seconds."
        exit 0
    else
        ((CHECK_COUNT++))
        sleep 5
    fi
done

# Step 3: Integrate Post-Deployment Hook in CI/CD Pipeline
- name: Deploy Application
  uses: <some-deployment-tool>
  with:
    manifest_path: path/to/deployment.yaml

- name: Wait for Deployment to Be Healthy
  run: >
    ./path/to/post-deploy-health-check.sh

#### Closing Summary
and Common Mistakes/Follow-Ups

By integrating health checks, we ensure that the "deploy done" signal only fires after traffic is verified as healthy. This approach requires careful configuration of probes and scripts but significantly improves deployment reliability.

#### Common Mistakes

Overlooking the need for post-deployment health checks.
Not configuring appropriate liveness and readiness probes in the manifest.
Failing to script or automate the health check process.

#### Follow-Ups

Discussing monitoring tools like Prometheus for more detailed insights.
Exploring automated rollback mechanisms if health checks fail.

**Code:**

```

apiVersion: apps/v1 kind: Deployment metadata: name: my-app spec: replicas: 3 selector: matchLabels: app: my-app template: metadata: labels: app: my-app spec: containers: - name: my-app-container image: my-app-image livenessProbe: httpGet: path: /healthz port: 8080 initialDelaySeconds: 30 periodSeconds: 10 readinessProbe: httpGet: path: /ready port: 8080 initialDelaySeconds: 5 periodSeconds: 10

CODE
Copy

```bash
#!/bin/bash
APP_URL=http://my-app-service:8080/healthz
HEALTH_THRESHOLD=3
CHECK_COUNT=0

while true; do
    RESPONSE=$(curl -s -o /dev/null -w "%{http_code}" $APP_URL)
    if [ "$RESPONSE" -eq 200 ]; then
        echo "Health check passed after $(($CHECK_COUNT * 5)) seconds."
        exit 0
    else
        ((CHECK_COUNT++))
        sleep 5
    fi
done
```

