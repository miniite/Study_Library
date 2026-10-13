---
type: "practice"
difficulty: "Deep"
---

# You need a distributed lease so exactly one worker runs a scheduled nightly reindex even if the holder GC-pauses. Explain why a plain lock is unsafe here and where the fencing token must be checked to prevent a double reindex.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Distributed Locks and Fencing](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/Distributed_Locks_and_Fencing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that exactly one worker performs a scheduled nightly reindex, we need to implement a distributed lease mechanism. A plain lock is insufficient because it does not account for the possibility of a worker being GC-paused (garbage collected and paused by the JVM) or failing to release the lock due to an unexpected crash. We must use a fencing token that is checked on each reindex attempt to prevent double reindexing.

#### Triaging the Question

The core issue here is ensuring atomicity in distributed systems, where multiple workers might try to acquire a lock simultaneously but only one should proceed with the reindex. A plain lock (like a distributed mutual exclusion) can fail because:

GC-Pauses: If the worker holding the lease gets paused by the JVM's garbage collection process, another worker might assume it has released the lock.
Crashes: The worker could crash before releasing the lock, leaving other workers to think they can proceed.
Detailed Solution

To address these issues, we need a fencing token:

Fencing Token Concept:
A fencing token is a unique identifier that changes over time and is checked on each reindex attempt.
Token Distribution:
The worker holding the lease periodically updates this token in a consistent way (e.g., incrementing it).
Token Checking Mechanism:
Before starting the reindex, the worker checks the current fencing token value.
If the token has changed since its last check, the worker knows another worker has already started the reindex and should not proceed.
Step-by-Step Solution
Worker Initialization:
Each worker initializes a fencing token with a unique starting value (e.g., system time in milliseconds).
Token Update Mechanism:
The worker holding the lease updates the token periodically (every minute, for example) to ensure it stays current.
Reindex Check and Execution:
Before performing the reindex, the worker checks the current token value.
If the token matches what the worker expects, it proceeds with the reindex; otherwise, it skips the operation.
Code Example

Below is a simplified example of how this might look in pseudocode:

PSEUDOCODE
# Worker Reindex Function
function performReindex(token):
    # Fetch current token value from distributed storage
    currentToken = fetchCurrentToken()

    # Check if token has changed since last check
    if currentToken == expectedToken:
        # Proceed with reindex
        executeReindex()
        # Update the local state to indicate we are holding the lease
        updateLocalState("holding")
    else:
        # Skip reindex as another worker already started it
        logInfo("Another worker is performing the reindex.")

# Worker Lease Management Function
function manageLease():
    while True:
        token = fetchNextToken()  # Increment and distribute new token

        # Periodically check for reindex requests
        if canPerformReindex(token):
            performReindex(token)

#### Common Mistakes
and Follow-Ups
Mistake: Not implementing the fencing mechanism, leading to potential double indexing.

#### Follow-Up
Discussing additional fail-safes, such as timeout mechanisms or retries if the token check fails temporarily.
Further Discussion: Exploring other distributed lease approaches like using a leader election algorithm (e.g., Raft) for more robust solutions.

**Comparison:**

```
Token	Current Value	Expected Value	Action
12345	1234567890	1234567890	Proceed with reindexing.
```

**Code:**

```
PSEUDOCODE
Copy
function performReindex(token):
    currentToken = fetchCurrentToken()
    if currentToken == expectedToken:
        executeReindex()
        updateLocalState("holding")
    else:
        logInfo("Another worker is performing the reindex.")

function manageLease():
    while True:
        token = fetchNextToken()
        if canPerformReindex(token):
            performReindex(token)
```

**Example input / output:**

```text
Input:
- Worker A starts and fetches token value.

Output:
- Worker A checks the current token (1234567890) against its expected value.
- Since both match, Worker A proceeds with reindexing.
```

**Data:**

```json
{
  "token": {
    "type": "integer",
    "value": 1234567890,
    "increment": true
  },
  "expectedToken": 1234567890,
  "currentToken": 1234567890
}
```

**Flow:**

1. Match
2. No Match
3. Worker Starts
4. Check Token
5. Perform Reindex
6. Skip Reindex and Log

