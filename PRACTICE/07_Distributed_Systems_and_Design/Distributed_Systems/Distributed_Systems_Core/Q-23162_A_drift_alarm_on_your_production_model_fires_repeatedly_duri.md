---
type: "practice"
difficulty: "Intermediate"
---

# A drift alarm on your production model fires repeatedly during an incident and each firing kicks off a duplicate expensive retrain contending for GPUs. Explain the idempotent, exactly-once retraining trigger with a dedup key and lease you'd design.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Idempotent Ingestion](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of redundant retrainings during an incident by designing an idempotent, exactly-once retraining mechanism using a deduplication key and lease system, we need to ensure that our system can handle drift alarms reliably without causing unnecessary resource contention. This involves understanding the problem, defining the necessary components, and reasoning through their implementation.

#### Triage

The question is asking for an idempotent solution to avoid redundant retraining triggered by a drift alarm during an incident. The key aspects include:

Ensuring that each trigger of the drift alarm leads to exactly one retraining.
Avoiding resource contention by managing concurrent requests.
Designing a mechanism that can handle the load and scale during incidents.

#### Assumptions
We are using a distributed system with multiple instances or services triggering retrainings.
The drift alarm fires repeatedly, leading to duplicate expensive retrain contending for GPUs.
Each instance of our service should be able to determine whether it should proceed with a retraining without unnecessary duplication.

#### First Principles Solution
Deduplication Key: Each trigger of the drift alarm will generate a unique key (e.g., a UUID or timestamp) that acts as an identifier for this specific event.
Lease System: We use a lease system to ensure that only one instance can proceed with retraining at any given time.
Step-by-Step Solution
Generate Deduplication Key: When the drift alarm fires, generate a unique key (e.g., UUID) and store it in a distributed hash table (DHT).
Lease Management:
Each service instance will attempt to acquire a lease by querying the DHT with the generated key.
If the lease is acquired (indicating that this instance can proceed), perform the retraining.
After completing the retraining, release the lease and remove the key from the DHT.

#### Implementation
Details
Distributed Hash Table (DHT)
Choose a reliable distributed storage solution like Redis or a custom implementation based on etcd or Cassandra.
Lease Protocol:
Use a time-based token system where each instance requests a lease and gets a time window within which it can proceed with the retraining.
Concurrency Control:
Implement a mechanism to handle race conditions, such as exponential backoff if multiple instances simultaneously attempt to acquire the same lease.

#### Example
Flow
Drift alarm fires → Generate unique key K.
Service instance 1 queries DHT for key K and finds it not present (indicating no active lease).
Service instance 1 acquires lease by setting a time-out value on K in the DHT.
Service instance 1 performs retraining.
After completion, service instance 1 releases the lease and removes K from the DHT.

#### Closing Summary

By implementing an idempotent retraining mechanism with a deduplication key and lease system, we ensure that each drift alarm trigger results in exactly one retraining, avoiding redundant GPU contention during incidents.

#### Common Mistakes/Follow-Ups
Common Mistake: Failing to implement proper concurrency control can lead to race conditions where multiple instances might acquire the same lease.

#### Follow-Up
Questions:
How do you handle cases where the DHT fails or a lease expires?
Can you scale this solution horizontally to manage more concurrent retrainings?

**Comparison:**

```
Deduplication Key	Lease Expiry
98f5262b-0aeb-40ff-a18e-d335df9a79c4	2023-10-15T12:00:00Z
```

**Code:**

```python
import uuid
from redis import Redis

# Initialize Redis client
redis_client = Redis(host='localhost', port=6379, db=0)

def generate_dedup_key():
    return str(uuid.uuid4())

def acquire_lease(dedup_key):
    # Set a lease for the given key with an expiry time
    result = redis_client.set(dedup_key, "leased", ex=60)  # Expiry in seconds
    if result:
        print(f"Lease acquired for {dedup_key}")
        return True
    else:
        print("Failed to acquire lease")
        return False

def release_lease(dedup_key):
    redis_client.delete(dedup_key)
    print(f"Released lease for {dedup_key}")

# Example usage
dedup_key = generate_dedup_key()
if acquire_lease(dedup_key):
    # Perform retraining
    perform_retraining()
    release_lease(dedup_key)
else:
    print("Another instance is performing the retraining")
```

**Example input / output:**

```text
Input:
- Drift alarm fires at 2023-10-15T11:59:58Z

Output:
- Service instance 1 generates deduplication key '98f5262b-0aeb-40ff-a18e-d335df9a79c4'
- Instance 1 attempts to acquire lease in Redis, succeeds
- Instance 1 performs retraining and releases the lease at 2023-10-15T12:00:01Z

Result:
- No redundant retrainings due to proper use of deduplication key and lease system.
```

**Data:**

```json
{
  "deduplication_key": "98f5262b-0aeb-40ff-a18e-d335df9a79c4",
  "lease_expiry": "2023-10-15T12:00:00Z"
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[Drift Alarm Fires] --> B[Generate Deduplication Key]
4. B --> C(Acquire Lease)
5. C --> D[Acquire Success]::yes
6. D --> E[Perform Retraining]
7. E --> F[Release Lease & Remove Key from DHT]
8. C --> G[Acquire Failure]::no
9. G --> H[Another Instance is Performing Retraining]

