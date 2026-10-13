---
type: "practice"
difficulty: "Deep"
---

# Your ArgoCD app is stuck in an OutOfSync loop even though nobody is pushing to the Git repo. Walk through how you'd trace whether the drift is from a mutating admission webhook, a controller writing back to the resource, or a bad ignoreDifferences config.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Concurrency Groups and Merge Queues](<../../../../TOOLS/Git_and_GitHub/Git_Core/Concurrency_Groups_and_Merge_Queues.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose why your ArgoCD app is stuck in an OutOfSync loop when no one is pushing to the Git repo, we need to systematically trace potential drift sources such as mutating admission webhooks, controller behavior, and misconfigured ignoreDifferences settings. Let's break this down step by step.

First, it’s important to understand that ArgoCD maintains a high-level reconciliation strategy: it compares the desired state (Git repo) with the actual running cluster state and applies changes to bring them in sync. If there’s drift, we need to identify its source to resolve the issue.

#### Triage
and Assumptions
No Git Pushes: Confirm that no new commits or pushes are happening on the Git repo.
App Status Check: Verify the ArgoCD application status via argocd app list to see if it’s indeed in an OutOfSync state.
Last Sync Time: Check when the last sync time was with kubectl get argo-cm -n <namespace>.
Event Logs: Inspect Kubernetes event logs using kubectl get events --sort-by='.metadata.creationTimestamp' to look for any recent errors or warnings.
Step 1: Mutating Admission Webhook

A mutating admission webhook might be altering the resources in a way that causes drift. To investigate:

Review the Webhooks: List all installed admission controllers using kubectl get validatingwebhookconfigurations and kubectl get mutatingwebhookconfigurations.
Inspect Configurations: Look at the YAML configurations for these webhooks to understand what they are doing.
Pod Logs: Check the logs of webhook pods for any errors or suspicious activity, such as kubectl logs <webhook-pod>.
Step 2: Controller Behavior

Controllers can write back to resources in ways that could cause drift. To investigate:

Controller Events: Look at controller-specific events using kubectl get events -n argo-cd --field-selector involvedObject.kind=Application,involvedObject.name=<app-name>.
Pod Logs: Examine the logs of ArgoCD controllers to see if they are making unexpected changes. Use kubectl logs <controller-pod>.
Step 3: IgnoreDifferences Configuration

Misconfigured ignoreDifferences settings can cause ArgoCD to ignore certain differences, leading to drift over time:

Check Configs: Review the application’s manifest file or Helm chart for any ignoreDifferences settings.
Policy Analysis: Ensure that these policies are correctly set and do not exclude changes that should be tracked.
Step 4: Cross-Verification

To ensure we haven’t missed anything, cross-check all findings:

Resource State: Use kubectl get pods -o wide, kubectl describe pods <pod-name>, and kubectl exec -it <pod-name> sh to inspect the resources directly.
ArgoCD Logs: Check ArgoCD logs for any synchronization issues using kubectl logs argocd-server.

#### Closing Summary

By systematically tracing through potential sources of drift, we can pinpoint the cause. The most likely culprits are mutating admission webhooks, controller behavior, or misconfigured ignoreDifferences settings.

#### Common Mistakes
and Follow-Ups
Missing Logs: Failing to check logs thoroughly.
Insufficient Context: Not reviewing all relevant configurations (webhook YAMLs, application manifests).
Overlooking Events: Ignoring Kubernetes events that provide critical context about what’s happening in the cluster.

Ensure you cover all bases by cross-verifying findings and considering edge cases. Always document your steps for future reference.

**Comparison:**

```
Resource	Type	Last Updated
argo-cm	Application	2 hours ago
example-app	App	3 minutes ago
Event	Time	Message
Warning	15:30	Error syncing application, message: "an error occurred while getting the current state for 'example-app':...
```

**Code:**

```
SH
Copy
# List mutating admission webhooks
kubectl get mutatingwebhookconfigurations

# Get ArgoCD application status
argocd app list

# Check last sync time of ArgoCD cluster manager
kubectl get argo-cm -n argo-cd

# Inspect Kubernetes events
kubectl get events --sort-by='.metadata.creationTimestamp'

# Logs of webhook pod
kubectl logs <webhook-pod>

# Controller-specific events
kubectl get events -n argo-cd --field-selector involvedObject.kind=Application,involvedObject.name=<app-name>
```

**Example input / output:**

```text
# Example output of kubectl get events --sort-by='.metadata.creationTimestamp'
LAST SEEN   TYPE      REASON     OBJECT          MESSAGE
2m30s       Warning   FailedSync Application example-app  Error syncing application, message: "an error occurred while getting the current state for 'example-app': error in creating pod spec, error: Deployment.apps "example-app" is forbidden: unable to validate object: [invalid value:, field=spec.template.spec.containers[*].env[*], reason=MutatingWebhookConfiguration/mutating-webhook-configuration.example.com is invalid: rules[0].operations: Invalid value: []string{"CREATE", "UPDATE"}: cannot include CREATE operation in the rules for a mutating webhook. Valid operations are: DELETE, UPDATE, WATCH]", source={"component":"sync","file":"/go/src/sigs.k8s.io/cluster-api-provider-aws/pkg/controllers/cluster.(*ClusterController).sync","func":"github.com/aws/eks-anywhere/cluster-api-controller/pkg/controllers.cluster.sync"},{"component":"controller","file":"/go/src/k8s.io/controller-manager/app/root_controller.go","func":"main.main"}; requeue_after: 1m0s
```

**Data:**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: example-app
spec:
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  source:
    repoURL: git@github.com:user/example-repo.git
    path: apps/cluster1
  ignoreDifferences:
  - group: apps/v1
    kind: Deployment
    fields: spec.template.spec.containers[*].env[*].name
```

**Flow:**

1. No
2. Yes
3. Start
4. Is there a recent Git push?
5. Mutating AdmissionWebhooks
6. Check webhook logs andconfigurations
7. Controllers
8. Inspect controller behavior,including sync events and logs
9. IgnoresDifferencesConfigurations
10. Review application manifestfor misconfigurations
11. Cross-verify all findings

