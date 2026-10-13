---
type: "practice"
difficulty: "Deep"
---

# Your deploy job succeeds but half the pods run the old image. Where do you look?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Idempotent Deploy Jobs](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a CI/CD pipeline reports success, but only half of the Kubernetes pods are running the new image while the other half is still using the old one, this indicates an issue with how the rollout or update process was managed. To diagnose and resolve this problem, we need to follow a structured approach by examining several key areas: the deployment strategy, the state of your cluster, and any potential misconfigurations in your CI/CD pipeline.

Triaging the Problem

The question is essentially asking us to identify the root cause that could explain why half the pods are stuck with an outdated image despite the overall success of the deployment. The interviewer might be probing into our understanding of Kubernetes' update mechanisms, the intricacies of rolling updates and deployments, and our ability to debug a partially failed rollout.

#### Assumptions
The CI/CD pipeline is configured correctly.
The Kubernetes cluster is functioning properly.
We have access to monitoring tools and logs for troubleshooting.

#### First-Principles Solution

Review the Deployment Strategy:

Determine if you are using a rolling update strategy, blue/green deployment, or another method. For instance, in a rolling update scenario, Kubernetes will gradually replace old pods with new ones.

Inspect the Rollout Status:

Check the status of your rollout using kubectl rollout status. This command provides insights into how many revisions have been created and whether they are successfully applied to the specified number of replicas.

Check Pod Events and Logs:

Use kubectl describe pod <pod-name> to get detailed information about any issues that might be causing the pods to not update.
Review the logs for both old and new pods using kubectl logs <pod-name>. Look for errors or warnings that might indicate why some pods are failing to update.

Verify Configurations:

Ensure that your deployment YAML files correctly specify how many replicas should be updated at once and the rolling strategy (e.g., max unavailable, max unschedulable).
Check if there are any resource constraints or network issues that might prevent some pods from updating.

Cluster State and Network Considerations:

Confirm that all nodes in your cluster have access to the new image.
Verify if there are any network policies or service mesh configurations that could be blocking traffic to the new pod IP addresses.

#### Closing Summary

By following these steps, you can systematically identify the root cause of why only half of the pods are running the new image. The key is to methodically check each layer of your deployment setup—from the CI/CD pipeline configuration to the Kubernetes cluster state and network configurations. Common mistakes include incorrect rollout strategies, misconfigured resource limits, or overlooked issues with network policies.

#### Common Mistakes/Follow-Ups
Not thoroughly inspecting the rollout status.
Failing to review pod events and logs for clues about deployment failures.
Assuming all nodes have equal access to new images without verifying.
Overlooking potential network latency or service mesh restrictions that could impact pods' ability to update.

**Code:**

```shell
# Example command to check the rollout status
kubectl rollout status deployment/my-deployment

# Example command to describe a pod and get detailed logs
kubectl describe pod my-pod-1234567890
kubectl logs my-pod-1234567890
```

**Example input / output:**

```text
# Example output of kubectl rollout status
deployment "my-deployment" successfully rolled out

# Example pod describe and log output
POD_STATUS:
Conditions:
  Type            Status    Reason
  ----            ------    ------
  ContainersReady True      All containers are ready.

EVENTS:
  Type     Reason                 Age   From                Message
  ----     ------                 ----  ----                -------
  Normal   SuccessfulRevision    5m    deployment-controller  Revision succeeded: "v2"
```

