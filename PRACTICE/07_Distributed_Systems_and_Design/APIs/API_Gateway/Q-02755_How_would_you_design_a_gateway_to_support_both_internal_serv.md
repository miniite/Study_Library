---
type: "practice"
difficulty: "Deep"
---

# How would you design a gateway to support both internal service-to-service traffic and external client traffic; do you use one gateway or split them, and why?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Gateway Routing and BFF Patterns](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an API Gateway that supports both internal service-to-service (S2S) traffic and external client traffic, we need to consider several factors including security, performance, scalability, and maintenance. The core idea is to create a robust gateway architecture that can handle diverse traffic types while ensuring separation of concerns.

We start by understanding the question: The interviewer wants to know how you would design a single API Gateway or if it’s better to split them into separate gateways for internal and external traffic. The key points they are probing include:

Whether splitting the traffic improves security and performance.
How to manage different types of authentication and authorization mechanisms.
The impact on network latency and throughput.
The overall architecture and how components interact.

#### Assumptions
we can make include:

Both S2S and client traffic need secure access control.
Performance optimization is a concern for both traffic types.
We have a mix of services, some more critical than others.

Let's reason step-by-step from first principles:

Single Gateway Approach:

#### Pros
Simplifies the deployment process; easier to maintain and update a single component.

#### Cons
Increased complexity in managing different access controls; potential bottleneck if not well-architected.

Split Gateways:

#### Pros
Better security separation; potentially lower latency for internal traffic due to reduced hops; more scalable architecture by focusing specific optimizations on each type of traffic.

#### Cons
More complex deployment and maintenance, increased development cost.

Given these considerations, a hybrid approach might be the best solution. We can use a single API Gateway that internally routes S2S requests differently from client requests based on headers or other metadata. This way, we leverage the benefits of both worlds without incurring the full costs of maintaining two separate gateways.

Here's how this could work:

External Client Traffic: Use standard authentication mechanisms like OAuth2 for securing access. Implement rate limiting, logging, and caching to optimize performance.

Internal Service-to-Service Traffic: Route these requests through a different path or with custom headers. Use token-based authentication (like JWT) but bypass traditional client-side security checks. Optimize network latency by reducing the number of hops and using direct service-to-service calls where possible.

To implement this, we can use a gateway framework like Kong or NGINX that supports flexible routing based on metadata. Here’s an example configuration for NGINX:

NGINX
http {
    # Route external client traffic
    server {
        listen 80;
        location /external/ {
            # Add client authentication and security checks here
            proxy_pass http://client-backend;
        }
    }

    # Route internal service-to-service traffic
    upstream internal_services {
        server svc1.internal:3000;
        server svc2.internal:3000;
    }

    server {
        listen 8443 ssl; # Use HTTPS for S2S, but without client-side security checks
        location /internal/ {
            add_header X-Internal true;
            proxy_pass http://internal_services;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            # Bypass client-side authentication and other overheads for S2S traffic
        }
    }
}

In this configuration, external requests hit the default 80 port and undergo standard security checks. Internal service-to-service requests are routed to a specific path (e.g., /internal/) with custom headers that bypass most of the client-side authentication processes.

This approach ensures separation of concerns while maintaining simplicity in deployment and maintenance.

**Code:**

```
NGINX
Copy
http {
    # Route external client traffic
    server {
        listen 80;
        location /external/ {
            # Add client authentication and security checks here
            proxy_pass http://client-backend;
        }
    }

    # Route internal service-to-service traffic
    upstream internal_services {
        server svc1.internal:3000;
        server svc2.internal:3000;
    }

    server {
        listen 8443 ssl; # Use HTTPS for S2S, but without client-side security checks
        location /internal/ {
            add_header X-Internal true;
            proxy_pass http://internal_services;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            # Bypass client-side authentication and other overheads for S2S traffic
        }
    }
}
```

**Flow:**

1. Yes
2. No
3. Client Request
4. Is Internal S2S?
5. Routed Internally
6. Routed Externally

