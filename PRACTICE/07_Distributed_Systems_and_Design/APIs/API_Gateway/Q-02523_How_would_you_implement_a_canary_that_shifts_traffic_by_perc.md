---
type: "practice"
difficulty: "Deep"
---

# How would you implement a canary that shifts traffic by percentage AND by cohort (e.g., 5% + all internal employees) at the gateway, and how do you make the routing decision sticky per user across the canary window?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Load Balancing and Zone Aware Routing](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Load_Balancing_and_Zone_Aware_Routing/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement a canary that shifts traffic by percentage AND by cohort (e.g., 5% + all internal employees) at the gateway, and to ensure that the routing decision is sticky per user across the canary window, we need to design a robust system that handles both load balancing and session affinity. This involves setting up an API Gateway that intelligently routes requests based on predefined rules while ensuring consistency in user sessions.

At a high level, our solution will involve:

Cohort Identification: Determining which users belong to the internal employee cohort.
Load Balancing Rules: Configuring the gateway to route traffic based on both percentage and specific cohorts.
Session Affinity (Sticky Sessions): Ensuring that a user is consistently routed to the same backend service instance for the duration of their canary window.

Let's break down each component in detail:

#### Triage

The question asks us to implement a canary deployment strategy using an API Gateway, where traffic can be shifted by both percentage and specific cohorts. The key challenges are:

Percentage-based Routing: Implementing a load balancer that routes 5% of the traffic to the canary backend.
Cohort-Based Routing: Ensuring all internal employees' requests go directly to the canary.
Sticky Sessions: Maintaining session affinity so that users receive consistent responses from the same backend service instance during their canary window.

#### Assumptions
The API Gateway supports custom routing rules and load balancing mechanisms.
Users are authenticated via a token or similar mechanism, allowing identification of internal employees.
Backend services are stateless to simplify implementation.

#### First-principles Solution
Cohort Identification

To identify internal employees, we can leverage an authentication mechanism that tags users with specific roles. For example, internal employees might have a role: "internal_employee" in their token. This information will be used to route requests appropriately.

Yes
No
User Request
Is User Internal Employee?
Route to Canary Backend
Randomly Route 5% to CanaryBackend, Otherwise Route toProduction Backend
Load Balancing Rules

For percentage-based routing (5%), we can use a weighted round-robin load balancing algorithm. This ensures that a fixed percentage of incoming requests are directed to the canary backend.

Load Balancer
Weighted Round-Robin
Canary Backend
Production Backend 95%
Sticky Sessions

To implement sticky sessions, we use a session manager that keeps track of user sessions and routes requests to the same backend service instance based on this information. The session manager can be implemented as part of our API Gateway or integrated with an existing load balancer.

No
Yes
User Request
Session Manager
Lookup User Session ID
Is Session Active?
Create New Session, Route toCanary Backend (orProduction)
Route to Same ServiceInstance as Last Time

#### Closing Summary
and Common Mistakes

The key components of the solution are:

Cohort identification: Tagging users based on roles.
Load balancing rules: Implementing a percentage-based routing strategy.
Sticky sessions: Ensuring consistent user sessions across multiple requests.

#### Common mistakes
include:

Overlooking session management, leading to inconsistent routing decisions.
Failing to properly tag and identify cohorts, resulting in incorrect routing.
Misconfiguring load balancers, causing traffic distribution errors.

#### Follow-ups
How would you handle user roles if they change during the canary window?
What happens if a backend service fails? How do you ensure failover?
Can you provide an example of how to configure this setup in [specific API Gateway]?

**Comparison:**

```
User ID	Service Instance
12345	canary-service-01
67890	production-service-01
```

**Code:**

```python
# Example Python code for session management (simplified)
class SessionManager:
    def __init__(self):
        self.sessions = {}

    def create_session(self, user_id, service_instance):
        self.sessions[user_id] = service_instance

    def get_service_instance_for_user(self, user_id):
        return self.sessions.get(user_id, None)

# Example of how to use the session manager
session_manager = SessionManager()
user_id = "12345"
service_instance = "canary-service-01"  # Assume this is assigned by load balancer
session_manager.create_session(user_id, service_instance)
```

**Example input / output:**

```

Example input and output for session management:

Input:

HTTP
Copy
GET /api/v1/data HTTP/1.1
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

Output:

HTTP
Copy
HTTP/1.1 200 OK
Content-Type: application/json

{
    "data": "Some data from canary service"
}
```

**Data:**

```json
{
    "session_data": {
        "12345": "canary-service-01",
        "67890": "production-service-01"
    }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User Request
6. Is User Internal Employee?
7. Route to Canary Backend
8. Randomly Route 5% to CanaryBackend, Otherwise Route toProduction Backend
9. 95% to Production Backend
10. Lookup User Session ID inSession Manager
11. Is Session Active?
12. Route to Same ServiceInstance as Last Time
13. Create New Session, Route toCanary Backend or ProductionBased on Rules

