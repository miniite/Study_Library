---
type: "practice"
difficulty: "Deep"
---

# Explain sticky sessions at an L7 load balancer and what problems session affinity creates for autoscaling and rolling deploys.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ TCP and Connection Tracking](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/TCP_and_Connection_Tracking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Sticky sessions, also known as session persistence or persistence, are a technique used in load balancing where the same user's requests are consistently directed to the same backend server during their session. At an L7 (Layer 7) load balancer, which operates on application layer protocols like HTTP, this is typically achieved by inspecting and maintaining a unique identifier within each client’s request. The key idea is that it ensures stateful sessions remain consistent across multiple requests, but it can introduce complexities for autoscaling and rolling deploys.

#### Triage

The question asks us to explain sticky sessions in the context of an L7 load balancer and discuss their implications on autoscaling and rolling deploys. Specifically, we need to understand how session affinity is implemented, its benefits, and the challenges it poses when scaling or updating applications.

#### First-Principles Solution

#### Implementation

At an L7 load balancer, sticky sessions can be implemented through various mechanisms:

Cookie Insertion: The load balancer inserts a cookie into the client’s browser session to track which backend server should handle that user's requests.
URL Rewriting: The load balancer modifies the URLs in the HTTP request headers or responses to include a sticky session identifier.
Benefits

#### Cons
istent State: Ensures that all requests from a single user are handled by the same backend server, maintaining state across multiple requests.
User Experience: Reduces the chances of session loss and improves performance as users experience fewer redirects or re-authentications.
Challenges for Autoscaling
Fixed Allocation: With sticky sessions enabled, the load balancer directs all traffic from a user to a single backend server. This means that even if another server is idle, it will not be used to handle requests.
Resource Imbalance: The backend server associated with a sticky session can become overburdened while other servers remain underutilized.
Scalability Issues: As the number of users increases, scaling becomes more challenging because each user’s session is locked into one server.
Challenges for Rolling Deploys
Down Time and Confusion: When rolling deploys are performed, you risk disrupting active sessions if not managed carefully. For instance, if a backend server goes down during an update, requests to it will fail.
Complex Rollout Strategies: To mitigate issues, you might need to implement additional strategies like blue-green deployments or canary releases with careful handling of sticky sessions.
Session Drift Risk: During deploys, users may be temporarily directed to new servers, causing session drift and potential loss.

#### Closing Summary

Sticky sessions are a powerful feature in load balancing that ensures stateful sessions remain consistent across multiple requests. However, they introduce significant challenges when it comes to autoscaling and rolling deploys. Proper planning and mitigation strategies are necessary to balance the benefits of sticky sessions with the need for flexible, scalable systems.

#### Common Mistakes/Follow-Ups
Not Considering Autoscaling: Failing to account for the fact that sticky sessions can lead to resource imbalance.
Ignoring Rollout Risks: Overlooking the potential disruptions during deploys and not implementing proper strategies.
Failure Mode Analysis: Not considering scenarios where backend servers fail or need maintenance, leading to session loss.

**Comparison:**

```
Scenario	Impact on Autoscaling	Impact on Rolling Deploys
User Load Increase	Resources may be underutilized	Risk of disrupting active sessions during deploys
Backend Server Maintenance	Idle servers remain unused	Need for careful session handling during maintenance
```

**Code:**

```python
# Example of cookie-based sticky sessions in a load balancer configuration file
loadbalancer_config = {
    "method": "round_robin",  # Default round-robin balancing method
    "sticky_sessions": True,
    "cookie_name": "session_id",
}
```

**Example input / output:**

```text
Input:
User makes a request to the load balancer with no session cookie.
Output:
Load balancer assigns the user to backend server 'backend-01' and sets a session cookie.

User makes subsequent requests with the same session cookie.
Output:
Load balancer consistently routes all requests to 'backend-01'.
```

**Data:**

```json
{
    "user_id": 123456,
    "session_id": "abcde12345",
    "server_assigned": "backend-01"
}
```

**Flow:**

1. Yes
2. No
3. User makes initial request
4. Is sticky session enabled?
5. Load Balancer assigns serverand sets cookie
6. User makes subsequentrequests with same cookie
7. Load Balancer directs traffic tosame server
8. Session state remainsconsistent across multiplerequests
9. Round-robin load balancingwithout session affinity

