---
type: "practice"
difficulty: "Deep"
---

# Your deploy tool reports success the moment the orchestrator accepts the manifest, not when pods are actually healthy and serving; how do you make 'deploy done' mean 'traffic verified'?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Deployment Verification](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Validating_Pipeline_Changes/Deployment_Verification.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue where our CI/CD tool reports a deployment as successful before all pods are healthy and serving, we need to ensure that the deployment is only marked as complete when traffic is verified. This involves several key steps from first principles:

Understanding the Current Workflow: The current setup reports success upon orchestrator acceptance of the manifest, which means it does not wait for the pods to be fully up and running or serving traffic.

Probing the Assumptions: We need to confirm that the goal is to ensure the deployment is both successful (pods are healthy) and that traffic is flowing correctly through these new deployments.

First-Principles Approach:

Health Checks for Pods: Implement or configure health checks on pods to verify they are ready.
Traffic Validation: Introduce a mechanism to validate that the deployed service is serving traffic, not just running pods.
Rolling Updates: Ensure any rolling updates can be controlled and monitored properly.

#### Solution
Steps:

Configure Pod Health Checks: Use liveness and readiness probes in Kubernetes to ensure only healthy pods are considered for serving traffic.
Service Mesh Integration: Integrate a service mesh like Istio, Envoy, or Linkerd to monitor traffic distribution and enforce retries and circuit breaking.
Traffic Shaping & Validation: Implement strategies such as canary deployments or blue/green deployments where new versions of the application are introduced gradually and validated before fully switching traffic.

Implementing Canary Deployments:

Create a Canary Release: Deploy a smaller fraction (e.g., 10%) of your user base to the new version.
Monitor Metrics: Use Prometheus, Grafana, or similar monitoring tools to track latency, errors, and success rates for both old and new versions.
Automated Rollbacks: Set up rules that automatically roll back to the previous version if metrics degrade beyond a certain threshold.

Automating Verification:

Post-Deployment Scripts: Write scripts or use existing CI/CD tools like Jenkins, GitLab CI, or GitHub Actions to run these checks after deployment.
Service Monitors: Define service monitors in Prometheus to continuously check the health of services and flag any issues.

#### Closing Summary
& Common Mistakes:

The key is a combination of robust health checks, traffic validation, and automated post-deployment verification.
Common pitfalls include not thoroughly testing canary deployments or failing to set up proper rollback mechanisms.

**Comparison:**

```
Step	Action	Tool/Framework
1	Configure pod health checks	Kubernetes
2	Integrate service mesh	Istio
3	Implement canary deployments	GitLab CI
4	Monitor metrics & validate	Prometheus
```

**Code:**

```bash
# Example YAML for Kubernetes liveness and readiness probes
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
spec:
  containers:
    - name: my-app-container
      image: my-app-image:v2
      livenessProbe:
        httpGet:
          path: /healthz
          port: 8080
      readinessProbe:
        httpGet:
          path: /ready
          port: 8080
```

**Example input / output:**

```text
Input:
- New deployment manifest accepted by orchestrator

Expected Output:
- Deployed pods marked as healthy and serving traffic
- Traffic validated through service mesh monitoring
- Canary release metrics stable, no rollback needed
```

**Data:**

```yaml
# Example service mesh configuration for Istio
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: my-service
spec:
  host: my-service
  trafficPolicy:
    loadBalancer:
      simple: ROUND_ROBIN
```

**Flow:**

1. Yes
2. No
3. Orchestrator Accepts Manifest
4. Pods Deployed
5. Are Pods Healthy?
6. Service Mesh Validates Traffic
7. Determine DeploymentSuccess
8. Mark Deployment as Done
9. Retry Health Checks

