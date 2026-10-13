---
type: "practice"
difficulty: "Deep"
---

# Your incident-automation must run a given remediation exactly once at its scheduled maintenance window across a fleet, tolerating node failure, without duplicate or missed runs. Explain the leader-election and lease design and the concurrencyPolicy and startingDeadline semantics if you build it on CronJobs.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Leader Election](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/Leader_Election.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that an incident-automation job runs exactly once per maintenance window across a fleet of nodes, while tolerating node failures and preventing duplicates or missed runs, we need to design a robust system involving leader election and lease management. The key components include the concurrencyPolicy and startingDeadlineSeconds semantics in Kubernetes CronJobs.

TL;DR

The solution involves using Kubernetes CronJobs with appropriate concurrency policies and lease designs to ensure that only one instance of the job runs at any given time, even across node failures and multiple maintenance windows. This is achieved through leader election for managing the job's execution on a single node and setting up proper leases to handle temporary node outages.

#### Triaging the Question

The question asks us to design a system where an incident-automation job (represented as a CronJob in Kubernetes) runs exactly once per maintenance window across a fleet of nodes. This requires handling:

Node Failures: Ensuring that if one node fails, another can take over.
Concurrency Control: Preventing multiple instances from running concurrently to avoid data inconsistencies or overlapping operations.

#### First-Principles Solution
Step 1: Understanding CronJobs and ConcurrencyPolicy

Kubernetes CronJobs are designed to run jobs on a schedule. The concurrencyPolicy determines how multiple instances of the same job should be handled:

Allow: Multiple instances can run concurrently.
Forbid: Only one instance is allowed at any time, but subsequent runs will wait until the previous one completes.
Replace: If a new instance starts while an existing one is running, it replaces the old instance and cancels its execution.

Given our requirement to prevent duplicate or missed runs, we need to choose Forbid for ensuring that only one job instance executes at any time across all nodes.

Step 2: Leader Election Mechanism

Leader election ensures that a single node in the fleet is responsible for executing the job. This mechanism involves:

Election: Nodes compete to be elected as the leader.
Lease Management: The leader maintains its leadership by periodically renewing a lease, which can be used to detect failures.
Step 3: Leases and Deadlines

To manage node failures gracefully:

leasesDurationSeconds: Time interval between lease renewals. If no renewal is received within this period, the node is considered failed.
startingDeadlineSeconds: Time duration after which a job instance should be terminated if it has not started yet.
Step 4: Putting It All Together
CronJob Configuration:
YAML
apiVersion: batch/v1
kind: CronJob
metadata:
  name: incident-automation
spec:
  schedule: "0 3 * * *"  # Run at 3 AM every day
  concurrencyPolicy: Forbid
  failedJobsHistoryLimit: 1
  successfulJobsHistoryLimit: 3
  startingDeadlineSeconds: 600  # Terminate job if it doesn't start within 10 minutes
  jobTemplate:
    spec:
      template:
        metadata:
          labels:
            app: incident-automation
        spec:
          containers:
          - name: incident-container
            image: <image-url>
            command: ["sh", "-c"]
            args:
            - >
              sleep 30; # Sleep to ensure only one job is running at a time (if another starts)
              /bin/sh -c 'for ((i=1;i<=10;i++)); do echo "Running $i"; sleep 60; done'
          restartPolicy: OnFailure
Leader Election with Kubernetes StatefulSet: To implement leader election, we can use a StatefulSet to ensure each node has a unique identifier and can compete for the leadership.
Step 5: ConcurrencyPolicy and StartingDeadlineSeconds
ConcurrencyPolicy: Forbid ensures that only one instance of the job runs at any time.
startingDeadlineSeconds: Ensures that if a job doesn't start within 10 minutes, it is terminated to avoid stale jobs.

#### Closing Summary
+ Common Mistakes/Follow-Ups

To summarize, we have designed our system using Kubernetes CronJobs with concurrencyPolicy: Forbid and proper lease management. This ensures that only one instance of the job runs at any time, even across node failures and multiple maintenance windows. Key considerations include:

Leader Election: Ensuring a single node can take over in case of node failures.
Leases: Managing node state to handle temporary outages effectively.

#### Common mistakes
include not setting concurrencyPolicy correctly or failing to implement proper lease management, which could lead to duplicate runs or missed jobs. Follow-ups might involve discussing how this solution scales with increasing fleet size and the trade-offs involved in different concurrency policies.

**Code:**

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: incident-automation
spec:
  schedule: "0 3 * * *"  # Run at 3 AM every day
  concurrencyPolicy: Forbid
  failedJobsHistoryLimit: 1
  successfulJobsHistoryLimit: 3
  startingDeadlineSeconds: 600  # Terminate job if it doesn't start within 10 minutes
  jobTemplate:
    spec:
      template:
        metadata:
          labels:
            app: incident-automation
        spec:
          containers:
          - name: incident-container
            image: <image-url>
            command: ["sh", "-c"]
            args:
            - >
              sleep 30; # Sleep to ensure only one job is running at a time (if another starts)
              /bin/sh -c 'for ((i=1;i<=10;i++)); do echo "Running $i"; sleep 60; done'
          restartPolicy: OnFailure
```

