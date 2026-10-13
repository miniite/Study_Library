---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd design a distributed lock or lease backed by etcd/ZooKeeper so exactly one worker runs a critical database maintenance job, why you'd prefer a lease with a fencing token over a plain lock when the holder can GC-pause, and where the fence must be checked.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Distributed Locks and Fencing](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/Distributed_Locks_and_Fencing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a distributed system that requires exactly one worker to run a critical database maintenance job, using a distributed lock or lease is crucial. In this scenario, etcd and ZooKeeper are common choices due to their robustness and wide adoption. Specifically, for the maintenance task, I would opt for a lease with a fencing token over a plain lock when the holder can undergo GC-pause. This approach ensures that only one worker acquires the lease and prevents potential deadlocks or resource leaks. The fence must be checked at critical points to ensure consistency.

#### Triage
What is Being Probed

The question aims to evaluate your understanding of distributed systems, specifically how to manage concurrent access to shared resources in a fault-tolerant manner. It also tests your ability to choose between different mechanisms (locks vs leases) and the importance of fencing tokens. The core concepts include:

Understanding Locks and Leases: Differences between these two mechanisms.
Lease with Fencing Token: How it ensures safety in distributed systems.
GC-Pause Considerations: Why GC-pause is a critical factor when choosing between locks and leases.

#### First-Principles Solution

To design this system, we'll use etcd (or ZooKeeper) to implement the lease mechanism with fencing tokens. The key steps are:

Initialization: Workers register themselves with etcd or ZooKeeper.
Lease Acquisition: A worker acquires a lease from etcd/ZooKeeper for the maintenance job.
Fencing Token: Upon acquiring the lease, the worker obtains a unique fencing token to ensure only one instance of the task runs at a time.
Task Execution: The worker executes the database maintenance job.
Lease Renewal and Fencing Token Management: The worker continuously renews its lease and manages the fencing token to avoid losing the lock due to GC-pause or other issues.
Detailed Steps
Step 1: Initialization

Each worker registers itself with etcd/ZooKeeper by creating a unique node for each instance. This step ensures that every worker has a unique identifier, which is crucial for managing leases and ensuring no two instances can conflict.

BASH
# Example in etcd
etcdctl set /workers/node-1234 "worker-uuid-123"
Step 2: Lease Acquisition

A worker acquires a lease from etcd/ZooKeeper. The lease is granted for a specific duration, and the worker must renew it periodically to keep the lock.

PYTHON
# Example in Python using etcd3 library
from etcd3 import Client

client = Client(host='localhost', port=2379)
lease = client.lease(60)  # Lease for 1 minute
key = '/lock/maintenance'
response = client.txn().compare(key_not_exists()).then_put(key, 'leased').commit()

if response:
    print("Lease acquired successfully.")
Step 3: Fencing Token Management

The worker generates a unique fencing token when it acquires the lease. This token is used to ensure that no other instance can hold the same lock during its lifecycle.

PYTHON
# Example in Python generating a fencing token
import uuid

fencing_token = str(uuid.uuid4())
print(f"Fencing Token: {fencing_token}")
Step 4: Task Execution

The worker proceeds with executing the critical database maintenance job while holding the lease and using the fencing token.

SQL
-- Example SQL for maintenance task
BEGIN TRANSACTION;
UPDATE table1 SET status = 'maintained' WHERE last_maintenance < now() - interval '2 hours';
COMMIT;
Step 5: Lease Renewal and Fencing Token Management

The worker must continuously renew its lease to prevent it from expiring. Additionally, the worker should periodically check for fencing tokens to ensure consistency.

PYTHON
# Example in Python renewing lease and checking fence token
while True:
    response = client.get(key)
    if not response.value == 'leased':
        print("Lease expired or invalidated.")
        break

    # Renew lease
    new_lease = client.lease(60)  # Extend the lease for another minute
    client.extend(response.header.mod_revision, lease_id=new_lease.id)

    # Check fence token (pseudo-code)
    if check_fence_token(fencing_token):
        print("Fence violated, retrying.")

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: Using a lease with a fencing token in etcd/ZooKeeper ensures that only one worker can perform the critical database maintenance job. This mechanism is robust against GC-pause and other issues, making it suitable for high-stakes tasks.

#### Common Mistakes
Ignoring lease renewal: A worker must renew its lease periodically to avoid expiration.
Missing fencing token checks: Ensuring that a worker does not encounter stale or invalid fence tokens is crucial.
Inadequate error handling: Properly managing lease expiration and fence token violations can prevent data inconsistencies.

#### Follow-Ups
Question: How would you handle scenarios where multiple workers might attempt to acquire the lease simultaneously?
Question: What are some best practices for managing long-running leases in a distributed system?

**Comparison:**

| Worker | Node ID   | State         |
|--------|-----------|---------------|
| W1     | 1234      | Acquired Lease|
| W2     | 5678      | Idle          |
| W3     | 9012      | Lease Expired |

**Code:**

```python
# Example code using etcd3 library to acquire and renew a lease with fencing token management
from etcd3 import Client

client = Client(host='localhost', port=2379)
lease = client.lease(60)  # Lease for 1 minute
key = '/lock/maintenance'

response = client.txn().compare(key_not_exists()).then_put(key, 'leased').commit()
if response:
    print("Lease acquired successfully.")

fencing_token = str(uuid.uuid4())

while True:
    response = client.get(key)
    if not response.value == 'leased':
        print("Lease expired or invalidated.")
        break

    # Renew lease
    new_lease = client.lease(60)  # Extend the lease for another minute
    client.extend(response.header.mod_revision, lease_id=new_lease.id)

    # Check fence token (pseudo-code)
    if check_fence_token(fencing_token):
        print("Fence violated, retrying.")
```

**Example input / output:**

```text
Input:
- Worker registers with etcd/ZooKeeper at /workers/node-1234.
- Acquires lease for maintenance task.

Output:
- Lease acquired successfully.
- Fencing token generated: abcdef1234567890
- Leases renewed periodically to prevent expiration.
```

**Data:**

```json
{
  "key": "/lock/maintenance",
  "value": "leased",
  "lease_id": "1234567890"
}
```

**Flow:**

1. Yes
2. Fence Violated?
3. Worker Registers
4. Acquire Lease?
5. Generate Fencing Token
6. Renew Lease Periodically
7. Check Fence Token
8. Retry Task
9. Execute Maintenance Task

