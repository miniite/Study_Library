---
type: "practice"
difficulty: "Expert"
---

# Your gateway platform team is asked to support 'API mesh' where any team can compose other teams' APIs into new endpoints declaratively; design the composition/orchestration layer, its failure semantics, and how you stop it from becoming a distributed monolith of hidden dependencies.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Control Plane and Data Plane Design](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/Control_Plane_and_Data_Plane_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In an API mesh architecture, teams can compose other APIs into new endpoints declaratively, enabling dynamic and flexible service discovery and routing. The challenge lies in designing a robust composition/orchestration layer that ensures seamless inter-service communication while maintaining operational transparency and preventing the emergence of hidden dependencies that could lead to a distributed monolith.

#### Triage

The question is essentially asking for a detailed design of the orchestration layer in an API mesh, focusing on how different services are composed into new endpoints, failure semantics, and strategies to avoid creating an unwieldy system. The key points to address include:

Composition Mechanism: How teams can define and compose APIs.
Orchestration Layer: The core component that handles routing, service discovery, and policy enforcement.
Failure Semantics: What happens when a service fails or is slow.
Dependency Management: Ensuring clarity in the dependencies between services to avoid hidden monoliths.

#### First Principles Solution

Service Discovery

Implement a dynamic service registry that can be updated by any team as they add, remove, or update APIs.
Use a lightweight protocol like DNS or mDNS for seamless updates and minimal disruption during changes.

API Composition API

Develop an API composition language (e.g., YAML, JSON) where teams define how APIs should be combined into new endpoints.
This composition layer should support complex operations such as chaining requests, aggregating responses, and handling failures gracefully.

Orchestration Layer

The orchestration layer should handle routing, load balancing, retries, timeouts, and fallbacks based on the defined policies in the API composition API.
Leverage a service mesh like Istio or Linkerd for robust service discovery and traffic management, which can also provide features such as circuit breaking, retries, and rate limiting.

Failure Semantics

Define clear semantics for failures: how to handle errors, timeouts, and retries.
Implement a robust monitoring and logging system to track the behavior of the API mesh and identify issues early.
Use canary releases and A/B testing mechanisms to introduce new changes gradually without affecting the entire ecosystem.

Dependency Management

Ensure each team is aware of their dependencies by documenting them clearly in the composition API.
Use dependency graphs or tools that visualize service interactions, making it easier to spot hidden dependencies.
Implement a policy enforcement layer that prevents teams from accidentally creating complex and tightly coupled services.

#### Closing Summary

Designing an API mesh requires a careful balance between flexibility and manageability. By implementing dynamic service discovery, using a well-defined composition API, leveraging a robust orchestration layer like a service mesh, ensuring clear failure semantics, and managing dependencies effectively, we can create a resilient and maintainable system that supports the needs of modern microservices architectures.

#### Common Mistakes/Follow-Ups
Mistake 1: Over-reliance on a single point of failure in the composition layer. Ensure redundancy and failover mechanisms are in place.
Mistake 2: Lack of clear documentation for dependencies, leading to hidden monoliths. Regularly audit and document service interactions.

#### Follow-Up
Discuss how to handle scenarios where APIs have non-deterministic behavior or are stateful.

**Comparison:**

```
Service	URL	Port
CustomerService	http://customer-service.com	8080
OrderService	http://order-service.com	8081
```

**Code:**

```yaml
# Example API Composition YAML

api:
  name: "CustomerInfo"
  version: "1.0"
  services:
    - service: "CustomerService"
      path: "/customers/{id}"
      method: "GET"
    - service: "OrderService"
      path: "/orders/{customerId}"
      method: "GET"
  composition:
    - operation: "merge"
      paths: ["/customers/{id}/orders"]
```

**Example input / output:**

```text
Request:
GET /customers/123/orders

Response (assuming both services are available):
{
  "customerId": "123",
  "orders": [
    {
      "id": "456",
      "status": "shipped"
    },
    {
      "id": "789",
      "status": "cancelled"
    }
  ]
}
```

**Data:**

```json
{
  "services": {
    "CustomerService": {
      "url": "http://customer-service.com",
      "port": 8080
    },
    "OrderService": {
      "url": "http://order-service.com",
      "port": 8081
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. API Composition Definition
4. Orchestration Layer
5. Is Service Down?
6. Failover to Backup Service
7. Retransmit Request
8. Return Error
9. Continue Processing

