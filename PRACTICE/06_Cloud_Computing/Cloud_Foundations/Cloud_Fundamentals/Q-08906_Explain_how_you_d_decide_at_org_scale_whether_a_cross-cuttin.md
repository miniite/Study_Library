---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd decide, at org scale, whether a cross-cutting concern (auth, rate limiting, tracing) lives in a shared library, a sidecar/mesh, or a gateway, given your language diversity.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cloud Network Topology](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Stateless_by_Default/Cloud_Network_Topology.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When making decisions about where to place cross-cutting concerns like authentication, rate limiting, and observability at an organizational scale with diverse languages, we need to consider several factors such as consistency, performance, security, and maintainability. At a high level, the key choices are whether these concerns live in a shared library, a sidecar/mesh (like a service mesh), or a gateway.

The core of this decision revolves around where you want to centralize your cross-cutting logic and how that impacts different aspects of the system architecture. Let’s break down the considerations step by step from first principles:

Shared Library: A shared library is ideal for consistent and reusable code across multiple services, especially when all services are written in the same language.
Sidecar/Mesh (Service Mesh): This approach centralizes these concerns within a network-level component that can dynamically intercept and manipulate traffic between services.
Gateway: Gateways act as the entry point to an API gateway, handling requests before they reach your services.

Now let’s explore each option in detail:

#### Triaging the Question

The question is probing how you would architect these cross-cutting concerns across a diverse language landscape while ensuring consistency and performance. We need to make informed decisions based on our current tech stack, organizational goals, and future scaling requirements.

#### Assumptions
The organization has services written in multiple languages (e.g., Python, Java, Go).
There is a need for centralizing authentication, rate limiting, and observability.

#### Cons
istency and maintainability are important considerations.
Step-by-Step Solution
1. Shared Library

#### Pros
Code reuse: Common logic can be shared across services.
Simplifies maintenance as you only need to update the library in one place.

#### Cons
Language dependency: If a service is written in a different language, it may not support or require the same dependencies.
Performance overhead: The shared library needs to communicate with each service, potentially adding latency.
2. Sidecar/Mesh

#### Pros
Decentralized: Each service can use its own library while still benefiting from centralized logic through a sidecar.
Flexibility: Each service can be written in the language of choice, and they can dynamically handle cross-cutting concerns.
Performance: No additional latency as the mesh handles interception at the network level.

#### Cons
Complexity: Adding a new component (sidecar) introduces more complexity to the system.
Overhead: Additional resources are required for each sidecar.
3. Gateway

#### Pros
Centralization: All requests go through one entry point, simplifying authentication and rate limiting.
Simplified Service Layer: Services can focus on business logic without handling these concerns.
Scalability: Easier to scale the gateway independently of services.

#### Cons
Single Point of Failure: A single gateway can become a bottleneck or a single point of failure.
Performance Overhead: Request processing in the gateway adds latency.
Evaluating the Options

Given our assumptions and options, we need to weigh these factors against each other:

For Authentication: Shared libraries are suitable if all services use the same language. For diverse languages, a sidecar/mesh approach can manage differences in authentication mechanisms while maintaining consistency.
For Rate Limiting: A gateway can be effective for global rate limiting policies, but individual service rate limits may require a sidecar/mesh setup to handle per-service configurations dynamically.
For Observability (Tracing): A service mesh is highly suitable as it can intercept requests and inject tracing information without modifying the services. Gateways can also perform this role.

#### Closing Summary

In summary, the choice between shared libraries, sidecars/meshes, or gateways depends on the specific requirements of each cross-cutting concern, the diversity of languages in your tech stack, and the overall architecture goals. A mixed approach might be optimal—using a shared library for common authentication logic, a service mesh for rate limiting and tracing, and possibly a gateway for global observability.

#### Common Mistakes/Follow-ups
Mistake: Ignoring the language diversity: Failing to consider that different languages may not support or require the same libraries.

#### Follow-up
Discussing how to ensure consistency across multiple gateways if you decide on a gateway approach.

#### Example Scenario
s

Let’s consider an example where we have services written in Python and Java. We would use:

A shared library for common authentication logic that both languages can utilize.
A service mesh like Istio to handle rate limiting and tracing, ensuring per-service flexibility.
An API Gateway (e.g., Kong) to manage global observability needs.

**Comparison:**

```
Cross-cutting Concern	Shared Library	Sidecar/Mesh	Gateway
Authentication	✅ (Lang Dep)	✅	✅
Rate Limiting	❌	✅	✅
Tracing	❌	✅	✅
```

**Code:**

```
MERMAID
Copy
graph TD
    subgraph ServiceMesh
        ServiceMesh --> SharedLibrary[Shared Authentication Library]
        ServiceMesh --> RateLimiter[Rate Limiter]
        ServiceMesh --> Tracer[Tracer]
    end

    subgraph Gateway
        Gateway --> GlobalAuth[Global Authentication Policies]
    end
```

**Example input / output:**

| Input (Request) | Output (Response) |
|-----------------|-------------------|
| GET /users      | OK, Authenticated with JWT |
| POST /login     | OK, Token generated and stored in header |

**Data:**

```json
{
  "service_mesh": {
    "auth_library": "shared-auth-library",
    "rate_limiter": "istio-rate-limiter",
    "tracer": "opencensus-tracer"
  },
  "api_gateway": {
    "global_auth_policies": ["basic_auth", "jwt"]
  }
}
```

**Flow:**

1. Yes
2. Pass
3. User Request
4. API Gateway
5. Is Gateway Enabled?
6. Request Authentication viaJWT
7. Proxy to Service Mesh
8. Service Mesh Route Table
9. Rate Limit Check
10. Tracing and Logging
11. Service Call
12. Service Response
13. Gateway Response

