---
type: "practice"
difficulty: "Intermediate"
---

# How does a GitHub Actions workflow deploy an updated Kubernetes manifest, and what does 'kubectl apply' do with it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Reducing Actions Costs](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reducing_Actions_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

In this response, we'll explore how a GitHub Actions workflow can be used to deploy updated Kubernetes manifests using kubectl apply. We'll start by understanding what kubectl apply does and then dive into creating a simple GitHub Actions workflow that handles the deployment.

#### Triaging the Question

The question asks about two key components:

GitHub Actions Workflow: This is an automated process defined in your repository to run when specific events occur (like code pushes).
kubectl apply: A command used to update resources from a manifest file, ensuring that the actual state matches the desired state.
Core Idea

The key idea here is understanding how kubectl apply works and integrating it into a GitHub Actions workflow for seamless Kubernetes deployments.

Step-by-Step Solution
What is kubectl apply?

kubectl apply takes a YAML or JSON manifest file as input, compares its contents with the current state of the cluster, and updates existing resources if necessary. If the manifest specifies new resources that do not exist, it will create them. This command ensures that your Kubernetes objects are always in sync with your desired configuration.

GitHub Actions Workflow

A simple GitHub Actions workflow can be set up to automatically deploy changes whenever a push is made to the repository containing your manifests. Here’s how you can achieve this:

Trigger: The workflow will run when someone pushes code to a specific branch.
Environment Setup: Ensure kubectl is installed and configured with the correct context (Cluster, Namespace).
Apply Manifests: Use kubectl apply -f to update resources in the Kubernetes cluster.

#### Example
Workflow File

Below is an example of a GitHub Actions workflow file that deploys updated manifests:

YAML
name: Deploy to Kubernetes

on:
  push:
    branches:
      - main # or any other branch you want to trigger on

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Set up kubectl
      run: |
        echo "Installing kubectl..."
        curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/stable/server/binaries/linux/amd64/kubectl)
        chmod +x ./kubectl
        sudo mv ./kubectl /usr/local/bin/kubectl

    - name: Deploy manifests
      run: |
        echo "Applying updated Kubernetes manifests..."
        kubectl apply -f kubernetes-manifests/
Explanation of the Workflow:
Checkout Repository: This step checks out your repository to get access to the files.
Set up kubectl: Download and install kubectl.
Deploy Manifests: Use kubectl apply to update Kubernetes resources.

#### Closing Summary

In summary, a GitHub Actions workflow can be used to automate deployments by setting up a job that runs whenever you push code. The job installs kubectl, ensuring it is set up correctly with the necessary permissions and context for your cluster. Then, it applies updated manifests using kubectl apply.

#### Common Mistakes
and Follow-Ups

Mistake: Not configuring the correct kubectl context or namespace.

#### Solution
Ensure your workflow sets up the right context before running kubectl apply.

Mistake: Failing to include necessary permissions in GitHub Actions.

#### Solution
Grant appropriate roles and secrets to your actions to run kubectl commands.

#### Follow-Up
How would you handle rollback scenarios if an update fails?

Answer: You could add a step to save the previous manifest state before applying changes, so that in case of failure, you can revert back.

#### Follow-Up
What are some best practices for using kubectl apply?

Answer: Use it with caution and consider using labels or annotations to manage versions or rollbacks.

**Code:**

```yaml
name: Deploy to Kubernetes

on:
  push:
    branches:
      - main # or any other branch you want to trigger on

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Set up kubectl
      run: |
        echo "Installing kubectl..."
        curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/stable/server/binaries/linux/amd64/kubectl)
        chmod +x ./kubectl
        sudo mv ./kubectl /usr/local/bin/kubectl

    - name: Deploy manifests
      run: |
        echo "Applying updated Kubernetes manifests..."
        kubectl apply -f kubernetes-manifests/
```

