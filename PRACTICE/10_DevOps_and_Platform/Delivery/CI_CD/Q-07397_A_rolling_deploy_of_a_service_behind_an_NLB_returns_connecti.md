---
type: "practice"
difficulty: "Deep"
---

# A rolling deploy of a service behind an NLB returns connection resets for a few seconds as old pods terminate; how would you use preStop hooks and deregistration delay to close that window?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Zero Downtime Deployments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/Zero_Downtime_Deployments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Rolling deploys can sometimes introduce connection resets due to the brief period during which old pods are terminating, while new ones are not yet available. To mitigate this issue, we can leverage preStop hooks in Kubernetes and implement a deregistration delay. This approach ensures that existing connections terminate gracefully before the old pod is fully removed from service.

#### Triaging the Question

The core of the question is about addressing connection resets during rolling deployments by utilizing preStop hooks and deregistration delays. The interviewer is probing your understanding of Kubernetes deployment strategies, lifecycle management, and how to minimize disruption in a service's availability.

#### Assumptions
We are using Kubernetes for managing our services.
The application is deployed behind an NLB (Network Load Balancer).
A rolling update strategy is being used with pod termination grace periods set appropriately.

#### First-Principles Solution

To close the window of connection resets, follow these steps:

Configure preStop Hooks: Use Kubernetes' preStop hooks to allow applications time to gracefully shut down.
Implement Deregistration Delay: Introduce a delay during which old pods remain registered with the NLB, ensuring that existing connections are not abruptly terminated.
Step 1: Configuring PreStop Hooks

PreStop hooks enable applications to run cleanup tasks before the termination of a pod. For example, this could involve flushing buffers or terminating ongoing requests.

YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-service
spec:
  template:
    spec:
      containers:
        - name: my-container
          image: my-app-image:latest
          lifecycle:
            preStop:
              exec:
                command: ["/bin/sh", "-c", "echo 'Gracefully shutting down...' && exit"]
Step 2: Implementing Deregistration Delay

To ensure old pods remain registered with the NLB for a brief period after termination, we can leverage Kubernetes' terminationGracePeriodSeconds and possibly custom logic within preStop hooks.

YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-service
spec:
  template:
    metadata:
      annotations:
        lifecyclehooks.k8s.io/terminate-delay: "5" # Delay in seconds for deregistration
    spec:
      containers:
        - name: my-container
          image: my-app-image:latest
          lifecycle:
            preStop:
              exec:
                command: ["/bin/sh", "-c", "echo 'Gracefully shutting down...' && sleep 3 && exit"]

#### Closing Summary
and Common Mistakes/Follow-Ups

By configuring preStop hooks and implementing a deregistration delay, we can minimize the window of connection resets during rolling deployments. This approach ensures that existing connections terminate gracefully before old pods are fully removed from service.

#### Common Mistakes

Failing to configure preStop hooks properly.
Not setting an appropriate termination grace period.
Ignoring the need for a deregistration delay, which can be critical in some scenarios.
Triaging Follow-Ups
What other strategies could you use to further improve service availability during rolling updates?
How would you handle cases where preStop hooks are not sufficient?

**Code:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-service
spec:
  template:
    metadata:
      annotations:
        lifecyclehooks.k8s.io/terminate-delay: "5"
    spec:
      containers:
        - name: my-container
          image: my-app-image:latest
          lifecycle:
            preStop:
              exec:
                command: ["/bin/sh", "-c", "echo 'Gracefully shutting down...' && sleep 3 && exit"]
```

**Flow:**

1. Yes
2. No
3. Deploy New Pod
4. NLB Routes Traffic to New Pod
5. Old Pods Begin Termination
6. Is preStop Hook Configured?
7. Graceful Shutdown Executed
8. Pod Marked for Removal, butStill Registered
9. Deregistration Delay Period
10. New Pod Takes Over Traffic
11. Connection Resets Possible

