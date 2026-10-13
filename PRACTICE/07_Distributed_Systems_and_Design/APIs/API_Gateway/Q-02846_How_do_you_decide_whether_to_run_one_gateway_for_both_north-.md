---
type: "practice"
difficulty: "Intermediate"
---

# How do you decide whether to run one gateway for both north-south external traffic and internal service-to-service, or split them; what operational and security reasons push you one way in a Kubernetes shop?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Gateway Routing and BFF Patterns](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between running a single API Gateway for handling both north-south (external) and east-west (internal service-to-service) traffic, or splitting them into separate gateways, there are several operational and security considerations to weigh. The decision hinges on factors such as the complexity of your services, the need for fine-grained security controls, and the overall maintainability of your infrastructure.

#### Triaging the Question

The core of this question is understanding whether a monolithic API Gateway approach or a split architecture aligns better with operational needs in a Kubernetes environment. The interviewer is probing into your ability to evaluate trade-offs between simplicity versus security and performance.

#### Assumptions
Kubernetes Environment: We assume that the services are containerized, potentially using Kubernetes for orchestration.
Security and Compliance Requirements: There might be specific compliance or security standards that need to be adhered to.
Operational Complexity: The size and complexity of your service mesh play a significant role.

#### First-Principles Solution
Single API Gateway Approach

A single API Gateway can simplify management by reducing the number of gateways to maintain and update. However, it may introduce several challenges:

Security: A single entry point might be vulnerable if not properly secured.
Complexity: Routing all traffic through a single gateway could become complex with numerous services.
Performance: High traffic volumes could lead to bottlenecks.
Split API Gateway Approach

Separating north-south and east-west traffic can provide better isolation, security, and performance:

Security: North-south traffic can be more tightly controlled, reducing the risk of external threats compromising internal services.
Performance: East-west traffic can be optimized independently for different service-to-service interactions.
Operational Simplicity: Easier to manage and scale each gateway separately.
Detailed Reasoning

Security Considerations:

External Threats: North-south traffic is exposed to external threats, which may require more stringent security measures (e.g., rate limiting, authentication). A split approach can ensure that these controls are applied specifically to the external interface.
Internal Service Security: East-west traffic does not face direct external threats but can still be vulnerable. A separate gateway allows for different security policies tailored to internal interactions.

Operational Simplicity:

Single Gateway Management: If services are relatively simple and well-defined, a single gateway might suffice. However, as the number of services grows, so does the complexity.
Separate Gateways: Each gateway can be managed independently, making it easier to update or scale specific parts of the infrastructure.

Performance:

Latency: A split approach can optimize routing for internal services, potentially reducing latency between them.
Throughput: Managing traffic separately allows for better load balancing and scaling strategies.

#### Example Scenario

#### Cons
ider a microservices architecture with both external APIs (north-south) and internal service interactions (east-west):

North-South Gateway:
YAML
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: external-gateway
spec:
  rules:
    - host: api.example.com
      http:
        paths:
          - path: /v1/*
            pathType: Prefix
            backend:
              service:
                name: external-api-service
                port:
                  number: 8080
East-West Gateway:
YAML
apiVersion: networking.k8s.io/v1
kind: ServiceMesh
metadata:
  name: internal-gateway
spec:
  services:
    - service: internal-api-service
      port: 9000

#### Closing Summary
and Common Mistakes

The decision to run a single or split API Gateway depends on the specific needs of your application. A single gateway is simpler but may introduce security risks, especially with increased complexity. Splitting the gateways provides better security and performance, but at the cost of additional management overhead.

#### Common Mistakes

Overlooking the importance of security for north-south traffic.
Underestimating the operational complexity introduced by multiple gateways.
Failing to consider the scalability implications of a single gateway in a growing service mesh.

**Comparison:**

```
Gateway Type	Security	Complexity	Performance
Single	Less	Lower	Higher
Split	More	Higher	Better
```

**Code:**

```yaml
# Example Ingress for North-South Gateway
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: external-gateway
spec:
  rules:
    - host: api.example.com
      http:
        paths:
          - path: /v1/*
            pathType: Prefix
            backend:
              service:
                name: external-api-service
                port:
                  number: 8080

# Example ServiceMesh for East-West Gateway
apiVersion: networking.k8s.io/v1
kind: ServiceMesh
metadata:
  name: internal-gateway
spec:
  services:
    - service: internal-api-service
      port: 9000
```

**Example input / output:**

```text
Input:
- External API call to /v1/resource
- Internal service-to-service call from ServiceA to ServiceB

Output:
- North-South Gateway handles the external request and routes it to the appropriate service.
- East-West Gateway manages the internal call, possibly with more relaxed security policies.

Dry-run Request:
GET /v1/resource

Response:
{
  "message": "Resource fetched successfully",
  "status_code": 200
}
```

**Data:**

```json
{
  "gatewayConfig": {
    "external": [
      { "host": "api.example.com", "paths": ["/v1/*"] }
    ],
    "internal": [
      { "service": "internal-api-service", "port": 9000 }
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. External API Call
4. Split Gateway?
5. East-West Gateway
6. North-South Gateway
7. Internal Service Calls
8. External Requests

