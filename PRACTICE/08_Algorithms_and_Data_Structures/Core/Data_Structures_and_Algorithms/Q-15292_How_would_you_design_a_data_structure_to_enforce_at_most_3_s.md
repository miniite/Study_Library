---
type: "practice"
difficulty: "Deep"
---

# How would you design a data structure to enforce 'at most 3 sessions per user' across a distributed auth service, and where does an in-memory counter break under node failure?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Concurrent Counters and CAS](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Concurrent_Counters_and_CAS.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of enforcing "at most 3 sessions per user" in a distributed authentication system, we need to design a data structure that can handle concurrent operations and manage session limits efficiently. The challenge here is ensuring consistency across multiple nodes while minimizing complexity and overhead.

The key question is where an in-memory counter breaks under node failure. This scenario highlights the limitations of relying solely on in-memory storage for state management. An in-memory solution might work well during normal operation but will fail to maintain data integrity if a node crashes, leading to potential session count discrepancies and security risks.

To solve this problem effectively, we need to adopt a more robust approach that can handle distributed transactions and provide fault tolerance.

#### Triage
Understanding the Problem

The core of the question involves designing a system to track user sessions with a strict limit. The in-memory counter approach is straightforward but fails under node failure due to lack of persistence. We must explore a distributed solution that ensures session limits are enforced even when nodes go down.

#### First-Principles Solution
Distributed Counter: Use a distributed counting mechanism like Redis or a distributed key-value store (e.g., Cassandra) with built-in atomic increment capabilities.
Pessimistic Locking: Implement pessimistic locking to prevent concurrent updates from violating the session limit.
Eventual Consistency: Ensure that all nodes eventually converge on the correct state, but allow for temporary inconsistencies.
Step-by-Step Solution

Data Model:

User ID: Unique identifier for each user.
Session Count: Number of active sessions for a given user.

Distributed Counter Implementation:

Use Redis to store the session count as a key-value pair where the key is user:<user_id> and the value is an integer representing the number of sessions.

Session Management:

On login, increment the session count in Redis.
On logout, decrement the session count.
Before allowing a new login, check if the current session count is less than 3.
Use WATCH and EXECUTE commands to ensure atomicity during increments.

Pessimistic Locking:

Implement a lock mechanism using Redis' SETNX (Set if Not Exists) command to acquire a lock before modifying the session count.
Release the lock after completing the operation.

Fault Tolerance:

Use a distributed consensus protocol like Raft or Paxos to ensure that all nodes eventually agree on the state of the sessions.
Implement a heartbeat mechanism to detect and handle node failures.
Code Example
PYTHON
import redis

def manage_session(user_id):
    r = redis.Redis(host='localhost', port=6379, db=0)

    # Acquire lock
    lock_key = f"user:{user_id}:lock"
    if not r.set(lock_key, '1', nx=True, ex=5):  # Lock for 5 seconds
        return "Failed to acquire lock."

    try:
        session_count_key = f"user:{user_id}"

        current_sessions = r.get(session_count_key)
        if current_sessions is None:
            current_sessions = 0

        # Check and enforce limit
        if int(current_sessions) >= 3:
            return "Session limit reached."

        new_session_count = int(current_sessions) + 1
        if r.setnx(session_count_key, new_session_count):
            print(f"Logged in user {user_id}. Current sessions: {new_session_count}")
        else:
            return "Failed to increment session count."

    finally:
        # Release lock
        r.delete(lock_key)

#### Common Mistakes
and Follow-Ups
Mistake 1: Failing to use transactions or locks, leading to race conditions.
Mistake 2: Overlooking fault tolerance, which can result in inconsistent session counts during node failures.

#### Follow-Up
Discussing alternative storage solutions like distributed databases with built-in transactional support (e.g., Cassandra).

#### Closing Summary

By using a distributed counter and implementing pessimistic locking, we ensure that the system enforces the "at most 3 sessions per user" rule even in the presence of node failures. The solution leverages Redis for simplicity and fault tolerance mechanisms to maintain state consistency.

