---
type: "practice"
difficulty: "Deep"
---

# How would you cache and serve a Top-N leaderboard efficiently using a Redis Sorted Set?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Redis Fundamentals](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Redis_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To efficiently cache and serve a Top-N leaderboard, we can leverage Redis's Sorted Sets to store and retrieve scores in an order-preserving manner. This approach ensures fast retrieval times while maintaining the ability to dynamically update the leaderboard as new entries come in.

#### Triage

The question is essentially about designing an efficient system for storing and retrieving top-N elements from a dataset using Redis, which is well-suited for this purpose due to its key-value data structures. Specifically, we'll focus on leveraging Redis's Sorted Sets (ZSET) feature, which allows us to rank elements based on scores.

#### Assumptions
The leaderboard will be updated frequently with new user scores.
We need to fetch the top-N entries quickly.
There are a limited number of unique users (e.g., less than 10,000).
Scores can change over time, and we want to reflect these changes in real-time or near-real-time.

#### Solution
Step-by-Step from First Principles

Modeling the Data:

Each user's score will be stored as a member of a Redis Sorted Set.
The score itself acts as the sort key, allowing us to rank users based on their scores.

Insertion and Deletion:

When a new score is added or an existing one is updated:
Yes
No
User submits score
Update Redis ZSET
Score exists?
Increment score
Add new member with score
Return updated leaderboardtop-N
If the score already exists, we increment it. Otherwise, we add a new entry.

Retrieving Top-N Entries:

To fetch the top-N entries:
User requests top-N scores
Get Redis ZSET range
Return top-N members
This operation is efficient and returns a sorted list of users based on their scores.

Handling Large Numbers of Users:

If the number of unique users grows significantly, consider partitioning the leaderboard into smaller sets or using other techniques like sharding.

Scalability Considerations:

Ensure that Redis is configured to handle high read and write loads efficiently.
Use appropriate network protocols (e.g., TCP) for stability in a production environment.

#### Closing Summary

By utilizing Redis's Sorted Sets, we can design an efficient system for caching and serving top-N leaderboards. This approach ensures both performance and simplicity while accommodating dynamic updates to the leaderboard.

#### Common Mistakes/Follow-Ups
Mistake 1: Using a simple hash or string structure instead of a sorted set, leading to slower retrieval times.
Mistake 2: Not considering edge cases such as ties in scores (how do you handle multiple users with the same score?).

#### Follow-Up
Question: How would you implement persistence and disaster recovery for this system?

**Comparison:**

| User ID | Score |
|---------|-------|
| user_123456789 | 95    |
| user_234567890 | 93    |
| user_345678901 | 92    |
| user_456789012 | 91    |

**Code:**

```python
import redis

# Initialize Redis client
client = redis.Redis(host='localhost', port=6379, db=0)

def update_score(user_id, score):
    # Increment the user's score or add a new one if it doesn't exist
    client.zincrby('leaderboard', amount=score, value=user_id)

def get_top_n(n):
    # Fetch top-N entries from the leaderboard
    return client.zrevrange('leaderboard', 0, n-1, withscores=True)
```

**Example input / output:**

```text
Input:
- update_score('user_987654321', 97)

Output:
- After the operation, the leaderboard might look like this:
  {
    "user_987654321": 97,
    "user_123456789": 95,
    "user_234567890": 93,
    "user_345678901": 92
  }

Input:
- get_top_n(3)

Output:
- [('user_987654321', 97), ('user_123456789', 95), ('user_234567890', 93)]
```

**Data:**

```json
{
  "leaderboard": {
    "user_123456789": 95,
    "user_234567890": 93,
    "user_345678901": 92,
    "user_456789012": 91
  }
}
```

**Flow:**

1. Yes
2. No
3. User submits score
4. Update Redis ZSET
5. Score exists?
6. Increment score
7. Add new member with score
8. Return updated leaderboardtop-N

