---
type: "practice"
difficulty: "Expert"
---

# Design a real-time-vs-batch reconciliation architecture for user-facing counters (views/likes) where the fast approximate number and the accurate nightly number disagree — how do you reconcile without users noticing jumps?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Real-time Counters](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Real-time_Counters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In this design, we will implement a high-level architecture that reconciles real-time and batch counter values (like views/likes) without causing noticeable discrepancies or jumps in user experience. The core idea is to use an approximate counter for fast updates and then perform a precise reconciliation during nightly batch processing.

#### Triaging the Question

The question revolves around designing an architecture where we maintain both real-time and accurate counts of user-facing counters (such as views/likes). We need to ensure that these two values eventually converge without causing noticeable discrepancies in the user interface. The key aspects being probed are:

Real-time vs Batch Processing: How to manage fast, approximate updates versus slower, precise batch reconciliations.
User Experience: Ensuring minimal impact on users when discrepancies arise.
Scalability and Performance: Handling high volumes of updates efficiently.

#### Assumptions
The system has a real-time counter that updates quickly but with some approximation errors.
We perform a nightly reconciliation to ensure the values are accurate, using a more precise algorithm or database operation.
There is a mechanism to detect discrepancies between the real-time and batch counts.
Users should not experience any noticeable jumps or inconsistencies in their view of the counters.

#### Solution
Step-by-Step Solution

Real-Time Counter

Use a distributed system like Redis for storing approximate values. This allows quick updates from various sources (e.g., HTTP requests, background tasks).
Implement an incremental counting strategy where each update adds or subtracts by 1.

Batch Reconciliation Mechanism

Set up a nightly batch process that triggers reconciliation.
The reconciliation logic should be robust and able to handle large volumes of updates efficiently.

#### Cons
ider using a database transactional mechanism for precise updates if needed.

Discrepancy Detection

Compare the real-time counter value with the latest accurate value (stored in a more reliable storage like PostgreSQL).
If significant discrepancies are detected, trigger an immediate reconciliation process to correct the values.

Immediate Reconciliation Process

Use a background task or queue system (e.g., RabbitMQ) to handle immediate reconciliations.
Ensure these tasks do not block other real-time operations by using asynchronous processing.

User Experience Mitigation

Implement logic that temporarily hides discrepancies during reconciliation. For example, use caching mechanisms to show the last known accurate value until the system is fully updated.
Optionally, provide a smooth transition by blending old and new values over time (e.g., linear interpolation).
Detailed Steps

Distributed Real-Time Counter

Redis
Redis
Real-time Update
Incremental Count
Distributed Storage

Nightly Batch Reconciliation

Cron Job
PostgreSQL
Nightly Trigger
Batch Process
Precise Update

Discrepancy Detection and Immediate Reconciliation

Compare
Discrepancy Detected
Process
Real-time Value
Accurate Value (DB)
Immediate Reconcile TaskQueue
Task Handling
Correct Values, Update UI

#### Closing Summary

By implementing a hybrid architecture that leverages real-time approximate counters and periodic batch reconciliations, we can ensure the system provides accurate counts while minimizing user disruption. This approach balances performance with accuracy and ensures that discrepancies are handled gracefully.

#### Common Mistakes/Follow-ups
Ignoring User Experience: Failing to account for how users perceive these discrepancies.
Over-engineering Immediate Reconciliation: Implementing complex logic without considering simpler, effective solutions like temporary caching.
Neglecting Performance Considerations: Not optimizing batch reconciliation processes for scalability.

**Comparison:**

```
Step	Action
1	Initialize Redis connection for real-time counter.
2	Set up PostgreSQL database connection for batch processing.
3	Define discrepancy detection threshold and reconciliation logic.
4	Trigger immediate reconciliation if discrepancies exceed threshold.
```

**Code:**

```python
# Example of a simple discrepancy detection function
def detect_discrepancy(real_time_value, accurate_value):
    threshold = 100  # Define the acceptable difference threshold
    if abs(real_time_value - accurate_value) > threshold:
        return True
    return False

# Example of an immediate reconciliation task queue
import asyncio
from redis import Redis
from psycopg2 import connect

redis_client = Redis()
db_connection = connect(db=0)

async def reconcile_values(real_time_key, accurate_value):
    real_time_value = int(redis_client.get(real_time_key))
    if detect_discrepancy(real_time_value, accurate_value):
        with db_connection.cursor() as cursor:
            sql = "UPDATE counters SET value = %s WHERE key = %s"
            cursor.execute(sql, (accurate_value, real_time_key))
        await redis_client.set(real_time_key, accurate_value)
```

**Example input / output:**

```text
# Input
real_time_key = 'user_likes'
accurate_value = 123456700

# Output
if detect_discrepancy(123456789, accurate_value):
    # Immediate reconciliation task triggered
    reconcile_values(real_time_key, accurate_value)
```

**Data:**

```json
{
  "real_time_counter": {
    "key": "user_likes",
    "value": 123456789
  },
  "accurate_counter": {
    "key": "user_likes",
    "value": 123456700
  }
}
```

**Flow:**

1. Yes
2. No
3. Real-time Update
4. Detect Discrepancy
5. Immediate Reconciliation
6. Update UI, Database
7. Nightly Batch Process

