---
type: "practice"
difficulty: "Expert"
---

# How would you architect an entitlement service that answers 'can user U do action A on resource R' in under 10ms p99 at 200k QPS while keeping policy evaluation auditable and centrally managed?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-border Access Governance](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Access_Governance/Cross-border_Access_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect an entitlement service that answers "can user U do action A on resource R" in under 10ms p99 at 200k QPS while maintaining policy evaluation auditable and centrally managed, we need to consider several key aspects of the problem: performance, scalability, security, governance, and auditability. The solution will involve a layered architecture with clear separation of concerns.

Firstly, let's understand what the question is asking:

Performance Requirement: Ensure that 99% of queries are answered in less than 10ms.
Scalability Requirement: Handle 200k QPS (queries per second) under normal load conditions.
Security and Governance: Maintain centrally managed policies with strict access controls, ensuring auditable decisions.
Step-by-Step Solution

Policy Layer:

The policy layer is responsible for managing all the policies that define what actions a user can perform on resources.
Policies will be stored in a highly scalable database like Cassandra or Amazon DynamoDB to ensure high availability and performance.

Decision Engine Layer:

This layer is where the actual evaluation of the policies happens. It should be optimized for speed but still maintain flexibility.
We can use a microservices architecture with each service responsible for specific types of policy evaluations (e.g., authorization, permissions).

Cache Layer:

Implement caching to reduce the load on the policy layer and improve response times.
Use Redis or Memcached as an in-memory cache to store frequently accessed policies and results.

Audit Logging Layer:

Ensure that every decision made by the system is logged for audit purposes.
Store audit logs in a central logging service such as ELK (Elasticsearch, Logstash, Kibana) or Splunk.

Centralized Management Interface:

Provide an interface to centrally manage and update policies.
This could be a web-based UI or API for administrators to make changes.
First-Principles Reasoning

High-Level Architecture Overview:

User Request: The user's request (U, A, R) is received by the API gateway.
API Gateway: Routes the request to the appropriate decision engine based on policy type.
Decision Engine: Evaluates the policies and returns a boolean result.
Cache Layer: Caches results for future requests.
Audit Logging: Logs every evaluation.

Performance Optimization:

Use in-memory caching to minimize database hits.
Implement distributed tracing to identify bottlenecks.
Optimize query performance using indexes and denormalization where necessary.

Scalability Considerations:

Design the system with microservices, each responsible for a specific type of policy evaluation.
Use load balancers to distribute traffic evenly across services.
Implement auto-scaling policies in cloud environments like AWS or Azure.

Security and Governance:

Ensure all communication between components is encrypted using TLS/SSL.
Apply role-based access control (RBAC) to manage permissions for administrators.
Regularly review and update policies to ensure they meet compliance requirements.

#### Closing Summary

By layering the architecture with a policy management system, decision engine, cache layer, audit logging, and centralized management interface, we can build an entitlement service that meets the performance, scalability, security, governance, and auditability requirements. The key is to maintain a clear separation of concerns and leverage modern technologies for efficient implementation.

#### Common Mistakes
and Follow-Ups
Overly Complex Policy Evaluation: Avoid creating overly complex policies that slow down evaluation.
Single Point of Failure: Ensure no single component can bring the entire system down by implementing redundancy and failover mechanisms.
Ignoring Audit Trails: Always ensure that all critical operations are logged for compliance and security audits.

**Comparison:**

```
User ID	Action	Resource
user1	read	resourceA
user2	write	resourceB
```

**Code:**

```python
# Example of a policy representation in Python
class Policy:
    def __init__(self, user_id, action, resource):
        self.user_id = user_id
        self.action = action
        self.resource = resource

# Example decision engine function
def evaluate_policy(policy, cache_client):
    # Check cache first
    result = cache_client.get(f"{policy.user_id}_{policy.action}_{policy.resource}")
    if result:
        return result

    # Evaluate policy from database (simplified)
    db_result = fetch_policy_from_db(policy)

    # Cache the result
    cache_client.set(f"{policy.user_id}_{policy.action}_{policy.resource}", db_result, ex=300)  # Cache for 5 minutes

    return db_result

# Example of a simple in-memory cache client
class SimpleCache:
    def __init__(self):
        self.cache = {}

    def get(self, key):
        return self.cache.get(key)

    def set(self, key, value, ex=None):
        self.cache[key] = (value, time.time() + ex) if ex else value

# Example of fetching policy from a database
def fetch_policy_from_db(policy):
    # Simulated database query
    policies = [
        Policy("user1", "read", "resourceA"),
        Policy("user2", "write", "resourceB")
    ]

    for p in policies:
        if p.user_id == policy.user_id and p.action == policy.action and p.resource == policy.resource:
            return True
    return False
```

**Example input / output:**

```text
Input:
- User ID: user1, Action: read, Resource: resourceA

Output:
- Result: True

Request (HTTP GET) -> Response (JSON):
{
  "user_id": "user1",
  "action": "read",
  "resource": "resourceA",
  "result": true
}
```

**Data:**

```json
{
  "policies": [
    {
      "user_id": "user1",
      "action": "read",
      "resource": "resourceA"
    },
    {
      "user_id": "user2",
      "action": "write",
      "resource": "resourceB"
    }
  ]
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[User Request] --> B(API Gateway);
4. B --> C(Evaluate Policy -> Cache Layer);
5. C --> D(Cache Hit) [True];
6. D --> E(Audit Logging);
7. C --> F(Database Query) [False];
8. F --> G(Evaluate Policy -> Cache Layer);
9. G --> H(Cache Set) (5m);
10. H --> I(Audit Logging);
11. I --> J[Response: True or False]

