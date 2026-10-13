---
type: "practice"
difficulty: "Intermediate"
---

# How would you roll back a bad Kubernetes deployment when your manifests are managed in Git via GitOps?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Reverts in GitOps Rollouts](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reverts_in_GitOps_Rollouts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When rolling back a bad Kubernetes deployment that is managed via GitOps, the key steps involve identifying the problematic commit, reverting it, and ensuring a smooth rollback process without disrupting the ongoing operations.

#### Triage

The core of this question revolves around understanding how to manage deployments in a Git-based environment (GitOps) and handle rollbacks efficiently. The interviewer might be probing your knowledge of Git workflows, Kubernetes deployment strategies, and the GitOps methodology.

#### Assumptions
You are familiar with basic Git commands.
Your manifests are stored in a Git repository (e.g., GitHub, GitLab).
You have access to the necessary permissions to push changes.
The problematic deployment is tracked using annotations or labels in your Kubernetes manifests.
Step-by-Step Solution:

Identify the Problematic Commit:

Use kubectl commands to identify which commit caused issues. For example:
BASH
kubectl get pods -o json | jq '.items[].metadata.annotations' | grep 'commit='
Alternatively, you can use a tool like Kubeaudit or custom scripts that monitor Kubernetes resources and log changes.

Check the Logs:

Review logs to understand what went wrong:
BASH
kubectl logs <pod-name> -c <container-name>

Identify the Correct Commit:

Once you identify the problematic commit, find a previous commit that is known to be stable and working as expected.

Revert the Bad Deployment:

Navigate to your Git repository.
Checkout the previous good state:
BASH
git checkout <previous-good-branch>
Alternatively, use git reset or git revert commands if you want to avoid creating a merge commit:
BASH
git revert <bad-commit-hash>^ --no-edit  # Reverts the last bad commit

Push Changes:

Push the changes back to your Git repository:
BASH
git push origin <branch-name>

Apply the Updated Manifests to Kubernetes:

Deploy the updated manifests using a GitOps tool like Flux, ArgoCD, or Kustomize.
Ensure that the new configurations are applied without downtime if possible. For example:
BASH
flux apply --prerelease # Or use ArgoCD's CLI to sync changes

Verify the Deployment:

Monitor Kubernetes resources and application health after applying the updated manifests.
Use kubectl commands to verify that everything is running as expected:
BASH
kubectl get pods,deployments,services -o wide
Check for any errors or warnings in the logs.

Clean Up:

If necessary, delete the problematic commit and clean up any temporary files or resources.
Document what went wrong and the steps you took to fix it, so that similar issues can be avoided in the future.

#### Closing Summary

Rolling back a bad Kubernetes deployment managed via GitOps involves systematically identifying the problematic commit, reverting changes, redeploying updated manifests, and verifying everything is running smoothly. The key is to maintain consistency between your Git repository and Kubernetes clusters, ensuring seamless rollback processes without service disruptions.

#### Common Mistakes/Follow-Ups
Mistake: Overwriting a live deployment with untested code.

#### Follow-Up
Always revert changes using the previous good commit or branch to minimize risks.
Mistake: Failing to update Kubernetes resources promptly after reverting Git commits.

#### Follow-Up
Ensure automated tools like Flux, ArgoCD, or custom scripts are in place for continuous synchronization.

**Comparison:**

| Operation        | Command                          |
|------------------|---------------------------------|
| Checkout branch  | `git checkout <previous-good-branch>` |
| Revert commit    | `git revert <bad-commit-hash>^ --no-edit` |
| Push changes     | `git push origin <branch-name>`     |
| Apply manifests  | `flux apply --prerelease`           |

**Code:**

```bash
# Example commands
git checkout <previous-good-branch>
git revert <bad-commit-hash>^ --no-edit
git push origin <branch-name>

kubectl get pods -o json | jq '.items[].metadata.annotations' | grep 'commit='

flux apply --prerelease
```

**Example input / output:**

```text
# Example input and output
$ git checkout origin/main
Switched to branch 'origin/main'

$ git revert a9b7c8d90f0e # Reverting the problematic commit

$ kubectl get pods -o json | jq '.items[].metadata.annotations' | grep 'commit='
"commit": "1234567890abcdef"

# Output after rolling back to previous good state
$ kubectl get pods,deployments,services -o wide
```

**Data:**

```yaml
# Example manifest
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
  annotations:
    commit: abc1234567890 # Track the commit hash
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
        image: nginx:1.23
```

**Flow:**

1. Rendering diagram…

