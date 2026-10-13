---
type: "practice"
difficulty: "Deep"
---

# Two senior teams are in a stalemate: one insists on gRPC internal APIs, the other on REST/JSON for developer ergonomics. As the deciding architect, how do you adjudicate this without becoming the bottleneck for every future protocol choice?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ API Composition and BFF](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Idempotent_Order_APIs/API_Composition_and_BFF.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To resolve the stalemate between using gRPC internal APIs and REST/JSON for developer ergonomics, we need to consider multiple aspects of our system design. The key is to balance performance, maintainability, and ease of development while ensuring that we don't become a bottleneck in future protocol choices.

The TL;DR is this: We should leverage both approaches by adopting a hybrid strategy where gRPC handles critical internal services for performance reasons, and REST/JSON is used for more user-facing APIs due to its broad support. However, this needs to be done carefully to avoid creating a single point of decision fatigue or technical debt.

#### Triage

The question essentially boils down to choosing between two protocols with different strengths and trade-offs: gRPC for performance and efficiency, and REST/JSON for ease of use and broader ecosystem support. The core issue is how to make this choice without creating a rigid architecture that could be limiting in the future.

#### Assumptions
Both teams have valid points.
We need to support both protocols simultaneously.
The decision should not overly complicate our architecture or development process.

#### First-Principles Solution

Critical Internal Services: gRPC is better suited for internal microservices due to its binary format, efficient serialization/deserialization, and built-in support for streaming and RPCs (Remote Procedure Calls). This reduces the overhead of network communication and improves overall performance.

User-Facing APIs: REST/JSON is more developer-friendly and widely supported by tools and frameworks. It’s easier for external clients to integrate and consumes less bandwidth due to its text-based nature.

Hybrid Approach: Design a modular system where internal services use gRPC and public-facing APIs use REST/JSON. This allows us to optimize our internal communications while providing an easy-to-use interface externally.

API Gateway: Introduce an API gateway that can handle requests from both internal services and external clients, abstracting the underlying protocol. The gateway should be intelligent enough to route traffic based on the request type and destination.

Documentation and Standardization: Establish clear guidelines for each team’s responsibilities and ensure consistent documentation across APIs. This helps in reducing misunderstandings and ensuring that changes are well-documented.

Monitoring and Feedback Loop: Set up monitoring tools to track performance metrics of both protocols and gather feedback from developers. Regular reviews can help identify pain points and make necessary adjustments.

Decoupling: Ensure that the choice of protocol does not tightly couple different parts of the system. For example, services should be designed to be agnostic about the communication protocol.

#### Closing Summary

By adopting a hybrid approach where critical internal services use gRPC and public-facing APIs use REST/JSON, we can balance performance and developer ergonomics effectively. The API gateway acts as a middle layer that handles protocol routing, providing flexibility and reducing overhead.

#### Common Mistakes/Follow-Ups

Becoming overly prescriptive about the choice of protocols without considering future needs.
Failing to provide clear guidelines for developers on how to use each protocol.
Not setting up monitoring mechanisms to track performance metrics and gather feedback.
Tight coupling between services due to rigid protocol choices.

**Comparison:**

```
Protocol	Internal Services	Public APIs
gRPC	Yes	No
REST/JSON	No	Yes
```

**Code:**

```python
# Example API Gateway Router Logic in Python
def route_request(request):
    if is_internal_service(request):
        return call_gRPC_service(request)
    else:
        return call_REST_service(request)

def is_internal_service(request):
    # Check based on request headers or metadata
    pass

def call_gRPC_service(request):
    # Call gRPC service logic here
    pass

def call_REST_service(request):
    # Call REST/JSON service logic here
    pass
```

**Example input / output:**

```text
Request to internal service:
- Input: HTTP POST request to /internal/service with JSON body.
- Output: gRPC response from the target microservice.

Request to public API:
- Input: HTTP GET request to https://example.com/api/v1/users.
- Output: JSON response {"id": 1, "name": "John Doe", "email": "john.doe@example.com"}
```

**Data:**

```json
{
  "internalService": {
    "protocol": "gRPC",
    "serviceEndpoints": ["http://internal-grpc-service:50051"]
  },
  "publicAPIs": {
    "protocol": "REST/JSON",
    "apiEndpoints": [
      "https://example.com/api/v1/users"
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. API Gateway
4. Is Request for Internal Service?
5. Call gRPC Service
6. Call REST/JSON Service
7. Return gRPC Response
8. Return JSON Response

