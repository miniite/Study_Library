---
type: "practice"
difficulty: "Deep"
---

# Your API must return a consistent ordering for a leaderboard endpoint under heavy concurrent score updates. How do you design pagination and tie-breaking so a client scrolling doesn't see items jump around?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Consistent Pagination](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Consistent_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a client scrolling through a leaderboard does not see items jumping around, we need to address two primary challenges: consistent ordering under concurrent updates and efficient pagination. The key is to design an API that maintains stability in the order of displayed results while minimizing the number of database reads.

The question is really probing how you can handle concurrent writes efficiently without causing reordering or flickering in the UI. This involves understanding the nature of leaderboard data, using appropriate locking mechanisms or optimistic concurrency control, and implementing pagination strategies that account for these constraints.

Let’s start by assuming a scenario where scores are updated frequently and the leaderboard must be consistent across multiple client requests. We'll use optimistic locking to prevent reordering issues and introduce a unique timestamp field to handle tie-breaking scenarios.

#### Triaging the Question
Understanding Concurrent Score Updates: Scores can change rapidly due to updates from various sources.

#### Cons
istency Under Concurrency: The order of displayed items should remain consistent across multiple client requests.
Pagination: We need to design pagination so that scrolling through the leaderboard does not show jumps in item order.

#### First-Principles Solution
Database Schema:
User
Score
Timestamp
API Design: Implement a GET endpoint that returns the leaderboard with an optional page and pageSize parameter.
Optimistic Locking: Use a version or timestamp field to ensure no concurrent updates cause reordering.
Tie-Breaking Logic: Introduce a unique tie-breaking mechanism when scores are equal.
Detailed Implementation

Database Schema:

User
+String userId
+Integer score
+DateTime timestamp

API Endpoint Design:

HTTP
GET /leaderboard?page=1&pageSize=5

Optimistic Locking and Tie-Breaking Logic:

Each time a user updates their score, the timestamp field is updated.
When fetching the leaderboard, include this timestamp in the query to ensure consistency.

Pagination Implementation:

PYTHON
def get_leaderboard(page, page_size):
    start = (page - 1) * page_size
    end = start + page_size

    # Fetch all users with a limit and offset based on pagination
    users = User.query.order_by(User.score.desc(), User.timestamp.desc()).limit(end).offset(start)

    # Convert to JSON response
    return [user.to_dict() for user in users]

Edge Cases:

If the timestamp field is not used, frequent concurrent updates might still cause flickering.
Ensure that all client requests use the latest version of the data.

#### Closing Summary

To prevent items from jumping around when scrolling through a leaderboard under heavy concurrent score updates, we need to implement a consistent ordering mechanism using optimistic locking and tie-breaking logic. By ensuring each update includes a unique timestamp field and fetching the leaderboard with this field included, we can maintain a stable order of displayed results.

#### Common Mistakes/Follow-Ups
Forgetting to include a version or timestamp field can lead to reordering issues.
Not considering edge cases where multiple updates occur almost simultaneously.
Failing to handle pagination efficiently might result in unnecessary database queries.

**Comparison:**

```
userId	score	timestamp
u3	90	2023-09-03T00:00:00Z
u2	95	2023-09-02T00:00:00Z
u1	100	2023-09-01T00:00:00Z
```

**Code:**

```python
from flask import Flask, request
from sqlalchemy.orm import Session

app = Flask(__name__)

def get_leaderboard(page, page_size):
    start = (page - 1) * page_size
    end = start + page_size

    with Session() as session:
        users = (
            session.query(User)
            .order_by(User.score.desc(), User.timestamp.desc())
            .limit(end)
            .offset(start)
            .all()
        )

        return [user.to_dict() for user in users]
```

**Example input / output:**

```text
GET /leaderboard?page=1&pageSize=5 HTTP/1.1
Host: leaderboard-api.com

HTTP/1.1 200 OK
Content-Type: application/json

[
    {"userId": "u3", "score": 90, "timestamp": "2023-09-03T00:00:00Z"},
    {"userId": "u2", "score": 95, "timestamp": "2023-09-02T00:00:00Z"},
    {"userId": "u1", "score": 100, "timestamp": "2023-09-01T00:00:00Z"}
]
```

**Data:**

```json
{
  "users": [
    {"userId": "u1", "score": 100, "timestamp": "2023-09-01T00:00:00Z"},
    {"userId": "u2", "score": 95, "timestamp": "2023-09-02T00:00:00Z"},
    {"userId": "u3", "score": 90, "timestamp": "2023-09-03T00:00:00Z"}
  ]
}
```

**Flow:**

1. User Updates Score
2. Update User Record with NewTimestamp
3. Fetch Leaderboard Sorted byScore and Timestamp
4. Return Leaderboard to Client
5. Client Displays LeaderboardConsistently

