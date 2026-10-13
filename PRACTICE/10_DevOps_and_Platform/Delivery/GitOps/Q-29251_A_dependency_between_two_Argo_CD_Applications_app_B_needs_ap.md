---
type: "practice"
difficulty: "Deep"
---

# A dependency between two Argo CD Applications (app B needs app A's service) causes flapping health on deploy — how do you express cross-application ordering?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Scaling Argo CD](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Argo_CD_vs_Flux/Scaling_Argo_CD.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

In a GitOps environment using Argo CD, ensuring that one application depends on another can sometimes lead to unexpected behavior like deployment flapping. To address this issue, we need to introduce cross-application ordering constraints in our deployment strategy. This involves configuring Argo CD's sync policies and possibly leveraging GitOps principles more rigorously.

#### Triage

The question is essentially about managing the sequence of deployments between two Argo CD applications where one application depends on another’s service. The core challenge here is ensuring that app B does not deploy until app A has successfully deployed and its service is up. This can be achieved by configuring appropriate sync policies in Argo CD to enforce order, potentially using Helm charts or custom GitOps practices.

#### Assumptions
Both applications are managed under a single GitOps repository.
We use Argo CD for continuous delivery of the applications.
The dependency between app A and app B is necessary but causes deployment flapping due to misconfigured ordering.
Step-by-Step Solution:
Identify the Dependency: First, we need to identify that app B depends on a service provided by app A. This dependency could be inferred from application code or manifest files.
Configure Sync Policies:
In Argo CD, each application has its own sync policy defined in a YAML file (e.g., sync-policy.yaml). We can modify this to include dependencies.
For example, if app B depends on the availability of app A, we might need to add a pre-sync hook or use annotations in Argo CD’s manifest files.
Use Pre-Sync Hooks:
Pre-sync hooks allow us to run custom scripts before starting the sync process. We can write a script that checks if app A is up and running before allowing app B to proceed with its deployment.
Leverage Helm Charts (if applicable):
If both applications are deployed via Helm charts, we can use preUpgrade hooks in the values.yaml file of app B. These hooks ensure that any preconditions (like ensuring app A is running) are met before the upgrade.
Custom GitOps Practices:
Implement a more rigorous GitOps practice where changes to app A trigger a webhook that updates app B only after app A has successfully deployed.

#### Example
Configuration:

Here’s an example of how you might configure pre-sync hooks in Argo CD:

YAML
# sync-policy.yaml for app B
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: appB
spec:
  project: default
  source:
    repoURL: https://github.com/your-repo/appB.git
  destination:
    server: https://kubernetes.default.svc
    namespace: appB-namespace
  syncPolicy:
    automated:
      prune: true
    retry:
      limit: 60
    syncOptions:
      - "prune=true"
  # Pre-sync hook to ensure app A is up and running
  preSync: |
    /path/to/script.sh --check-app-A

#### Closing Summary
& Common Mistakes/Follow-Ups:

In summary, ensuring cross-application ordering in Argo CD requires configuring sync policies with appropriate hooks or leveraging Helm charts for custom pre-upgrade checks. Common mistakes include not properly identifying the dependency, neglecting to implement reliable health checks, and misconfiguring GitOps practices.

#### Follow-ups
might involve discussing alternative solutions like using Kubernetes-native resources (like MutatingWebhookConfigurations) or integrating Argo CD with CI/CD pipelines that enforce deployment order based on artifact availability.

**Code:**

```yaml
# Example sync-policy.yaml for app B
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: appB
spec:
  project: default
  source:
    repoURL: https://github.com/your-repo/appB.git
  destination:
    server: https://kubernetes.default.svc
    namespace: appB-namespace
  syncPolicy:
    automated:
      prune: true
    retry:
      limit: 60
    syncOptions:
      - "prune=true"
  preSync: |
    /path/to/script.sh --check-app-A
```

**Example input / output:**

```text
# Example script for pre-sync check in app B
#!/bin/bash
if ! kubectl get svc appA-service -n appA-namespace; then
  echo "App A is not ready. Aborting sync."
  exit 1
fi
```

**Data:**

```yaml
# Example values.yaml for app B with preUpgrade hook
preUpgrade: |
  #!/bin/bash
  if ! kubectl get svc appA-service -n appA-namespace; then
    echo "App A is not ready. Aborting upgrade."
    exit 1
  fi
```

**Flow:**

1. Yes
2. No
3. Sync App B
4. Is App A Ready?
5. Proceed with Sync
6. Wait for App A to Be Ready

