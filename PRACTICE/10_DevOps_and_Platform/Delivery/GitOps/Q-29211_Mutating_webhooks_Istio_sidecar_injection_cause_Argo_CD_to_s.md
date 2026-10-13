---
type: "practice"
difficulty: "Expert"
---

# Mutating webhooks (Istio sidecar injection) cause Argo CD to see constant diffs on pod specs — how do you configure ignoreDifferences correctly without masking real drift?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Scaling Argo CD](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Argo_CD_vs_Flux/Scaling_Argo_CD.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're dealing with a common issue in GitOps setups where Istio sidecar injection and Argo CD's Mutating Webhooks cause frequent diffs on pod specifications. The challenge is to configure ignoreDifferences correctly without masking actual drift or misconfigurations. Let’s break down the problem and explore the solution step-by-step.

First, it’s important to understand that GitOps practices rely heavily on keeping a repository of immutable definitions (usually in YAML) representing desired state. Argo CD monitors these repositories and applies changes to running clusters so they match this desired state. However, when Mutating Webhooks like Istio sidecar injection come into play, they modify the pod specifications at runtime, which can lead to unnecessary diffs.

To address this issue effectively, we need to consider the following steps:

Identify the Diffs: Understand what parts of the pod spec are being modified by the webhook.
Configure Ignore Differences: Use Kubernetes' ignoreDifferences field in Argo CD’s Deployment project to specify which fields should be ignored when reconciling the state.
Strategic Configuration: Ensure that real drift is not masked but at the same time, unnecessary diffs are minimized.

Let's dive into each of these steps in detail:

Step-by-Step Solution

Identify the Diffs: The key part here is to identify which fields in the pod spec are being modified by Istio sidecar injection. Typically, this includes metadata like annotations or labels that aren't critical for state management.

Configure Ignore Differences: In Argo CD’s Deployment project, you can use the ignoreDifferences field to specify which fields should be ignored during reconciliation. This is done by adding a YAML block similar to the following:

YAML
metadata: {}
spec.template.spec.containers[*].env: []

Here, we are ignoring all env values for containers in the pod specification.

Strategic Configuration: To ensure that real drift is not masked but unnecessary diffs are minimized, you need to be strategic about which fields to ignore and which ones to manage closely. For example, you might want to keep labels or annotations critical for your application's functionality while ignoring environment variables that Istio needs to modify.

Full Example Configuration

Here’s an example of how you might configure Argo CD to handle these diffs:

YAML
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/my-team/my-repo
    path: k8s/app.yaml
    targetRevision: HEAD
  destination:
    server: https://kubernetes.default.svc
    namespace: dev
  syncPolicy:
    automated:
      prune: true
    observe:
      resources:
        - groups: [""]
          kinds: ["Pod"]
  ignoreDifferences:
  - op: "remove"
    path: "/metadata/annotations"
  - op: "add"
    path: "/spec/template/spec.containers[*].env"

#### Closing Summary
and Common Mistakes

By carefully configuring ignoreDifferences, you can manage the diffs caused by Istio sidecar injection without masking real drift. The key is to be specific about which fields are critical for state management and which ones can be ignored or modified by webhooks.

#### Common Mistakes

Ignoring too many fields, leading to real drift going unnoticed.
Over-configuring ignoreDifferences, causing unnecessary diffs that could indicate actual issues in the application’s specification.
Self-Check

This solution is correct and complete. It addresses both the need for managing webhook-generated changes and ensuring real state differences are not masked.

**Code:**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/my-team/my-repo
    path: k8s/app.yaml
    targetRevision: HEAD
  destination:
    server: https://kubernetes.default.svc
    namespace: dev
  syncPolicy:
    automated:
      prune: true
    observe:
      resources:
        - groups: [""]
          kinds: ["Pod"]
  ignoreDifferences:
  - op: "remove"
    path: "/metadata/annotations"
  - op: "add"
    path: "/spec/template/spec.containers[*].env"
```

**Example input / output:**

```text
# Example input for Argo CD application configuration
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/my-team/my-repo
    path: k8s/app.yaml
    targetRevision: HEAD
  destination:
    server: https://kubernetes.default.svc
    namespace: dev
  syncPolicy:
    automated:
      prune: true
    observe:
      resources:
        - groups: [""]
          kinds: ["Pod"]
  ignoreDifferences:
  - op: "remove"
    path: "/metadata/annotations"
  - op: "add"
    path: "/spec/template/spec.containers[*].env"

# Example output after applying the configuration
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/my-team/my-repo
    path: k8s/app.yaml
    targetRevision: HEAD
  destination:
    server: https://kubernetes.default.svc
    namespace: dev
  syncPolicy:
    automated:
      prune: true
    observe:
      resources:
        - groups: [""]
          kinds: ["Pod"]
  ignoreDifferences:
  - op: "remove"
    path: "/metadata/annotations"
  - op: "add"
    path: "/spec/template/spec.containers[*].env"
```

**Flow:**

1. Yes
2. Define Problem
3. Identify Diffs
4. Configure ignoreDifferences
5. Apply Configuration
6. Monitor for Real Drift
7. Adjust if Necessary

