---
type: "practice"
difficulty: "Intermediate"
---

# What could cause a pod to be stuck in CrashLoopBackOff and how would you debug it?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Troubleshooting 5xx Errors](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Troubleshooting_5xx_Errors.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a pod is stuck in CrashLoopBackOff on Kubernetes, it indicates that the container within the pod has repeatedly failed to start or exit successfully. This state can be caused by various issues, and debugging involves diagnosing these root causes through systematic steps.

#### Triage

The question asks about potential reasons for a pod entering CrashLoopBackOff, which is a common issue in Kubernetes clusters.
The interviewer is probing your understanding of container deployment, Kubernetes health checks, logging mechanisms, and troubleshooting practices.

#### Assumptions
include that you are familiar with basic Kubernetes concepts such as Pods, Containers, Deployments, and the importance of liveness and readiness probes.

#### First-principles solution

Review Pod Status: Check the pod’s status via kubectl get pods to confirm it is indeed in CrashLoopBackOff. Examine the events using kubectl describe pod <pod-name> for error messages indicating why the container failed.
Logs and Metrics: Inspect logs from the container to understand what went wrong during startup or execution. Use kubectl logs <pod-name>.
Check Configuration: Ensure that your deployment configuration (e.g., Deployment, DaemonSet) is correctly set up with proper liveness and readiness probes if required.
Resource Constraints: Verify that there are no resource constraints like memory or CPU limits causing the container to crash.
Image Issues: Confirm that the Docker image is valid and contains a script for the container’s entry point, which should be defined in the ENTRYPOINT field of your Dockerfile.
Dependency Problems: Investigate if any dependencies required by the application are missing or misconfigured.

#### Closing summary
By following these steps, you can systematically identify and address why a pod is stuck in CrashLoopBackOff, leading to its successful deployment and operation within the Kubernetes cluster.

#### Common mistakes
and follow-ups:

Missing important log entries due to insufficient verbosity or incorrect logging configuration.
Overlooking resource constraints that could lead to crashes.
Ignoring application-specific issues such as misconfigured environment variables or missing files.

**Code:**

```bash
# Example command to check pod status and events
kubectl get pods -o wide
kubectl describe pod <pod-name>
```

**Example input / output:**

```

Example:

SHELL
Copy
$ kubectl get pods
NAME                     READY   STATUS             RESTARTS   AGE
my-pod-6574b78df7-g9wz2   0/1     CrashLoopBackOff   3          2m

$ kubectl describe pod my-pod-6574b78df7-g9wz2
Name:               my-pod-6574b78df7-g9wz2
Namespace:          default
Priority:           0
Node:               node1/10.0.0.5
Start Time:         Wed, 01 Jan 2023 12:00:00 +0000
Labels:             app=my-app
                    pod-template-hash=6574b78df7
Annotations:        <none>
Status:             Running
IP:                 10.244.0.1
Controlled By:      Deployment/my-deployment
Containers:
  my-container:
    Container ID:   containerd://3e4d9...
    Image:          gcr.io/my-app:v1.0
    Image ID:       sha256:f87b...
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Error
      Exit Code:    137
      Started:      Wed, 01 Jan 2023 12:05:46 +0000
      Finished:     Wed, 01 Jan 2023 12:05:48 +0000
    Ready:          False
    Restart Count:  3
    Limits:
      cpu:     100m
      memory:  128Mi
    Requests:
      cpu:      100m
      memory:   64Mi
    Liveness:   http-get http://:8080/healthz delay=3s timeout=1s period=3s #success=1 #failure=3
    Readiness:  http-get http://:8080/ready delay=5s timeout=1s period=3s #success=2 #failure=3
    Environment:
      DB_USER:      <value>
      DB_PASSWORD:  <value>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-z948b (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True

Events:
  Type     Reason                  Age                From               Message
  ----     ------                  ----               ----               -------
  Normal   Scheduled               3m                 default-scheduler  Successfully assigned default/my-pod-6574b78df7-g9wz2 to node1
  Warning  FailedCreate            3m (x3 over 3m)    kubelet, node1     Error creating: Post "http://10.96.0.1:443/api/v1/namespaces/default/pods": service "kube-apiserver" is blocked by CloudFlare
  Normal   Pulling                 2m (x3 over 3m)    kubelet, node1     pulling image "gcr.io/my-app:v1.0"
  Warning  Failed                  2m (x3 over 3m)    kubelet, node1     Error: ErrImagePull
  Normal   BackOff                 2m (x3 over 2m)    kubelet, node1     Back-off pulling image "gcr.io/my-app:v1.0"
  Warning  Failed                  59s (x4 over 2m)   kubelet, node1     Error: ImagePullBackOff
```

**Flow:**

1. Review Pod Status
2. Check pod’s status
3. Inspect logs using `kubectllogs`
4. Verify container configurationand liveness probes
5. Check resource constraintswith 'kubectl get pod<pod-name> -o yaml'
6. Ensure Docker image is validand contains the correct entrypoint script
7. Investigate dependency issuesusing application-specific toolsor logs
8. Document findings and takecorrective actions

