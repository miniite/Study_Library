---
type: "practice"
difficulty: "Expert"
---

# An endpoint's read-after-write consistency broke when you added a read replica: clients POST then immediately GET stale data. How do you diagnose and fix this while keeping reads on replicas?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Consistent Pagination](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Consistent_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: This issue arises due to the introduction of a read replica, which can be confusing because writes are still happening in the primary database but reads might not yet reflect those changes due to replication lag or cache issues. To diagnose and fix this, we need to ensure that reads on replicas are only served after they have caught up with the primary database.

Triaging the Issue

The problem here is a read-after-write inconsistency when using a read replica. The core issue lies in ensuring that client GET requests respect the write consistency of the primary database. This involves understanding how and where data replication happens, potential caching issues, and strategies to mitigate these problems.

#### Assumptions
and Context
Database Architecture: You have a single primary database where writes occur.
Read Replicas: These are secondary databases that replicate data from the primary with some delay (replication lag).
Client Behavior: Clients send POST requests to write data, followed by immediate GET requests to check for the new data.
Step-by-Step Solution

Identify the Cause:

Replication Lag: Replicas might not be in sync immediately after a write.
Cache Issues: In-memory caches (e.g., Redis) might serve stale data if they are not invalidated properly.

Diagnostic Steps:

Enable Logging: Add detailed logging to both the primary and replica databases to track when writes occur and when reads happen.
Check Replication Lag: Use tools provided by your database management system (DBMS) to monitor replication lag between the primary and replicas.
Invalidate Caches: Ensure that after a write, any relevant in-memory caches are invalidated.

Fixing the Issue:

Replication Configuration: Optimize replication settings to reduce lag or ensure near real-time updates.
Read-After-WRITE Mechanism: Implement a read-after-write mechanism where reads can only occur if the replica has caught up with the primary.
Caching Strategy: Use cache invalidation strategies to ensure that client requests hit the latest data.

#### Implementation
Example:

PYTHON
import time

def write_data(data):
    # Write logic for the primary database
    pass

def read_data():
    start_time = time.time()

    while True:
        if is_replica_catchup():
            return get_latest_data_from_replica()
        elif (time.time() - start_time) > timeout:  # Define a reasonable timeout
            raise Exception("Replica catch-up timed out")

Testing and Validation:

Perform thorough testing with various scenarios to ensure that reads always reflect the latest data.
Monitor performance metrics such as read latency, write throughput, and system resource usage.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, ensuring read-after-write consistency involves carefully managing replication lag and cache invalidation. The key is to implement mechanisms that guarantee that reads on replicas are only served after the replica has caught up with the primary database.

#### Common Mistakes

Ignoring initial replication lag and assuming it will resolve quickly.
Not using proper logging and monitoring tools.
Failing to invalidate in-memory caches properly after writes.
Overlooking edge cases where write-heavy operations might cause performance bottlenecks.

For follow-ups, consider discussing more advanced techniques like multi-master setups, distributed caching strategies, or even using a message queue for asynchronous writes and reads. These can provide additional layers of control over data consistency.

