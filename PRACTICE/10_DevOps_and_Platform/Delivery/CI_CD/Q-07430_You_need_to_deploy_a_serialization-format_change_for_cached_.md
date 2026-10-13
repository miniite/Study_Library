---
type: "practice"
difficulty: "Expert"
---

# You need to deploy a serialization-format change for cached objects, but old and new pods coexist during rollout and hit deserialization errors; how would you sequence the format change safely across a rolling deploy?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Database Migrations in Deployments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/Database_Migrations_in_Deployments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of deploying a serialization-format change in a CI/CD pipeline where old and new pods coexist, we need to ensure that both versions can handle cached objects without deserialization errors. This involves carefully sequencing the deployment process to manage the transition between different object formats.

The core idea is to use a rolling update strategy with proper configuration of both the application code and the environment to support backward compatibility during the transition period.

#### Triage

The key aspects this question probes are:

Understanding of CI/CD Pipelines: Knowledge of how deployments are orchestrated.
Handling Backward Compatibility: Ensuring that old systems can still work with new object formats without breaking.
Deployment Strategy: Choosing an appropriate strategy (e.g., rolling update, blue-green) and understanding its implications.

#### First-Principles Solution

#### Assumptions

We have a service where cached objects use a serialization format that needs to be updated.
Old pods will continue running while new ones are being deployed.
The application codebase can handle both formats during the transition period.

Deployment Strategy: Use a rolling update with staged traffic management.

Steps:

Prepare New Pods: Build and package the new version of the application that supports the updated serialization format.
Rolling Deployment: Gradually roll out new pods while keeping old ones running. Ensure each step in the deployment process is carefully monitored for deserialization errors.
Traffic Management: Use a proxy or gateway to route traffic to both versions of the service based on the presence of cached objects and their format.
Health Checks: Implement health checks to detect if a pod is experiencing issues due to the new format. If an issue is detected, revert the deployment incrementally.

Configuration:

Define environment variables or configuration files that can be toggled to enable the new serialization format in a controlled manner.
Use labels and selectors in Kubernetes to manage different versions of pods effectively.

Monitoring: Set up monitoring to track any issues related to deserialization errors during the transition period.

#### Closing Summary

By following this approach, we ensure that the service remains available and responsive throughout the deployment process. The rolling update strategy helps minimize downtime and potential data loss while allowing time for old and new systems to coexist smoothly.

#### Common Mistakes/Follow-Ups
Ignoring Health Checks: Failing to set up proper health checks can lead to a degraded service state without timely detection.
Inadequate Traffic Management: Not having a mechanism in place to route traffic based on the presence of cached objects can result in unexpected errors and downtime.
Overlooking Rollback Strategy: A well-thought-out rollback plan is crucial. If something goes wrong, you should be able to revert changes incrementally without causing disruption.

**Comparison:**

```
Phase	Action
1	Build & package new version of application.
2	Update deployment config to enable new format.
3	Deploy new pods with updated configuration.
4	Monitor for deserialization errors.
5	Rollback if issues are detected and health checks fail.
```

**Code:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app-container
        image: my-app:v1
        env:
          - name: NEW_FORMAT_ENABLED
            value: "false"
---
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: my-app
```

**Example input / output:**

```text
# Example input and output during the deployment process
Input (Before Update):
- Cache contains objects in old format.
- New pods are being deployed with updated serialization.

Output (After Initial Rollout):
- Old pods continue to handle cached objects in the old format.
- New pods start handling new format objects, but can still communicate with old ones.
- Traffic is split between old and new versions for gradual transition.
```

**Data:**

```yaml
# Example of how to toggle the format change in a configuration file
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  new_format_enabled: "false"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      containers:
      - env:
        - name: NEW_FORMAT_ENABLED
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: new_format_enabled
```

**Flow:**

1. Start Deployment
2. Deploy New Pods
3. Monitor for Errors
4. Rollback if Issues
5. End Deployment
6. Route Traffic to Both Versions
7. Migrate Traffic Gradually

