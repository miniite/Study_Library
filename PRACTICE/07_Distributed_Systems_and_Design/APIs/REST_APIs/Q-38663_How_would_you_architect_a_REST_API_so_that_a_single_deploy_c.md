---
type: "practice"
difficulty: "Deep"
---

# How would you architect a REST API so that a single deploy can serve two contract versions simultaneously and route by header, and how do you retire the old version once traffic drains?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ API Versioning and Deprecation](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/API_Versioning_and_Deprecation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a REST API where a single deployment can serve two contract versions simultaneously based on a request header, we need to ensure that the system is flexible enough to handle both contract versions seamlessly and gracefully deprecate the older version when traffic diminishes. This involves understanding the routing logic, versioning strategy, and fallback mechanisms.

#### Triage

The key aspects of this question are:

Routing Strategy: How to route requests based on a header to serve either the old or new version.
Versioning: Implementing a mechanism to support multiple versions of an API simultaneously.
Traffic Drain Management: Gradually shifting traffic from the older version to the newer one and eventually retiring it.

#### Assumptions
The application uses HTTP headers to specify which contract version to use (e.g., X-Version: v1 for old, X-Version: v2 for new).
The API is stateless and can handle different versions concurrently.
There are monitoring tools in place to track request volumes.

#### First-Principles Solution
Versioning Strategy

We will use a combination of URL paths and header-based routing. For example, both /api/v1/endpoint and /api/v2/endpoint could exist but only one might be active at any given time based on the headers.

Routing Logic

Implement a middleware that intercepts requests and checks the X-Version header to determine which service version should handle the request. If no valid header is present, default to handling with the old version (v1).

Service Discovery and Load Balancing

Use a load balancer that supports header-based routing. This allows us to route traffic to different versions of services based on the X-Version header.

Backend Services
Old Version (v1): Serve requests using the legacy code.
New Version (v2): Implement new features and functionalities, serving requests with updated logic.

#### Implementation
Steps
Define Contract Versions: Clearly define what changes are introduced in each version to avoid breaking existing integrations.
Update Middleware: Add a middleware layer that inspects the X-Version header and forwards the request appropriately.
Configure Load Balancer: Set up a load balancer with a configuration file that routes requests based on the specified header.
Monitor Traffic: Use monitoring tools to track traffic distribution between versions.
Gradual Retirement of Old Version: Slowly increase traffic to the new version while decreasing it for the old version until all traffic is directed towards v2.

#### Example
Middleware
PYTHON
def route_request(request):
    # Default to v1 if no header is present
    version = request.headers.get('X-Version', 'v1')

    if version == 'v1':
        return v1_service.handle_request(request)
    elif version == 'v2':
        return v2_service.handle_request(request)
    else:
        raise ValueError("Unsupported version")

#### Closing Summary

By implementing a header-based routing strategy and using middleware to handle request redirection, we can ensure that both versions of the API coexist during deployment. Gradual traffic migration allows us to safely phase out older versions while ensuring smooth transitions for all clients.

#### Common Mistakes/Follow-Ups
Incorrect Header Handling: Failing to properly inspect or configure headers might result in incorrect routing.
Sudden Service Downtime: Abruptly shutting down the old service without a proper drain period can cause downtime.
Client Compatibility Issues: Ensuring that all clients are updated to support the new version is crucial.

**Code:**

```python
def route_request(request):
    # Default to v1 if no header is present
    version = request.headers.get('X-Version', 'v1')

    if version == 'v1':
        return v1_service.handle_request(request)
    elif version == 'v2':
        return v2_service.handle_request(request)
    else:
        raise ValueError("Unsupported version")
```

**Flow:**

1. v1
2. v2
3. unsupported
4. Receive Request
5. Check X-Version
6. V1 Service
7. V2 Service
8. Raise ValueError
9. Serve V1 Request
10. Serve V2 Request

