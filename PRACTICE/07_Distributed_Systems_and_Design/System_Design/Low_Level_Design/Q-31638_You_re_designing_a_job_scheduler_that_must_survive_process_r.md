---
type: "practice"
difficulty: "Expert"
---

# You're designing a job scheduler that must survive process restarts and not run a missed cron twice; how do you model persistence, leader election, and at-most-once vs at-least-once execution?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Two-level Cache Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/Two-level_Cache_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a robust job scheduler that survives process restarts without running missed jobs twice, we need to ensure three key aspects: persistence, leader election, and handling message delivery semantics (at-most-once vs at-least-once). Persistence ensures that the state of the system is saved across restarts. Leader election guarantees only one instance runs critical operations. At-most-once delivery prevents duplicate execution, while at-least-once ensures all jobs are processed.

First, let's break down each requirement and then reason through how to implement them from first principles.

**Comparison:**

```
Message Delivery Semantics	At-Most-Once vs At-Least-Once
At-Most-Once	Guarantees no duplicate execution but might miss some messages. Ideal for non-critical tasks.
At-Least-Once	Ensures all messages are delivered at least once, potentially leading to duplicates in critical systems like a job scheduler. Essential for ensuring task completion even if the system fails between processing.
```

**Code:**

```python
# Example of a simplified persistence model using Redis for state storage
import redis

client = redis.Redis(host='localhost', port=6379, db=0)

def save_state(state):
    client.set('job_scheduler.state', state)

def load_state():
    return client.get('job_scheduler.state')

save_state({'last_processed': 12345})
```

**Example input / output:**

```text
Input:
- A list of jobs with IDs and cron expressions

Output:
- Ensured that no job is run twice for missed executions
- Only one instance processes critical tasks at any time
- Correct message delivery semantics are maintained
```

**Data:**

```json
{
  "jobs": [
    {"id": 1, "name": "daily_backup", "cron_expression": "* * * * *"},
    {"id": 2, "name": "hourly_sync", "cron_expression": "0 * * * *"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Job Scheduling Request
4. Is Leader?
5. Process Job
6. Forward to Leader Node
7. Save State in Persistent Store
8. Leader Processes Job
9. Notify Worker NodesCompletion

