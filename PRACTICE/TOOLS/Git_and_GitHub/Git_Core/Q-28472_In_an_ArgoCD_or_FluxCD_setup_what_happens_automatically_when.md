---
type: "practice"
difficulty: "Intermediate"
---

# In an ArgoCD or FluxCD setup, what happens automatically when you merge a change to the manifests in Git?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Concurrency Groups and Merge Queues](<../../../../TOOLS/Git_and_GitHub/Git_Core/Concurrency_Groups_and_Merge_Queues.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you merge changes to your Kubernetes manifest files in Git (specifically using tools like ArgoCD or FluxCD), several automated steps are triggered. These tools continuously sync your local cluster state with the latest code from your Git repository, ensuring that the actual environment mirrors what’s defined in your manifests.

To understand this process better, we need to break down the key components and how they interact:

Git Repository: This is where your Kubernetes manifest files are stored.
CI/CD Pipeline or Synchronization Tool (e.g., ArgoCD or FluxCD): These tools monitor the Git repository for changes and automatically apply those changes to the cluster.

Here’s a step-by-step explanation of what happens when you merge changes in Git:

#### Triaging the Question

The question is essentially asking about the automation flow that occurs post-merge, specifically within the context of ArgoCD or FluxCD. The interviewer might be probing your understanding of continuous deployment concepts and how they are implemented in a GitOps workflow.

#### Assumptions
Continuous Deployment Environment: The environment where you’re deploying Kubernetes resources.
Git Repository: Typically private or public on GitHub, GitLab, Bitbucket, etc.
ArgoCD/FluxCD Configuration: Properly set up and configured to watch specific branches and/or repositories.

#### Solution
From First Principles

Repository Monitoring:

Both ArgoCD and FluxCD continuously monitor the specified Git repository for changes in real-time using Webhooks or native Git polling.

Change Detection:

When a merge occurs (commit is pushed), these tools detect the change and trigger a sync process.

Sync Process:

The sync process begins by fetching the latest changes from the specified branch.
It then validates the manifests for any issues, such as syntax errors or missing dependencies.
If valid, it applies the updated manifests to the cluster using Kustomize, Helm, or other deployment strategies configured in your tool’s setup.

Resource Management:

Any deleted resources are removed from the cluster.
New or updated resources are created, ensuring the cluster state matches the Git repository.

Error Handling and Logging:

If any issues arise during sync (e.g., invalid manifests), these tools typically log errors and notify the user through their UIs or by sending alerts via integrations like Slack or email.

Sync Strategy:

ArgoCD and FluxCD can be configured to apply changes in different ways, such as reconciling resources one-by-one or applying all changes at once.

#### Closing Summary

In summary, when you merge a change into your Git repository using tools like ArgoCD or FluxCD, the process involves continuous monitoring of the repository for updates. Upon detecting a merge, these tools validate and apply the updated Kubernetes manifests to your cluster, ensuring that the actual environment aligns with the desired state defined in your code.

#### Common Mistakes/Follow-Ups
Failing to Validate Manifests: Ensuring all changes are validated before applying can prevent errors.
Handling Large-Scale Changes: Managing large-scale deployments without proper testing or rollback strategies can lead to disruptions.
Resource Conflicts: Overwriting resources that might be in use by other applications or processes.

**Comparison:**

```
Step	Action
1	Monitor Git repository for changes.
2	Detect merge event.
3	Fetch latest manifests from Git.
4	Validate manifest files.
5	Apply updated manifests to cluster.
```

**Code:**

```python
# Example of a simple script to simulate Git commit detection and sync process

def git_commit_detected(repository, branch):
    print(f"Commit detected on {repository} - {branch}")

def apply_manifests(manifest_file):
    # Simulate applying manifests
    print(f"Applying manifests from {manifest_file}")

git_commit_detected("repo-name", "main")
apply_manifests("/path/to/manifests.yaml")
```

**Example input / output:**

```text
Input (Git commit detected on repo-name - main)
Output (Applying manifests from /path/to/manifests.yaml)

Input (Manifest validation successful)
Output (Resources created or updated in cluster)
```

**Data:**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: example-app
  namespace: argo
spec:
  project: default
  source:
    repoURL: https://github.com/user/repo.git
    targetRevision: main
  destination:
    server: https://kubernetes.default.svc
    namespace: staging
```

**Flow:**

1. Yes
2. No
3. Monitor Git Repository
4. Commit Detected?
5. Validate Manifests
6. Apply Updated Manifests
7. Wait for Next Poll/Notification

