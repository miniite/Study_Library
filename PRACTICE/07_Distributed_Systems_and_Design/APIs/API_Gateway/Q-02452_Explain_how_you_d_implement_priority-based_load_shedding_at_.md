---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd implement priority-based load shedding at the gateway so that during overload you drop low-value traffic (health-check pollers, analytics) before checkout requests; how do you classify and tag priority?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Load Balancing and Zone Aware Routing](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Load_Balancing_and_Zone_Aware_Routing/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement priority-based load shedding at an API Gateway during overload conditions, we need to ensure that high-priority traffic such as checkout requests is given precedence over lower-value traffic like health-check pollers or analytics. This involves classifying and tagging different types of traffic with appropriate priority levels, configuring the gateway to monitor system health and trigger shedding based on load metrics, and implementing a dynamic routing strategy.

#### Triage
What's Being Probed

The question is probing how we can manage resource allocation under overload conditions by dynamically adjusting which requests are processed. It requires an understanding of load balancing mechanisms, traffic classification techniques, and the ability to design a system that can scale without sacrificing critical operations.

#### Assumptions
Load Metrics: The gateway should have access to real-time or near-real-time metrics on incoming traffic and resource usage.
Traffic Classification: We need a mechanism to classify and tag different types of requests based on their importance.
Gateway Capabilities: The API Gateway supports dynamic routing, health checks, and can manage multiple backend services with varying load shedding priorities.

#### First Principles Solution

Classify and Tag Traffic:

Define priority levels (e.g., high, medium, low).
Use metadata headers or annotations in the request to tag traffic.

Monitor System Health:

Implement monitoring for critical services such as checkout requests.
Set thresholds for load and trigger shedding when thresholds are breached.

Dynamic Routing:

Route high-priority traffic first, then medium, and finally low-priority traffic.
Drop lower-value traffic if the gateway detects an overload condition.
Step-by-Step Implementation

Define Priority Levels:

High: Critical operations like checkout requests.
Medium: Non-critical but still important operations like user account updates.
Low: Lower-value operations like health-check pollers or analytics.

Tagging Requests:

Add metadata headers in the request to indicate its priority level (e.g., X-Priority with values "high", "medium", "low").
For example, a checkout request might have X-Priority: high.

Monitoring and Health Checks:

Use Prometheus or another monitoring system to track incoming traffic and backend service health.
Set up alerts for when the load exceeds predefined thresholds.

Dynamic Routing Configuration:

Configure the API Gateway to route requests based on priority levels.
When an overload condition is detected, drop lower-priority requests first.

#### Example
Implementation
Health Check Poller
Low Priority
Analytics Request
Low Priority
Checkout Request
High Priority
Load Shedding
Processed

#### Closing Summary
+ Common Mistakes/Follow-Ups
Summary: By implementing a multi-tiered priority system, we ensure that critical operations are not impacted during overload conditions. The key steps involve tagging traffic, monitoring load, and configuring the gateway to route requests based on their importance.

#### Common Mistakes
Not properly classifying traffic leading to incorrect prioritization.
Failing to implement real-time health checks which can lead to delayed or ineffective shedding decisions.
Overlooking edge cases where dynamic routing might inadvertently fail.

#### Follow-Ups
How would you ensure the system remains resilient under extreme conditions?
What are some best practices for tagging and monitoring different types of requests?
Can you discuss any trade-offs involved in this approach?

**Comparison:**

| Request  | Method | URL         | Priority |
|----------|--------|-------------|----------|
| Checkout | POST   | /api/checkout | high     |
| Ping     | GET    | /api/ping   | low      |
| Analytics| GET    | /api/analytics | low    |

**Code:**

```python
# Example code snippet to classify traffic using headers
def classify_request(request):
    priority = request.headers.get('X-Priority')
    if not priority:
        return 'low'
    if priority == 'high':
        return 'high'
    elif priority == 'medium':
        return 'medium'
    else:
        return 'low'

# Example code snippet to handle load shedding
def route_request(request, current_load):
    priority = classify_request(request)
    if current_load > threshold and priority == 'low':
        # Drop low-priority request
        pass
    else:
        process_request(request)
```

**Example input / output:**

```text
Input:
- High-priority requests: /api/checkout (POST) with X-Priority: high
- Low-priority requests: /api/ping (GET), /api/analytics (GET) both with X-Priority: low

Output:
- High-priority request processed first.
- Low-priority requests are dropped if system is overloaded.
```

**Data:**

```json
{
  "requests": [
    { "url": "/api/checkout", "method": "POST", "headers": { "X-Priority": "high" } },
    { "url": "/api/ping", "method": "GET", "headers": { "X-Priority": "low" } },
    { "url": "/api/analytics", "method": "GET", "headers": { "X-Priority": "low" } }
  ]
}
```

**Flow:**

1. Health Check Poller
2. Low Priority
3. Analytics Request
4. Low Priority
5. Checkout Request
6. High Priority
7. Processed
8. Load Shedding

