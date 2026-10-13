---
type: "practice"
difficulty: "Deep"
---

# How would you implement a Python distributed lock backed by DynamoDB or Redis so that only one instance of a scheduled cleanup job runs across your fleet, and what failure modes (lost lock, clock skew) do you plan for?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Cache Invalidation Strategies](<../../../../TOOLS/Python/Python_Language/Cache_Invalidation_Strategies/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement a Python-based distributed lock using either DynamoDB or Redis, ensuring only one instance of a scheduled cleanup job runs across your fleet, we need to ensure atomicity and strong consistency in our locking mechanism. This involves creating a unique lock key that can be checked and acquired atomically.

TL;DR
Choose the Locking Backend: Use either DynamoDB or Redis.
Create a Unique Key: Generate a unique identifier for each cleanup job instance.
Acquire the Lock: Implement atomic operations to check and acquire the lock.
Release the Lock: Ensure the lock is released upon completion of the cleanup job, handling edge cases like process termination.

#### Triage

The question probes your understanding of distributed systems, specifically around managing concurrent access in a fleet of instances. Key assumptions include:

The system must handle multiple instances running on different servers.
Clock skew and network issues may cause temporary loss of lock ownership.
We need to manage cleanup jobs with atomic operations.

#### Solution
Step 1: Choose the Locking Backend

For this example, I’ll use Redis for its simplicity and wide availability. DynamoDB could be used but requires more setup due to its document-based nature and eventual consistency model.

PYTHON
import redis
import time

def get_redis_client():
    return redis.Redis(host='localhost', port=6379, db=0)
Step 2: Create a Unique Key

For each cleanup job instance, generate a unique key. This can be based on the job ID and the current timestamp.

PYTHON
import uuid

def create_unique_key(job_id):
    return f"cleanup_lock:{job_id}:{time.time_ns()}"
Step 3: Acquire the Lock

Implement atomic operations to check and acquire the lock using Redis's SETNX command, which sets a key if it does not exist.

PYTHON
def acquire_lock(key, timeout=10):
    r = get_redis_client()
    while True:
        acquired = r.setnx(key, 'true')
        if acquired:
            return acquired
        time.sleep(timeout / 10)
Step 4: Release the Lock

Ensure the lock is released upon completion of the cleanup job. Use a try-finally block to handle exceptions and ensure the key is always removed.

PYTHON
def release_lock(key):
    r = get_redis_client()
    with r.lock(key, timeout=10) as lock:
        if not lock.is_acquired:
            return False
    # Perform cleanup job here
    try:
        pass  # Cleanup logic goes here
    finally:
        r.delete(key)

#### Closing Summary
and Common Mistakes/Follow-Ups

To ensure robustness, consider the following failure modes:

Lost Lock: If a process crashes before releasing the lock, another instance may acquire it. To mitigate this, implement periodic re-acquisition checks.

Clock Skew: Ensure clock synchronization across your fleet to avoid false positives in acquiring locks.

Network Issues: Implement exponential backoff and retries for network operations to handle transient issues.

Additional follow-ups might include:

How do you ensure lock expiration in case of process crashes?
Can you optimize the solution for high traffic scenarios?

#### Common Mistakes
Failing to use atomic operations, leading to race conditions.
Not handling edge cases like clock skew and network failures.
Ignoring cleanup processes that might leave locks unacquired.

