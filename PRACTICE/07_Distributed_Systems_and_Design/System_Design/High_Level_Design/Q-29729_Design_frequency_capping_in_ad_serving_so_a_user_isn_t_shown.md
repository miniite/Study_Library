---
type: "practice"
difficulty: "Deep"
---

# Design frequency capping in ad serving so a user isn't shown the same ad more than N times in 24h across devices — where do you store the per-user counters and how do you keep the check under a few ms?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Real-time Price Feeds](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Real-time_Price_Feeds.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To design frequency capping for ad serving, we need to ensure that a user is not shown the same ad more than (N) times in 24 hours across multiple devices. This involves storing per-user counters and efficiently checking them within milliseconds. We can use distributed databases with cache layers and rate limiting mechanisms.

#### Triaging the Question

The core of this question revolves around ensuring frequency capping—limiting ad exposure to users over a defined period—to maintain user experience and prevent abuse. The key aspects are:

Storage: Where should we store the per-user counters?
Efficiency: How can we keep checks under 1ms?

#### First Principles Solution
Key Assumptions
Users have multiple devices.
Each ad has a unique identifier.
(N) is the maximum number of times an ad can be shown to a user in 24 hours.
We need real-time or near-real-time updates and checks.
Design Steps
User-ID Storage: Use a distributed system (e.g., Cassandra, DynamoDB) for storing per-user counters.
Ad ID Storage: Store ad IDs in a similar system to manage their usage count.
Rate Limiting Mechanism: Implement rate limiting and caching strategies.
Detailed Solution

Data Model

User Counters Table: Store user-specific counters.
PLAINTEXT
UserCounters (user_id, device_id, ad_id, counter)
Ad Usage Table: Track usage of each ad across all devices.
PLAINTEXT
AdUsage (ad_id, timestamp, count)

Distributed Database

Use a distributed database like Cassandra or DynamoDB to ensure high availability and scalability.

Caching Layer

Implement an in-memory cache (e.g., Redis) for quick lookups.
Cache the per-user counters and ad usage data.

Rate Limiting Algorithm

Check if the user has already seen the ad within 24 hours by querying the database and cache layer.
If not, increment the counter in both the database and cache.

#### Implementation
Example

Yes
No
Rate Exceeded
User
Device
Ad Request
Cache Hit?
Show Ad
DB Check & IncrementCounter
Show Ad
Do Not Show Ad, LogViolation

#### Closing Summary

By using a distributed database for persistent storage and an in-memory cache for fast lookups, we can efficiently manage frequency capping. The key is to ensure that the system scales well and provides real-time updates.

#### Common Mistakes
and Follow-Ups
Mistake: Failing to implement rate limiting or caching mechanisms, leading to inefficient queries.

#### Follow-Up
Discussing how to handle edge cases, such as network partitions or failures in the distributed systems. Consider adding retries and fallback strategies.

**Comparison:**

| User | Device | Ad ID | Counter |
|------|--------|-------|---------|
| user_123456789 | device_A | ad_123456789 | 0 |

**Code:**

```python
# Example Python code for checking frequency capping using Redis cache

import redis

def check_frequency_capping(user_id, ad_id):
    r = redis.Redis(host='localhost', port=6379, db=0)

    # Check cache first
    key = f"user_{user_id}_ad_{ad_id}"
    if r.exists(key):
        return True  # Ad already shown to user

    # Check database (if cache miss)
    with r.pipeline() as pipe:
        pipe.hget('user_counters', user_id, ad_id)
        pipe.hget('ad_usage', ad_id)
        _, counter = pipe.execute()

    if not counter or int(counter) >= N:  # N is the max allowed shows
        return False

    # Update counters and cache
    r.setex(key, 86400, 1)  # Cache for 24 hours
    r.hincrby('user_counters', user_id, ad_id)
    r.hincrby('ad_usage', ad_id)

    return True
```

**Example input / output:**

```text
Input: Check if user with ID 'user_123456789' can see ad 'ad_123456789'
Output: True (if they haven't seen it in the last 24 hours)
```

**Data:**

```json
{
  "UserCounters": {
    "user_123456789": {
      "device_A": 5,
      "device_B": 0
    }
  },
  "AdUsage": {
    "ad_123456789": 10
  }
}
```

**Flow:**

1. Yes
2. No
3. Rate Exceeded
4. User
5. Device
6. Ad Request
7. Cache Hit?
8. Show Ad
9. DB Check & IncrementCounter
10. Show Ad
11. Do Not Show Ad, LogViolation

