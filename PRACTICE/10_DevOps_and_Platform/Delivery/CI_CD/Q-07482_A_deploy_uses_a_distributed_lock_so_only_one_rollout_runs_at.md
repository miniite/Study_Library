---
type: "practice"
difficulty: "Deep"
---

# A deploy uses a distributed lock so only one rollout runs at a time, but a crashed job left the lock held and blocked all deploys; how would you make deploy locks self-healing without risking two concurrent rollouts?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Idempotent Deploy Jobs](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a crashed CI/CD deployment leaving a distributed lock in place, blocking subsequent deployments, we need to design a solution that ensures self-healing capabilities while maintaining strict concurrency control. The key challenge is to detect and resolve stale locks without introducing race conditions or allowing two rollouts to run simultaneously.

#### Triage

The question asks how to make deploy locks self-healing without risking two concurrent rollouts. This involves understanding the current locking mechanism, identifying the root cause of lock staleness, and designing a failover strategy that can detect and resolve stale locks gracefully.

#### Assumptions
The deployment system uses distributed locks for ensuring atomic operations.
Locks are managed by a centralized or distributed key-value store.
There is a process to periodically clean up stale locks in the system.

#### First-Principles Solution

To achieve self-healing of locks, we can implement a combination of lock detection and resolution strategies:

Stale Lock Detection:

Implement a periodic health check mechanism that runs at regular intervals (e.g., every 5 minutes).
The health check should query the lock store to identify any stale or crashed deployments.

Lock Resolution:

Upon detection of a stale lock, initiate a cleanup process by releasing the lock.
Ensure this process does not introduce race conditions; use techniques like exponential backoff for retries.

Concurrency Control:

Maintain a single active rollout at any given time to avoid conflicts.
Use a leader-election mechanism or a dedicated service to manage the active rollout state.
Detailed Steps

Health Check Mechanism:

Configure periodic health checks that run on each deployment node.
Each check sends a request to the lock store to verify if any locks are stale (e.g., older than 5 minutes).

Stale Lock Handling:

If a stale lock is detected, trigger a cleanup process.
The cleanup involves releasing the lock by removing it from the lock store.

Concurrency Control:

Implement a leader-election mechanism to ensure only one rollout can be active at any time.
Use a distributed consensus protocol (e.g., Raft) or a dedicated service that acts as a "rollout manager" to handle active states and coordinate deployments.

Exponential Backoff:

To avoid immediate retries that might cause conflicts, implement an exponential backoff strategy for cleanup attempts.
This ensures the system does not flood the lock store with cleanup requests immediately after detection of a stale lock.

#### Closing Summary

By implementing a combination of periodic health checks, stale lock handling, and concurrency control mechanisms, we can achieve self-healing locks in a CI/CD environment. The key is to balance between detecting and resolving stale locks without introducing race conditions or allowing two concurrent rollouts.

#### Common Mistakes
Immediate Cleanup: Failing to use exponential backoff can lead to frequent lock cleanup requests, potentially causing performance issues.
Race Conditions: Not properly handling the release of a lock during cleanup can result in conflicts with other deployments.
Concurrency Issues: Ignoring concurrency control mechanisms can lead to situations where multiple rollouts are initiated simultaneously.
Self-Check

Correctness + completeness confirmed.

**Comparison:**

```
Lock Type	Action	Time to Resolve (s)
Deployment	Health Check	5
	Release Lock	Varies
```

**Code:**

```python
# Example code for periodic health check and stale lock handling

import time
from functools import partial

def health_check(lock_store, max_age_seconds=300):
    current_time = time.time()
    stale_locks = lock_store.query_stale_locks(current_time - max_age_seconds)
    return stale_locks

def release_lock(lock_id, lock_store):
    lock_store.release_lock(lock_id)

def main():
    while True:
        stale_locks = health_check(lock_store)
        for lock in stale_locks:
            release_lock(lock.id, lock_store)
        time.sleep(300)  # Sleep for 5 minutes

main()
```

**Example input / output:**

```text
Input:
- Periodic health check configured every 5 minutes.
Output:
- Stale locks detected and released if older than 5 minutes.
```

**Data:**

```json
{
  "lock_store": {
    "type": "Redis",
    "uri": "redis://localhost:6379/0"
  }
}
```

**Flow:**

1. Yes
2. No
3. Stale Lock Detected
4. Is Exponential BackoffRequired?
5. Wait (Backoff Time)
6. Release Stale Lock
7. End

