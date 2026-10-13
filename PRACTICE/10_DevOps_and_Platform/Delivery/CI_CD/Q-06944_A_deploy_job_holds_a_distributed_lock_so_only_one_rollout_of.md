---
type: "practice"
difficulty: "Deep"
---

# A deploy job holds a distributed lock so only one rollout of a service happens at a time, but a crashed job left the lock held and blocked all deploys; how do you make deploy locks self-healing?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Idempotent Deploy Jobs](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue where a crashed deploy job holds onto a distributed lock, blocking subsequent deploys, we need to implement self-healing mechanisms that ensure the lock is released if the job crashes or times out. This involves several steps from first principles:

Understand the Problem: A distributed lock ensures atomicity in deployments by preventing concurrent rollouts of services. However, a crashed deploy job can hold onto the lock indefinitely, causing all subsequent deploys to stall.
Identify Key Components: The key components are the locking mechanism (e.g., using etcd or Consul), the deployment service that acquires and releases locks, and the monitoring system that detects stuck jobs.

We'll start by breaking down these components and reasoning through a solution step-by-step.

**Code:**

```python
# Example Python code snippet to illustrate lock acquisition and release

import time
from distributed_lock import DistributedLock

def deploy_service(service_name):
    try:
        with DistributedLock(f"/deploy/{service_name}") as lock:
            # Perform deployment steps here
            print(f"Deploying {service_name}...")
            time.sleep(2)  # Simulate deployment duration
            print("Deployment complete.")
    except LockAcquisitionException as e:
        print(f"Failed to acquire lock: {e}")
```

**Example input / output:**

```text
# Input for the deployment process with a stuck job scenario

Input: A deploy request to service "app-service"
Output: Deployment of app-service fails due to lock held by crashed previous job.

Input: Attempting redeploy after initial failure
Output: Redeploy request is blocked until original lock times out or is manually released.
```

**Data:**

```yaml
# Example configuration for distributed lock service
lock_service:
  backend: etcd
  endpoint: http://localhost:2379
deploy_locks:
  - path: /deploy/*
    timeout: 600  # 10 minutes
```

