---
type: "practice"
difficulty: "Intermediate"
---

# What is configuration drift and how does GitOps detect it?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Reconciliation Failures](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Reconciliation_Failures/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Configuration drift in a DevOps context refers to the discrepancy between the desired state of an application or infrastructure as defined by code in version control (like your Git repository) and its actual deployed state on your cluster, such as Kubernetes clusters. In other words, it's when your production environment diverges from what is specified in your configuration files.

#### Triaging the Question

The interviewer is probing into your understanding of how GitOps works to ensure that infrastructure or application configurations are always up-to-date and consistent with the desired state defined by code. This involves recognizing how to detect these discrepancies, which can lead to unexpected behaviors or security vulnerabilities in production environments. We'll assume the current configuration on a Kubernetes cluster has drifted from what is specified in the Git repository.

Step-by-Step Solution
Understanding Desired State: In GitOps, you define the desired state of your infrastructure and applications using declarative configuration files stored in your Git repository.
Actual State Monitoring: The actual state of these configurations needs to be continuously monitored to ensure it matches what is declared in the Git repository.
Detection Mechanisms:
Event-driven Architecture: GitOps often uses event-driven architectures where any change to the configuration files triggers a series of events that need to be processed.
Difference Detection Tools: Tools like kubeval or kubetools can be used to compare the desired state with the actual state, highlighting any differences.
Webhooks and Automation Pipelines: Webhooks can notify continuous integration/continuous deployment (CI/CD) pipelines when a change occurs in the Git repository. These pipelines then run scripts or commands to check the current state of the cluster against the desired state.
Handling Drift:
Rollback Mechanisms: If drift is detected, mechanisms like rolling back changes or automatically updating the actual state to match the desired state can be implemented using tools and automation.
Alerting and Notifications: To prevent unnoticed issues, alerts and notifications should be configured when significant drifts are detected.

#### Closing Summary

In summary, GitOps aims to minimize configuration drift by ensuring that all changes are driven from a central source of truth (your Git repository) and continuously validated against the actual state. This approach helps maintain consistency between your development environment and production, reducing the risk of unexpected behaviors or security vulnerabilities.

#### Common Mistakes/Follow-ups
Overlooking Continuous Monitoring: Failing to set up continuous monitoring can result in undetected drifts.
Ignoring Event Handling: Not properly handling events from Git repository changes can lead to delayed or missed validation checks.
Not Implementing Rollback Mechanisms: Without robust rollback mechanisms, once drift is detected, manual intervention may be required, leading to downtime and increased risk.

**Comparison:**

```
Scenario	Expected Action
Replicas Mismatch	Update deployment to match desired replica count.
Image Tag Change	Revert or update image in the manifest file and redeploy.
Service Port Change	Update service configuration with new port number and apply changes.
```

**Code:**

```bash
# Example script for detecting configuration drift using kubetools

#!/bin/bash

# Fetch current state from cluster
current_state=$(kubectl get all -o yaml)

# Get desired state from Git repository
desired_state=$(git checkout main && git pull origin main && cat config.yaml)

# Compare states and output differences
echo "Current State: $current_state"
echo "Desired State: $desired_state"

# Use kubetools to highlight drift
kubetools diff --current="$current_state" --desired="$desired_state"
```

**Example input / output:**

```text
# Example input -> output for drift detection

Input (Current State):
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: example-app
    spec:
      containers:
      - name: example-container
        image: busybox

Input (Desired State):
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: example-app
    spec:
      containers:
      - name: example-container
        image: busybox

Output (Drift):
Replica count mismatch detected. Desired state specifies 2 replicas, but current state has 3 replicas.
```

**Data:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: example-app
    spec:
      containers:
      - name: example-container
        image: busybox
        command: ["sh", "-c", "echo Hello world"]
---
apiVersion: v1
kind: Service
metadata:
  name: example-service
spec:
  ports:
  - port: 80
```

**Flow:**

1. Yes
2. No
3. Config Change Detected
4. Is Desired State Different fromActual?
5. Alert DevOps Team for ManualFix
6. Apply Patch to Correct DriftAutomatically
7. Verify Configuration is Now inSync

