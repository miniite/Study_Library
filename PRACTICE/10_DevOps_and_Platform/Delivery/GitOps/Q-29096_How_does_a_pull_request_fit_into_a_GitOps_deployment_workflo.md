---
type: "practice"
difficulty: "Basic"
---

# How does a pull request fit into a GitOps deployment workflow?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Sync Notifications](<../../../../10_DevOps_and_Platform/Delivery/GitOps/GitOps_Fundamentals/Sync_Notifications.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In GitOps, pull requests (PRs) are a crucial part of ensuring that changes to infrastructure and application configurations are managed in a controlled, version-controlled manner. They allow teams to review code before it's merged into the main branch, thereby reducing the risk of unintended changes affecting production environments.

#### Triaging the Question

The question is asking about how pull requests fit into the GitOps deployment workflow. Here’s what we're probing:

Understanding of GitOps: GitOps is a practice that aims to manage infrastructure as code and ensure consistency between desired state (what you want) and current state (what actually exists).
Role of Pull Requests in GitOps: We need to explain how PRs are used to propose, review, and merge changes into the main configuration repositories.
Key Assumptions:
Infrastructure as Code (IaC): All infrastructure configurations are stored in code repositories like Git.
Automated Deployment Pipelines: Changes go through automated pipelines for validation before being applied.

#### First-Principles Solution
Step-by-Step Breakdown
Code Repository Setup: In a GitOps setup, the state of your cluster is codified using IaC tools (e.g., Terraform, Ansible) and stored in version control systems like GitHub or GitLab.
Proposing Changes: A developer proposes a change by creating a pull request against the main configuration branch. This PR contains the updated code for the desired state.
Code Review Process: Other team members can review the changes to ensure they are appropriate and do not introduce any issues. This step is crucial in maintaining quality and security.
Approval and Merging: Once the pull request receives approval, it gets merged into the main branch. However, before merging, it’s common practice to run a series of checks (e.g., CI/CD pipelines) to validate that the changes are safe and do not break anything.
Applying Changes: Upon successful validation, the changes are applied to the live environment through an automated deployment process.
Detailed Workflow

Initial State: The current state of the cluster is represented by code in a Git repository. For example:

YAML
apiVersion: v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
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

Developer Proposes a Change: A developer modifies this configuration to add more replicas or change the container image.

YAML
apiVersion: v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 5  # Increased from 3 to 5
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.9.0  # Updated to a newer version
        ports:
        - containerPort: 80

Code Review: The PR is created and reviewed by the team. The reviewer ensures that the new state does not conflict with any other changes or policies.

Validation via CI/CD Pipelines: Once approved, a series of checks are run:

Linting to ensure code follows conventions.
Syntax validation to check if it’s syntactically correct.
Simulation checks (if available) to simulate the deployment and verify there won’t be any issues in production.

Automated Deployment: If all checks pass, the changes are deployed:

BASH
git apply <path-to-pr-branch>
kubectl apply -f .

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, pull requests are essential for ensuring that all changes to infrastructure and application configurations go through a controlled review process. This helps maintain the integrity of your production environment by minimizing the risk of unintended changes.

#### Common Mistakes

Skipping code reviews.
Failing to integrate CI/CD pipelines.
Not validating the proposed state before deployment.

#### Follow-Ups

How do you handle conflicts in pull requests?
What tools or processes do you use for simulation and validation?
Can you walk me through a real-world example of integrating GitOps with Kubernetes?

**Comparison:**

|Step| Description|
|---|---|
|1| Code Repository Setup|
|2| Proposing Changes via PR|
|3| Code Review Process|
|4| Validation via CI/CD Pipelines|
|5| Automated Deployment|

**Code:**

```yaml
# Example IaC code before changes
apiVersion: v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
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

# Example IaC code after changes
apiVersion: v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 5  # Increased from 3 to 5
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.9.0  # Updated to a newer version
        ports:
        - containerPort: 80
```

**Example input / output:**

```text
# Example of applying changes via GitOps workflow

# Initial state (before PR)
$ kubectl get deployments
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           1d

# Developer creates a pull request with updated IaC code

# Code review and approval process

# Validation via CI/CD pipeline (if set up)

# Applying changes after validation
$ git apply <path-to-pr-branch>
$ kubectl apply -f .
```

**Data:**

```yaml
# Initial IaC configuration before changes
apiVersion: v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
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

# Updated IaC configuration after changes
apiVersion: v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 5  # Increased from 3 to 5
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.9.0  # Updated to a newer version
        ports:
        - containerPort: 80
```

**Flow:**

1. Code Repository
2. Propose Changes (PullRequest)
3. Code Review
4. Validation (CI/CD Pipeline)
5. Automated Deployment
6. Apply Changes (kubectl apply)

