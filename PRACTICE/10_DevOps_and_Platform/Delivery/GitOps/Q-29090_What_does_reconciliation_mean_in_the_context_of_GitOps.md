---
type: "practice"
difficulty: "Basic"
---

# What does 'reconciliation' mean in the context of GitOps?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Resilience to Git Outages](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Reconciliation_Failures/Resilience_to_Git_Outages.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

Reconciliation in GitOps refers to the process by which a DevOps tool ensures that the state of the infrastructure matches the desired state defined in the configuration files stored in version control. It's like ensuring your actual setup mirrors the blueprint you have stored.

#### Triaging the Question

The question is essentially asking about how a DevOps practice, specifically GitOps, maintains consistency between the codebase and the deployed environment. The reconciliation process is critical because it automatically updates the infrastructure to match the state defined in the configuration files, thereby ensuring that the actual deployment matches the intended design.

#### Assumptions
We assume the reader has a basic understanding of version control systems (like Git) and the concept of deploying applications.
Reconciliation involves comparing the current state with the desired state and making necessary changes to align them.

#### First-Principles Solution
Step 1: Understanding Desired State

In GitOps, the desired state is defined in configuration files stored in a version control system (VCS). These files typically use declarative formats like YAML or JSON. For example:

YAML
apiVersion: v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
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
Step 2: Detecting State Discrepancies

The reconciliation process uses a tool or service to periodically check the current state of the infrastructure against these configuration files. This step involves fetching the actual deployed state and comparing it with what is defined in Git.

Step 3: Applying Changes

If discrepancies are detected, the tool will apply changes to bring the infrastructure into alignment with the desired state. This can involve automated updates or sending instructions to the underlying infrastructure management tools (e.g., Kubernetes) to make the necessary adjustments.

#### Closing Summary

In summary, reconciliation in GitOps is a fundamental process that ensures the deployed environment matches the configuration defined in code. By continuously comparing and adjusting the actual state with the desired state, it helps maintain consistency and reliability in deployments.

#### Common Mistakes/Follow-Ups
Mistake 1: Assuming Reconciliation Only Happens During Deployment
Reconciliation occurs periodically, not just during deployment.

#### Follow-Up
1: Explaining Continuous vs. Batch Reconciliation
Discuss the trade-offs between continuous reconciliation (faster updates but more frequent errors) and batch reconciliation (slower updates but fewer errors).
Triaging Further:
Understanding how different GitOps tools handle reconciliation can be a good follow-up.
The impact of network latency or downtime during reconciliation should also be considered.

**Comparison:**

```
Step	Description
1	Define desired state in Git.
2	Detect current state through API calls or tools.
3	Apply changes to align infrastructure with Git config.
```

**Code:**

```yaml
apiVersion: v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
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

1. Define Desired State
2. Detect Current State
3. Apply Changes
4. Check Success
5. Reconciliation Complete

