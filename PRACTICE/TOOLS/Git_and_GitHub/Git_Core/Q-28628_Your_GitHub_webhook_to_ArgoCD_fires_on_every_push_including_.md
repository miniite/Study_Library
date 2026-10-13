---
type: "practice"
difficulty: "Intermediate"
---

# Your GitHub webhook to ArgoCD fires on every push including docs-only commits, causing needless reconciles and rate-limit noise. How do you filter webhook events by path or event type so the controller only syncs on relevant manifest changes?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reducing Actions Costs](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reducing_Actions_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with Git and GitHub webhooks in a DevOps context, it's crucial to optimize workflows to reduce unnecessary operations and ensure that your CI/CD pipeline is not overly triggered by irrelevant events. In this scenario, you want to filter webhook events so that the Argo CD controller only syncs on relevant manifest changes, avoiding needless reconciles and rate-limiting issues.

To achieve this, we need to configure our GitHub webhook to send specific event types or paths, thereby filtering out unneeded commits like those affecting documentation. Let's break down how to set this up step-by-step from first principles.

**Code:**

```yaml
# Example GitHub webhook payload settings in Argo CD application resource
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: example-app
spec:
  source:
    repoURL: git@github.com:org/repo.git # Your repository URL
    path: "path/to/manifests"            # Path to your manifests in the repo
    targetBranch: master                 # Target branch for sync
  destination:
    server: https://k8s.example.com      # Kubernetes cluster API endpoint
    namespace: default                   # Namespace where applications are deployed
  syncPolicy:
    automated:
      prune: true                        # Prune unused resources on the cluster
    reconcileOptions:
      filterGitChangeEvents: true        # Enable filtering of Git change events
      gitChangePathsRegex: "*.yaml"      # Regex to match paths of relevant manifests
```

**Example input / output:**

```

To demonstrate, consider a typical webhook event payload:

JSON
Copy
{
  "ref": "refs/heads/master",
  "before": "94b7d1d3556c2a0a58e8b0a2f6c71b612bcbf123",
  "after": "9e6a58dbd9bda7df360b961e24ae0d98a8d1bfa6",
  "server": {
    "url": "https://github.com",
    "name": "GitHub"
  },
  "repository": {
    "url": "git@github.com:example/repo.git",
    "pusher": {
      "id": 34567890,
      "username": "octocat"
    }
  },
  "commits": [
    {
      "id": "9e6a58dbd9bda7df360b961e24ae0d98a8d1bfa6",
      "message": "Fix doc.md for release 1.0",
      "distinct": true,
      "timestamp": "2023-02-15T14:47:08Z",
      "author": {
        "username": "octocat"
      }
    }
  ]
}

With the filterGitChangeEvents and gitChangePathsRegex settings in place, Argo CD will only trigger a sync if changes are made to YAML files under the specified path.
```

**Flow:**

1. Yes
2. No
3. GitHub Webhook Trigger
4. Filter Git Change Events?
5. Argo CD Syncs RelevantManifest Changes
6. Rate Limiting and UnnecessaryReconciles

