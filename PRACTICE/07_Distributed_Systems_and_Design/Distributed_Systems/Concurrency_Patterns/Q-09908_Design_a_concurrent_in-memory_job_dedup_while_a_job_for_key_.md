---
type: "practice"
difficulty: "Deep"
---

# Design a concurrent in-memory job dedup: while a job for key K is running, new requests for K should attach to the running one, not start a second. Where's the race between "check running" and "register running"?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Concurrent Data Structures](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Concurrent_Data_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a concurrent in-memory job deduplication system, we need to ensure that if a job with key ( K ) is already running, any new requests for this key should not start a duplicate job but instead attach to the existing one. The challenge here is managing concurrency and ensuring atomicity during checks and updates. This problem primarily revolves around race conditions where multiple concurrent threads might interfere with each other.

#### Triaging the Question

The core issue we need to address is detecting whether a job for key ( K ) is already running before registering it as "running." The critical race condition happens between checking if a job is running and actually marking it as such. If not handled properly, multiple threads could erroneously detect that no job is running and start new instances simultaneously.

#### Assumptions
Each job has a unique key ( K ).
Jobs are initiated by sending requests for specific keys.
We need to maintain state in memory (in a map or similar structure) to track which jobs are currently running.
Concurrency control mechanisms are available and correctly implemented.

#### First-Principles Solution
Shared State Management: Use a thread-safe data structure to manage the state of each job key.
Atomic Operations: Ensure that operations like checking if a job is running and updating its status are atomic.
Concurrency Control: Implement locking mechanisms (e.g., mutex) or use higher-level synchronization constructs.
Step-by-Step Solution

Define the Data Structure:

Use a concurrent map to store the state of each job key, e.g., Map<K, AtomicBoolean> where K is the job key and AtomicBoolean tracks whether the job is running.

Check and Register Job:

When receiving a new request for key ( K ):
Check if there's already an ongoing job using an atomic read.
If not, proceed to mark it as "running."
Otherwise, attach the new request to the existing running job.

Here’s how this can be implemented in code:

JAVA
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicBoolean;

public class JobDeduplicator {
    private final ConcurrentHashMap<K, AtomicBoolean> jobStates = new ConcurrentHashMap<>();

    public void handleRequest(K key) {
        // Atomic read to check if the job is running.
        AtomicBoolean isRunning = jobStates.get(key);
        boolean wasRunning = isRunning != null && isRunning.get();

        if (!wasRunning) {
            // Mark as "running" atomically.
            try {
                boolean registered = jobStates.computeIfAbsent(key, k -> new AtomicBoolean(true)).compareAndSet(false, true);
                if (registered) {
                    // Handle the new request logic here.
                } else {
                    // The job was running; attach the new request to it.
                }
            } finally {
                // Ensure cleanup in case of an exception or race condition.
                jobStates.put(key, new AtomicBoolean(false));
            }
        } else {
            // Attach the new request to the existing running job if necessary.
            // This part depends on your application logic.
        }
    }
}

#### Closing Summary

In summary, to effectively handle concurrent in-memory job deduplication, we need a robust mechanism that ensures atomicity and thread-safety. By leveraging concurrent data structures and atomic operations, we can prevent race conditions where multiple threads might erroneously start new jobs for the same key.

#### Common Mistakes/Follow-Ups
Ignoring Atomicity: Failing to use atomic operations to safely check and update job states.
Incorrect Handling of Failures: Not properly managing cleanup or reinitialization in case of exceptions.
Missing Concurrency Control: Overlooking the need for proper locking mechanisms, which can lead to race conditions.

For further discussion, common follow-ups might include:

How would you scale this solution if more jobs are expected?
What happens when a job fails and needs to be restarted?
How do you ensure data consistency across multiple nodes in a distributed system?

**Code:**

```java
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicBoolean;

public class JobDeduplicator {
    private final ConcurrentHashMap<K, AtomicBoolean> jobStates = new ConcurrentHashMap<>();

    public void handleRequest(K key) {
        // Atomic read to check if the job is running.
        AtomicBoolean isRunning = jobStates.get(key);
        boolean wasRunning = isRunning != null && isRunning.get();

        if (!wasRunning) {
            // Mark as "running" atomically.
            try {
                boolean registered = jobStates.computeIfAbsent(key, k -> new AtomicBoolean(true)).compareAndSet(false, true);
                if (registered) {
                    // Handle the new request logic here.
                } else {
                    // The job was running; attach the new request to it.
                }
            } finally {
                // Ensure cleanup in case of an exception or race condition.
                jobStates.put(key, new AtomicBoolean(false));
            }
        } else {
            // Attach the new request to the existing running job if necessary.
            // This part depends on your application logic.
        }
    }
}
```

**Flow:**

1. Yes
2. Receive Request
3. Is Job Running?
4. Attach to Existing Job

