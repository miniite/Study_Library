---
type: "practice"
difficulty: "Intermediate"
---

# Your service's p99 is dominated by a slow N+1 in an ORM you can't easily rewrite. How do you find it in traces, batch it with a dataloader/IN query, and prevent regressions in review?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tail Latency After Expansion](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Tail_Latency_After_Expansion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to identify and optimize slow queries caused by N+1 issues, then ensure efficient batching through DataLoader or similar mechanisms. Finally, we must implement robust regression tests to prevent future regressions.

#### Overview
The core issue is a slow N+1 query pattern in an ORM that we can't easily rewrite. We'll use tracing tools to identify the problematic queries, batch them using DataLoader for performance improvements, and add unit tests to safeguard against future changes causing regression.

#### Triage

The question asks us to find N+1 issues from traces, optimize them with batching techniques, and ensure these optimizations don't regress in code reviews.
We assume the service uses a relational database and an ORM like Hibernate or Entity Framework.
The primary goal is to maintain performance while adhering to coding standards.

#### First Principles Solution

Identify N+1 Queries Using Tracing Tools:

Use tracing tools (e.g., Jaeger, Zipkin) to instrument the application. These tools can provide insights into SQL query execution times and identify slow queries.
Implement query profiling in development environments to capture and analyze slow queries.

Optimize with DataLoader/Batched Queries:

Once identified, optimize the ORM mappings or database queries by batching related queries together using a DataLoader pattern.
For example, if fetching a user also requires fetching their associated posts, use a batched query to fetch both in one go rather than making separate requests for each post.

Implement Regression Tests:

Write unit tests that simulate the problematic scenarios and verify they perform well under load.
Use tools like Pact or Mockserver to create test doubles that mimic actual database responses, ensuring changes don't inadvertently break existing functionality.

Ensure Changes Do Not Regress During Reviews:

Integrate static code analysis tools (e.g., SonarQube) into the CI/CD pipeline to catch potential regressions.
Establish code review practices where peers check for performance implications of new features or changes.

Monitor and Iterate:

Continuously monitor application performance metrics after implementing optimizations.
Use APM tools (e.g., New Relic, Datadog) to set up alerting on performance anomalies.

**Comparison:**

```
User ID	Name	Posts
1	User One	Post A, Post B
2	User Two	Post C
3	User Three	None
```

**Code:**

```python
# Example of DataLoader pattern in Python using SQLAlchemy
from sqlalchemy.orm import Session

class UserLoader:
    def __init__(self, session: Session):
        self.session = session

    def batch_load_fn(self, user_ids):
        # Batch load users and their posts
        users_and_posts = self.session.query(User).options(joinedload(User.posts)).filter(User.id.in_(user_ids))
        return {user.id: (user, [post for post in user.posts]) for user in users_and_posts}
```

**Example input / output:**

```
Input: user_loader = UserLoader(session)
Output:
PYTHON
Copy
results = user_loader.batch_load_fn([1, 2, 3])
print(results)

Outputs:

JSON
Copy
{
  "1": {
    "id": 1,
    "name": "User One",
    "posts": [{"title": "Post A"}, {"title": "Post B"}]
  },
  "2": {
    "id": 2,
    "name": "User Two",
    "posts": [{"title": "Post C"}]
  },
  "3": {
    "id": 3,
    "name": "User Three",
    "posts": []
  }
}
```

**Data:**

```json
{
  "user_id": [1, 2, 3],
  "expected_result": [
    {
      "id": 1,
      "name": "User One",
      "posts": [{"title": "Post A"}, {"title": "Post B"}]
    },
    {
      "id": 2,
      "name": "User Two",
      "posts": [{"title": "Post C"}]
    },
    {
      "id": 3,
      "name": "User Three",
      "posts": []
    }
  ]
}
```

**Flow:**

1. Instrument Application
2. Identify N+1 Queries
3. Batch Queries with DataLoader
4. Integrate Regression Tests
5. Maintain Code Reviews andStatic Analysis
6. Monitor Performance MetricsContinuously

