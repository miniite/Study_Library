---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd design a distributed lock or lease so exactly one worker in your fleet runs a given scheduled retraining job, tolerating the holder GC-pausing. Why is a lease with a fencing token safer than a plain lock here?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Locks and Fencing](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/Distributed_Locks_and_Fencing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a distributed system where exactly one worker in a fleet runs a given scheduled retraining job, we need to ensure mutual exclusion and handle scenarios like garbage collection (GC) pauses gracefully. A common approach is to use a lease with a fencing token, which provides stronger guarantees than a plain lock.

In this context, a lease involves acquiring a time-limited lock that can be renewed periodically. If the worker holding the lease crashes or GCs, another worker can acquire the lease. The fencing token ensures that only one job instance runs at any given time by providing unique identifiers and ensuring consistency across distributed components.

Let's delve into the first-principles of this design:

Mutual Exclusion: We need to ensure that only one worker can execute the retraining job at a time.
Garbage Collection Handling: The system should handle GC pauses, where holding the lock might be temporarily lost or delayed.
Fencing Token Safety: This token ensures that only a single instance of the job runs by preventing stale locks.

#### Triaging the Question

The question is asking about designing a mechanism to ensure exactly one worker can execute a scheduled retraining job in a distributed system, while also handling GC pauses. The core concepts involved are:

Mutual Exclusion: Ensuring no two workers run the same task simultaneously.
Handling GC Pauses: Managing situations where the current lock holder might not be able to maintain its lock due to GC operations.

#### First Principles Solution
Lease Mechanism: Use a lease mechanism with periodic renewal to ensure that the worker holding the lease can maintain it even if it encounters temporary issues like GC pauses.
Fencing Tokens: Implement a fencing token to prevent multiple instances of the job from running concurrently by ensuring only one worker has active ownership.
Detailed Solution

Lease Mechanism:

Each worker node attempts to acquire a lease for the retraining job.
The lease is initially acquired using an atomic operation, such as GET_LOCK in MySQL or similar distributed lock mechanisms like ZooKeeper's create.
Periodically (every few seconds), the worker renew its lease by issuing another request. If it cannot renew the lease within a timeout period, it releases the lease and exits.

Fencing Token:

When a node acquires the lease, it generates a unique token.
This token is stored in a shared, reliable storage (e.g., Redis or a database) to ensure that other nodes can verify its validity.
Other nodes periodically check if they have an older or invalid fencing token and will not proceed with the job.

Handling GC Pauses:

If a worker node experiences a GC pause, it might fail to renew its lease in time.
Upon renewal failure, another worker can acquire the lease and continue the retraining job, ensuring that only one instance runs at any given moment.

#### Implementation
Details

Worker Node Code:

PYTHON
import redis
from threading import Thread, Lock

class RetrainerWorker:
    def __init__(self, job_id):
        self.job_id = job_id
        self.lock_client = redis.Redis()
        self.lock_key = f"retraining_lock:{job_id}"
        self.token = None

    def acquire_lock(self, lease_time=60):
        lock_id = self.lock_client.eval(
            "return redis.call('SET', KEYS[1], ARGV[1], 'EX', ARGV[2])",
            1,
            self.lock_key,
            f"{self.job_id}-{self.token}",
            lease_time
        )
        if lock_id == 0:
            raise Exception("Failed to acquire lock")

    def renew_lock(self, lease_time=60):
        self.lock_client.eval(
            "return redis.call('EXPIRE', KEYS[1], ARGV[2])",
            1,
            self.lock_key,
            lease_time
        )

    def run_job(self):
        while True:
            try:
                # Acquire the lock with a fencing token
                self.acquire_lock()
                # Perform retraining job
                self.retrain_model()
                # Renew the lock to avoid expiration
                self.renew_lock()
            except Exception as e:
                print(f"Exception: {e}")
            finally:
                time.sleep(10)  # Sleep before retrying

if __name__ == "__main__":
    worker = RetrainerWorker(job_id="model_123")
    Thread(target=worker.run_job).start()

Fencing Token Management:

Workers generate and store unique tokens in shared storage.
When attempting to acquire a lock, they check if the token is valid and newer than any existing tokens.

#### Closing Summary

By implementing a lease mechanism with periodic renewal and using fencing tokens, we ensure mutual exclusion and handle GC pauses effectively. The lease approach allows us to maintain the lock even during temporary failures, while the fencing token ensures that only one worker can execute the retraining job at a time, preventing race conditions and stale locks.

#### Common Mistakes
and Follow-Ups
Not Handling Renewals Properly: Failing to renew leases in a timely manner can lead to lease expiration and concurrent execution.
Incorrect Token Management: Improper validation of tokens might result in multiple instances running simultaneously.
Ignoring GC Pauses: Not accounting for temporary losses of lock ownership during GC operations can cause job duplication or failures.

These considerations are crucial for building robust, fault-tolerant distributed systems.

