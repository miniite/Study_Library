---
type: "practice"
difficulty: "Deep"
---

# During a Kubernetes upgrade you need to know which manifest revision each namespace last successfully synced, but Git history alone doesn't tell you what's live. How do you correlate the live cluster state to a Git SHA using Argo's sync history or a commit annotation?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reproducible Builds from Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reproducible_Builds_from_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to establish a clear understanding of how Kubernetes namespaces and Git are used together in an environment where Argo CD is employed for continuous delivery. The key challenge here is correlating the live state of each namespace with its corresponding Git revision (SHA). This can be achieved by leveraging Git annotations or Argo's sync history.

First, let’s break down the problem:

Git History: Provides a chronological log of all changes made to the repository.
Live Cluster State: Represents the current configuration and status of deployed applications in Kubernetes namespaces.
Argo CD: Manages deployments by syncing GitOps manifests with live clusters.
Commit Annotations: Can be used to tag specific Git commits with metadata, including information about which cluster namespace they were applied to.

Given these elements, we need a method to correlate the manifest revisions in Git with the live state of each namespace. Here’s how we can achieve this:

Review Argo CD Sync History:

Argo CD keeps track of sync events and can be queried via its API.
We can query the Argo CD API to get a list of all sync operations, including those that were successful.

Extract Annotations from Git Commits:

Commit annotations provide metadata directly in Git, making it easier to link specific commits to cluster namespaces.
For example, we might add an annotation like argo-cd/namespace to each relevant commit.

Correlate Sync Events with Cluster Namespaces:

Use the Argo CD API to get a list of sync operations and their timestamps.
Cross-reference these timestamps with the Git history using annotations.

Here’s a step-by-step approach:

Query the Argo CD API for successful sync events:
BASH
argocd app list --namespace <NAMESPACE> | grep 'Synced'
Extract the latest commit SHA from each sync event.
Use the extracted SHAs to query Git history and extract annotations:
BASH
git log --grep='argo-cd/namespace:<NAMESPACE>'

Let’s assume you have a sample Argo CD application named my-app with two namespaces, dev and prod. Here’s an example of how this might look in practice:

Argo CD API Query:

BASH
argocd app list --namespace dev | grep 'Synced'  # Output: Synced at 2023-10-01T14:56:02Z
argocd app list --namespace prod | grep 'Synced'  # Output: Synced at 2023-10-01T15:23:07Z

Extract Git SHAs:

BASH
git log -1 --grep='argo-cd/namespace:dev'
git log -1 --grep='argo-cdev-cd/namespace:prod'

This process will yield the SHAs of the commits that were last successfully synced to each namespace.

Correlate SHAs and Timestamps: Use a script or tool to correlate these timestamps with the commit SHAs, creating a mapping from dev namespace to its corresponding Git SHA and vice versa.
BASH
#!/bin/bash

# Example function to map sync events to Git SHAs
map_sync_to_sha() {
    local namespace=$1
    local sync_timestamp=$(argocd app list --namespace $namespace | grep 'Synced' | awk '{print $5}')

    # Query Git history for the latest commit that matches this namespace
    git log -1 --grep="argo-cd/namespace:$namespace" --pretty=format:"%H"
}

# Example usage
map_sync_to_sha dev  # Output: <SHA>
map_sync_to_sha prod  # Output: <SHA>

By following these steps, you can effectively correlate the live cluster state with the Git SHAs, ensuring that you know which manifest revision each namespace last successfully synced.

**Comparison:**

| Namespace | Sync Timestamp          | Latest Git SHA       |
|-----------|-------------------------|---------------------|
| dev       | 2023-10-01T14:56:02Z    | SHA-of-last-synced-dev |
| prod      | 2023-10-01T15:23:07Z    | SHA-of-last-synced-prod |

**Code:**

```bash
# Example script to map sync events to Git SHAs
#!/bin/bash

map_sync_to_sha() {
    local namespace=$1
    local sync_timestamp=$(argocd app list --namespace $namespace | grep 'Synced' | awk '{print $5}')

    # Query Git history for the latest commit that matches this namespace
    git log -1 --grep="argo-cd/namespace:$namespace" --pretty=format:"%H"
}

# Example usage
map_sync_to_sha dev  # Output: <SHA>
map_sync_to_sha prod  # Output: <SHA>
```

**Example input / output:**

```text
Input:
argocd app list --namespace dev | grep 'Synced'
Output:
Synced at 2023-10-01T14:56:02Z

Input:
argocd app list --namespace prod | grep 'Synced'
Output:
Synced at 2023-10-01T15:23:07Z

Input:
git log -1 --grep="argo-cd/namespace:dev" --pretty=format:"%H"
Output:
SHA-of-last-synced-dev

Input:
git log -1 --grep="argo-cd/namespace:prod" --pretty=format:"%H"
Output:
SHA-of-last-synced-prod
```

**Data:**

```yaml
- app: my-app
  namespace: dev
  sync_timestamp: "2023-10-01T14:56:02Z"
- app: my-app
  namespace: prod
  sync_timestamp: "2023-10-01T15:23:07Z"
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Query Argo CD API for syncevents
6. Extract timestamps
7. Map timestamps to Git SHAs
8. Namespace: 'dev'
9. Fetch SHA using grep
10. Namespace: 'prod'
11. Fetch SHA using grep
12. Output results

