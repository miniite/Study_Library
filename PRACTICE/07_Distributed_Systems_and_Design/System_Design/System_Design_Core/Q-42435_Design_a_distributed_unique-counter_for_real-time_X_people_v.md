---
type: "practice"
difficulty: "Intermediate"
---

# Design a distributed unique-counter for real-time 'X people viewing this product' that's cheap and roughly accurate. Why not a per-request DB increment, and what's your decay strategy?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Clickstream Counting Pipeline](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To design a distributed unique counter for tracking real-time 'X people viewing this product,' we need to address several challenges: ensuring uniqueness across multiple servers, achieving scalability, and maintaining accuracy at a reasonable cost. We can use a combination of a consistent hashing ring with a Redis database and implement a decay strategy to handle the staleness issue.

#### Triaging the Question

The question asks us to design a distributed unique counter that tracks real-time 'X people viewing this product.' The key requirements are:

Uniqueness: Each view should be counted only once.
Scalability: The solution must work across multiple servers without centralizing all data on one node.
Cost Efficiency: We want to keep the implementation as simple and cost-effective as possible.
Accuracy: While absolute precision is not required, we need to minimize false positives and negatives.
Step-by-Step Solution
Problem Breakdown

Why Not a Per-Request DB Increment?

Single Point of Failure (SPOF): A single database increment operation can become a bottleneck and SPOF.
Latency: Network latency between multiple clients and the database can introduce significant delays, affecting real-time updates.

#### Cons
istent Hashing Ring with Redis:

We can use a consistent hashing ring to distribute the counter values across multiple Redis instances. Each client hashes the unique ID (e.g., user session or IP address) to determine which Redis instance is responsible for incrementing the count.

Decay Strategy:

To maintain accuracy, we need to periodically reset counters to avoid false positives due to clients who have been inactive.

#### Implementation

#### Cons
istent Hashing Ring:

Use a consistent hashing algorithm like MurmurHash or CRC32 on the client side to map unique IDs to Redis instances.

Redis Database for Counters:

Each Redis instance will maintain a counter value corresponding to its assigned range of user IDs.
Increment operations are fast and atomic within each Redis instance, avoiding the need for complex locking mechanisms.

Decay Strategy Implementation:

Every client should periodically send an "alive" signal to their respective Redis instance.
If no signal is received after a certain period (e.g., 5 minutes), the Redis instance marks that user ID as inactive and resets its counter value.
To handle stale data, we can implement periodic background tasks on each Redis instance to check for inactive entries and reset them.

#### Example
Workflow

Client Request:

PLAINTEXT
Client hashes session ID -> gets server responsible for incrementing count -> sends increment request -> server increments count in Redis -> sends updated count back to client.

Decay Check:

Periodic task runs on each Redis instance:
PLAINTEXT
For each user ID, check if the last alive signal was more than 5 minutes ago -> If yes, mark as inactive and reset counter.

#### Trade-offs
Accuracy vs. Cost: While this approach is cost-effective due to distributed nature and minimal database operations, it may introduce some inaccuracies in real-time tracking due to decay strategy.

#### Closing Summary

By using a consistent hashing ring with Redis for our unique counter system, we can achieve scalability, reduce single points of failure, and minimize costs. The decay strategy ensures that the counters remain roughly accurate by periodically marking inactive users and resetting their counts.

#### Common Mistakes/Follow-Ups
Not Handling Inactive Users: Failing to implement a decay mechanism can lead to false positives.
Ignoring Network Latency: Assuming all clients are equally close to the database can cause issues in real-world scenarios with varying network conditions.
Not Considering Edge Cases: What happens if a client disconnects and reconnects? How do you handle retries without causing excessive increments?

**Comparison:**

```
Client	Session ID	Responsible Server
0	session123	server1
1	session456	server2
```

**Code:**

```python
# Pseudocode for consistent hashing ring assignment
def assign_redis_server(session_id):
    hashed_session_id = hash_function(session_id)
    # Return the server responsible for incrementing count based on hashed session ID

# Example of a Redis client increment operation
import redis

r = redis.Redis(host='localhost', port=6379, db=0)
user_id = "session123"
count = r.incr(f'counter:{user_id}')
```

**Example input / output:**

```
Input: User session ID (e.g., session123)
Output: Incremented count for the corresponding Redis instance
PLAINTEXT
Copy
Client sends 'session123' -> Server assigns to 'server1' -> server1 increments counter -> Response with new count.
```

**Data:**

```json
{
  "servers": ["server1", "server2", "server3"],
  "hashes": {
    "session123": "server1",
    "session456": "server2"
  }
}
```

**Flow:**

1. Yes
2. Client Sends Request
3. Is Session ID Valid?
4. Hash Session ID
5. Assign Responsible RedisServer
6. Send Increment Request toRedis Instance
7. Receive Updated Count fromRedis
8. Return New Count to Client

