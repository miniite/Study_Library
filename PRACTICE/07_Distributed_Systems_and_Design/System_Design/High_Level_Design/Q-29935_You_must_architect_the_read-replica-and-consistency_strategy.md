---
type: "practice"
difficulty: "Deep"
---

# You must architect the read-replica-and-consistency strategy for a product where users must see their own edits instantly but stale reads are fine for others — how do you route sticky reads use write-tokens and keep the fast path off the primary?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-region Write Paths](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Cross-region_Write_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a read-replica-and-consistency strategy for your product, we must ensure that each user sees their own edits instantly while allowing stale reads to be acceptable for others. This requires a nuanced approach involving write tokens and intelligent routing strategies. The primary challenge is balancing the load on the main database with minimizing staleness in secondary reads.

Firstly, let's break down what we need:

Immediate consistency for writes (writes go directly to the primary).
Stale but consistent reads for others (reads from replicas).
A mechanism to route sticky reads (reads that users want to be as fresh as possible) without overwhelming the primary.

The key idea is to use write tokens and a smart routing algorithm to manage these requirements efficiently.

**Comparison:**

```
Route	Description
Replicas	Used for stale reads. Helps in offloading the primary database and ensuring high availability.
Primary	Used for sticky reads by users who need to see their latest edits immediately.
```

**Code:**

```python
# Example of a simplified token system in Python
class WriteToken:
    def __init__(self, user_id):
        self.user_id = user_id

def route_read_request(token=None):
    if token is None or not token.user_id:
        # Route to replica for stale reads
        return "replica"
    else:
        # Route to primary for sticky reads
        return "primary"

# Example usage
user1_token = WriteToken(user_id=1)
route_read_request()  # Default: replica
route_read_request(token=user1_token)  # Primary
```

**Example input / output:**

```
Input: A user initiates a read request with an optional write token.
Output: The system routes the request to either a replica or the primary database based on whether a valid write token is provided.

Example:

PYTHON
Copy
user1 = {"id": 1, "token": "abc123"}
route_read_request()  # Returns 'replica'
route_read_request(token=user1["token"])  # Returns 'primary'
```

**Data:**

```json
{
    "user_tokens": [
        {"id": 1, "token": "abc123"},
        {"id": 2, "token": "def456"}
    ]
}
```

**Flow:**

1. Yes
2. No
3. User Initiates Read Request
4. Has Write Token?
5. Route to Primary
6. Route to Replica

