---
type: "practice"
difficulty: "Expert"
---

# When you denormalize per query pattern into Cassandra you get write amplification. How do you manage and bound it as query patterns grow?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Denormalization and Locking](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Denormalization_Trade-offs_and_Patterns/Denormalization_and_Locking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with Cassandra, write amplification due to denormalization can be a significant challenge as query patterns grow. Essentially, the problem arises when frequent updates in one table lead to multiple writes across many tables, increasing write latency and reducing throughput. To manage and bound this issue, you need to adopt a multi-faceted approach that includes strategic data modeling, caching strategies, and optimization techniques. Let's break down how you can address this challenge.

**Code:**

```python
# Example code snippet for querying Cassandra using DataStax Python driver
from cassandra.cluster import Cluster

cluster = Cluster(['127.0.0.1'])
session = cluster.connect()

def get_user_profile(user_id):
    query = "SELECT * FROM user_profiles WHERE user_id=%s"
    result = session.execute(query, (user_id,))
    return result.one()
```

**Example input / output:**

```

Given an input:

PYTHON
Copy
get_user_profile(123)

Output:

JSON
Copy
{
  "user_id": 123,
  "username": "john_doe",
  "email": "john@example.com",
  "phone": "+1-555-1234"
}
```

**Data:**

```json
{
  "user_id": 123,
  "username": "john_doe",
  "email": "john@example.com",
  "phone": "+1-555-1234"
}
```

