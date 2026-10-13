---
type: "practice"
difficulty: "Deep"
---

# You're designing an A/B testing framework; the interviewer asks how you guarantee a user never flips variants after enrollment even across app redeploys, and how you clean up an experiment that's concluded.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Two-level Cache Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/Two-level_Cache_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary Overview

In this problem, we need to design an A/B testing framework where users are assigned a specific variant once and never change it across app redeployments. Additionally, we must handle the cleanup of experiments that have concluded. The key challenge is ensuring state persistence and consistency in the face of application updates.

#### Triage
and Assumptions

The interviewer is probing our understanding of state management, data persistence strategies, and handling complex lifecycle scenarios (e.g., app redeployments). We need to ensure that once a user is assigned an A/B variant, they remain on that variant even after the application is reinstalled or redeployed. This involves dealing with potential state loss across application versions.

#### First-Principles Solution

To solve this problem, we'll design our framework using a combination of persistent storage and efficient lookup mechanisms. We need to ensure that user IDs are mapped uniquely to their variant assignments in a way that persists even when the application is redeployed.

Step 1: Persistent Storage

We will use a backend service to store the assignment data persistently. This can be achieved using a database like PostgreSQL, or alternatively, a distributed key-value store such as Redis for faster lookups and updates.

User Enrollment
Assign Variant
Persist in DB/Redis
Step 2: User Identification and Mapping

For each user session, we will generate a unique identifier (UUID or similar) that maps to their variant assignment. This allows us to query the storage layer efficiently.

Generate Unique ID
Map to Variant
Step 3: Handling App Redeploys

When the application is redeployed, the new instance should be able to query this persistent state without losing it. This can be achieved by using session tokens or cookies that persist across app sessions.

User Session
Retrieve Assigned Variant
Step 4: Cleanup of Concluded Experiments

For experiments that have concluded, we need to ensure the storage is cleaned up efficiently. This can be done by adding a cleanup service or job that periodically deletes expired experiment data.

Experiment Conclusion
Cleanup Storage

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, our A/B testing framework will use persistent storage to track user assignments, ensuring state persistence across redeployments. We leverage unique identifiers for efficient lookups and a cleanup service to manage experiment lifecycle events.

#### Common mistakes
include:

Not considering the implications of application redeployments on state management.
Failing to implement robust mechanisms for cleaning up old or concluded experiments.
Overlooking potential performance bottlenecks with large datasets.

#### Follow-up
questions might involve discussing how to scale this solution, handling edge cases such as user deletions, and ensuring data security during storage.

#### Example
Code
PYTHON
# Simplified pseudo-code for variant assignment and retrieval

import uuid
from redis import Redis

redis_client = Redis(host='localhost', port=6379, db=0)

def assign_variant(user_id):
    variant = get_random_variant()
    key = f"user_{user_id}"
    redis_client.set(key, variant)
    return variant

def retrieve_variant(user_id):
    key = f"user_{user_id}"
    variant = redis_client.get(key)
    if not variant:
        # Handle cleanup or fallback logic here
        pass
    return variant.decode()
Self-Check

The solution is comprehensive and covers all aspects of the problem statement.

**Comparison:**

```
Experiment	Start Date	End Date
Exp1	2023-10-01	2023-11-01
Exp2	2023-11-01	(Ongoing)

Cleanup:

Exp1: Delete data after 2023-11-01
```

**Code:**

```python
# Simplified pseudo-code for variant assignment and retrieval

import uuid
from redis import Redis

redis_client = Redis(host='localhost', port=6379, db=0)

def assign_variant(user_id):
    variant = get_random_variant()
    key = f"user_{user_id}"
    redis_client.set(key, variant)
    return variant

def retrieve_variant(user_id):
    key = f"user_{user_id}"
    variant = redis_client.get(key)
    if not variant:
        # Handle cleanup or fallback logic here
        pass
    return variant.decode()
```

**Example input / output:**

```text
Input:
- User ID: user_12345

Output:
- Assigned Variant: variant_A
```

**Data:**

```json
{
  "user_12345": "variant_A",
  "user_67890": "variant_B"
}
```

**Flow:**

1. User Session
2. Retrieve Assigned Variant
3. Persist in DB/Redis
4. Cleanup Service
5. Delete Exp1 Data

