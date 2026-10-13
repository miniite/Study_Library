---
type: "practice"
difficulty: "Deep"
---

# You have to decide the granularity of your REST resources: fewer coarse endpoints tuned for your UI, or many normalized fine-grained ones. How do you weigh coupling, cacheability, and reusability, and what do you choose for a fast-moving product?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Coarse vs Fine-grained Endpoints](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Coarse_vs_Fine-grained_Endpoints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding the granularity of REST resources in a fullstack application, we must balance several factors including coupling between layers, cacheability, and reusability. For a fast-moving product, choosing the right level of granulation can significantly impact performance, maintainability, and user experience. Here’s how to approach this decision:

Firstly, let's clarify what we mean by "granularity." Coarse-grained endpoints serve broad purposes, often tied closely to specific UI actions. Fine-grained endpoints break down a task into smaller, more granular operations.

Coupling refers to the extent to which different parts of the system are interdependent. Coupled systems can be harder to maintain and test, as changes in one part affect others. Cacheability deals with how easily responses from these APIs can be cached, reducing load on servers and improving performance. Reusability is about whether an API endpoint can serve multiple purposes or clients.

In a fast-moving product, you want to ensure that each choice supports rapid development cycles while maintaining the system's overall health. This requires weighing the trade-offs between coupling, cacheability, and reusability carefully.

To illustrate this, let’s consider two scenarios: using coarse-grained endpoints for UI-driven actions versus fine-grained endpoints for more granular operations.

Coarse-Grained Endpoints

#### Example
A GET /user endpoint that fetches all user information including their profile, recent activities, and preferences. This single endpoint could be called by the UI to update the user's dashboard.

User Interface
GET /user
Server Logic

#### Pros

Simplifies client-side logic as there are fewer endpoints to manage.
Reduces network calls and potentially improves performance in the short term.

#### Cons

Higher coupling between UI and server. Changes to user preferences might require changes in both places.
Less cacheable because each request brings back a lot of data, which may not always be needed or change frequently.
Harder to reuse this endpoint for other clients or services that only need specific parts of the user information.
Fine-Grained Endpoints

#### Example
Separate endpoints like GET /user/profile, GET /user/activities, and GET /user/preferences. Each endpoint serves a distinct piece of data.

User Interface
GET /user/profile
GET /user/activities
GET /user/preferences
E2,E3,E4
Server Logic

#### Pros

Lower coupling because each client can request only the data it needs, reducing redundant data transfer.
More cacheable. Smaller payloads mean better caching efficiency and reduced load on servers.
Higher reusability. These endpoints could be used by other clients or microservices that require specific pieces of user information.

#### Cons

Increased complexity in client-side logic as more endpoints need to be managed.
Potentially higher network overhead due to multiple requests, though modern HTTP/2 and caching strategies can mitigate this.
More server-side processing required for each request, which could impact performance if not optimized properly.
Decision Factors
Product Velocity: For a fast-moving product, fine-grained endpoints can accelerate development by allowing teams to incrementally add functionality without disrupting existing workflows.
Resource Constraints: If bandwidth and computational resources are limited, coarse-grained endpoints might be more suitable as they reduce the number of network calls and server processing demands.
Client-Side Complexity: Consider the complexity your client developers will face with each endpoint. Fine-grained APIs can make development easier by providing clear separation of concerns.

#### Conclusion

For a fast-moving product, fine-grained endpoints are often preferable due to their ability to enhance reusability and cacheability while reducing coupling. However, this decision should be revisited periodically as the application evolves. A hybrid approach might also work well—using coarse-grained APIs for frequently used operations and finer-grained ones for more specialized needs.

Common pitfalls include:

Over-normalizing endpoints which can complicate client-side logic unnecessarily.
Underestimating the performance benefits of fewer, broader calls in favor of many small requests.
Ignoring the importance of reusability in future integrations or microservices architectures.

**Comparison:**

| Type           | Coarse-Grained          | Fine-Grained                |
|----------------|------------------------|----------------------------|
| Coupling       | High                    | Low                         |
| Cacheability   | Lower                   | Higher                     |
| Reusability    | Lower                   | Higher                     |

**Code:**

```python
# Example of a coarse-grained API endpoint in Flask
@app.route('/user', methods=['GET'])
def get_user():
    user = User.query.get(user_id)
    return jsonify({
        'id': user.id,
        'name': user.name,
        'profile': user.profile,
        'activities': [activity.serialize() for activity in user.activities],
        'preferences': user.preferences
    })

# Example of a fine-grained API endpoint
@app.route('/user/profile', methods=['GET'])
def get_user_profile():
    user = User.query.get(user_id)
    return jsonify({
        'id': user.id,
        'name': user.name,
        'profile': user.profile
    })
```

**Example input / output:**

```text
# Input for Coarse-Grained Endpoint
GET /user

# Output for Coarse-Grained Endpoint
{
  "id": 1,
  "name": "Alice",
  "profile": "Developer",
  "activities": [
    {"date": "2023-10-01", "action": "Code Review"},
    {"date": "2023-10-02", "action": "Bug Fix"}
  ],
  "preferences": {
    "theme": "dark",
    "notifications": true
  }
}

# Input for Fine-Grained Endpoint (Profile)
GET /user/profile

# Output for Fine-Grained Endpoint (Profile)
{
  "id": 1,
  "name": "Alice",
  "profile": "Developer"
}
```

**Data:**

```json
{
  "user": {
    "id": 1,
    "name": "Alice",
    "profile": "Developer",
    "activities": [
      {"date": "2023-10-01", "action": "Code Review"},
      {"date": "2023-10-02", "action": "Bug Fix"}
    ],
    "preferences": {
      "theme": "dark",
      "notifications": true
    }
  }
}
```

**Flow:**

1. User Interface
2. GET /user
3. Server Logic
4. GET /user/profile
5. GET /user/activities
6. GET /user/preferences

