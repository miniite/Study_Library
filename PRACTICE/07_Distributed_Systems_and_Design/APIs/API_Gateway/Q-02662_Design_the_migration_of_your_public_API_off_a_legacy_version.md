---
type: "practice"
difficulty: "Deep"
---

# Design the migration of your public API off a legacy versioning scheme (URI v1/v2) to a header/content-negotiation scheme without breaking thousands of external consumers; how do you run both at the gateway, track migration progress, and set the sunset with partner comms?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Gateway Routing and BFF Patterns](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To migrate from a legacy URI versioning scheme (v1/v2) to a header/content-negotiation scheme without breaking external consumers, we need to design an API Gateway that can handle both versions gracefully. This involves running both schemes concurrently while tracking the migration progress and setting up a sunset strategy with clear communication to partners.

First, let's triage the question: The core challenge is maintaining backward compatibility as you transition from v1/v2 URIs to a header-based versioning scheme. We need to ensure that external consumers can continue using their existing endpoints without breaking anything. Additionally, we must plan for and communicate about the eventual sunset of the old versioning method.

#### Assumptions

The legacy API has thousands of active consumers.
The new versioning approach will use headers (e.g., X-API-Version) to determine which version a request should be routed to.
The migration process needs to be gradual and manageable, with clear communication channels for all partners.
Step-by-Step Solution

Gateway Design:

Implement the API Gateway to accept both URI v1/v2 and header-based requests.
Use a routing strategy that checks the request method, path, and headers to determine which version of the API should be invoked.
URI
Header
v1
v2
API Request
URI or Header?
Invoke Legacy v1/v2 API
Header Value?
Route to Legacy v1 API
Route to Legacy v2 API

Parallel Operation:

Ensure the gateway can handle both URI and header requests simultaneously.
Set up a transparent proxy that forwards incoming requests based on the versioning logic.

Tracking Migration Progress:

Use analytics tools within the gateway to track which consumers are using v1/v2 versus the new header-based scheme.
Implement health checks and usage metrics to monitor how many requests each version is handling.

Sunsetting Legacy Versions:

Plan a clear sunset date for removing support for URI-based versioning.
Communicate this plan to all partners, providing them with ample time to update their systems.
No
Yes
Gateway
Sunset Date?
Migrate Consumers to Header
Sunset Legacy API

Communication Strategy:

Send regular updates to all partners about the migration progress.
Provide clear documentation on how to update their systems to use the new header-based versioning scheme.

#### Closing Summary

By designing an API Gateway that can handle both URI and header-based requests, we ensure a smooth transition for external consumers. Tracking migration progress through analytics and setting up a clear sunset strategy with partner communication will help manage the process effectively. The key is to maintain backward compatibility while progressively moving towards a more modern versioning approach.

#### Common Mistakes/Follow-Ups
Breaking Existing Consumers: Failing to provide a clear transitional path can lead to service disruption for existing consumers.
Neglecting Analytics: Without proper tracking, it's hard to identify issues or assess the impact of the migration.
Insufficient Communication: Poor partner communication can result in unpreparedness and resistance during the sunset phase.

**Comparison:**

```
Metric	Value
Current Consumers	5000
New Version Users	300
Legacy Version Users	4700
```

**Code:**

```
MERMAID
Copy
graph LR;
     A[API Request] --> B{URI or Header?};
     B -- URI --> C[Invoke Legacy v1/v2 API];
     B -- Header --> D{Header Value?};
     D -- v1 --> E[Route to Legacy v1 API];
     D -- v2 --> F[Route to Legacy v2 API];
```

**Example input / output:**

```text
Input: `GET /users` with `X-API-Version: v2`

Output: Response from the legacy version 2 API.

Input: `GET /users` without any specific versioning header

Output: Response from the legacy version 1 API.
```

**Data:**

```yaml
api_gateway_config:
  routing_strategy: uri_or_header_based
  analytics_enabled: true
  sunset_date: 2023-12-31
```

**Flow:**

1. URI
2. Header
3. v1
4. v2
5. API Request
6. URI or Header?
7. Invoke Legacy v1/v2 API
8. Header Value?
9. Route to Legacy v1 API
10. Route to Legacy v2 API

