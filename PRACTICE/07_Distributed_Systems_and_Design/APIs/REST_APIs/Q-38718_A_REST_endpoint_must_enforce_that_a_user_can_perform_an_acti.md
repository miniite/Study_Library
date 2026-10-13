---
type: "practice"
difficulty: "Expert"
---

# A REST endpoint must enforce that a user can perform an action at most N times per day across a distributed cluster, counting accurately even under concurrent requests and node failures. How do you build this quota?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ API Rate Limiting](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/API_Rate_Limiting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce a daily usage limit on a REST endpoint in a distributed system, we need to ensure that each user can perform an action at most N times per day, even under concurrent requests and node failures. This problem requires a combination of state storage, synchronization mechanisms, and a robust design for handling concurrency.

#### Triage
Understanding the Question

The core question is about implementing rate limiting with a daily quota in a distributed environment. The interviewer is probing our understanding of distributed systems, state management, synchronization, and how to handle concurrent requests effectively. We need to assume that there are multiple nodes in the cluster and that user actions can be performed concurrently.

#### First-Principles Solution
State Storage: Store the quota usage for each user.
Synchronization Mechanisms: Ensure atomic updates to the quota state, even under concurrent requests.
Concurrency Handling: Design a mechanism to handle simultaneous requests from different nodes without race conditions.
Step 1: State Storage

We can store the quota state in a distributed key-value store like Redis or Cassandra. This allows us to have a single source of truth for each user's daily quota.

Yes
No
User Action
Check Quota Usage
Is Quota Exceeded?
Return Error
Decrement Quota
Perform Action
Save State
Step 2: Synchronization Mechanisms

To ensure atomic updates, we can use a distributed lock. Redis provides a SETNX command to achieve this.

Request
Yes
No
Yes
A
Get Lock on User Quota
Lock Acquired?
Check Quota Usage
Is Quota Exceeded?
Decrement Quota
Perform Action
Release Lock
Return Error
Step 3: Concurrency Handling

To handle concurrent requests, we can use a token bucket or leaky bucket algorithm. This approach ensures that the quota is not exceeded even if multiple nodes try to decrement the quota simultaneously.

Request
Yes
No
Yes
A
Get Token from Bucket
Token Available?
Check Quota Usage
Is Quota Exceeded?
Decrement Quota
Perform Action
Return Success
Return Error

#### Closing Summary
and Common Mistakes

To summarize, we need a combination of distributed state storage and synchronization mechanisms to enforce the quota. The key is ensuring atomic updates and handling concurrent requests properly.

#### Common mistakes
include:

Failing to use a distributed lock, leading to race conditions.
Not using a token bucket or leaky bucket algorithm, causing quotas to be exceeded under high concurrency.
Storing quota states in a non-distributed manner, which can lead to inconsistencies across nodes.

**Comparison:**

```
User ID	Daily Quota
user_001	5
user_002	3
```

**Code:**

```python
import redis

def check_and_perform_action(user_id, r_client):
    # Check if the lock is available (using SETNX)
    with r_client.lock(f"quota_lock_{user_id}", timeout=60) as lock:
        if not lock.acquired:
            return "Error: Quota exceeded"

        # Get the current quota usage
        quota_usage = int(r_client.get(user_id))

        # Check if the quota is already exhausted
        if quota_usage >= N_MAX:
            return "Error: Quota exceeded"

        # Perform the action and decrement the quota
        r_client.decrby(user_id, 1)
        # Perform the actual action here

        return "Action performed successfully"
```

**Example input / output:**

```text
Input: `POST /action?userId=user_001`
Output:
- If the quota is available: "Action performed successfully"
- If the quota is exhausted: "Error: Quota exceeded"
```

**Data:**

```yaml
user_quotas:
  user_001: 5
  user_002: 3
```

**Flow:**

1. Yes
2. No
3. User Action
4. Check Quota Usage
5. Is Quota Exceeded?
6. Return Error
7. Decrement Quota
8. Perform Action
9. Save State

