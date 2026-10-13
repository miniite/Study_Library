---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd design an org-wide approach to API versioning and backward compatibility in a cloud microservices estate, so that hundreds of services can evolve independently without a coordinated flag-day upgrade ever being required.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Data Lifecycle and Retention](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Stateless_by_Default/Data_Lifecycle_and_Retention.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an organization-wide approach to API versioning and backward compatibility for a cloud-based microservices estate with hundreds of services, we need to focus on maintaining flexibility while ensuring that changes in one service do not break other services relying on older versions. The key is to adopt a strategy that supports gradual evolution and minimizes disruption during the transition period.

The core idea is to leverage API gateways and version control strategies that allow for seamless transitions between different API versions, thereby enabling independent evolution of microservices without requiring a coordinated flag-day upgrade. This involves implementing API versioning techniques such as path-based or query parameter-based methods, while also considering semantic versioning principles to manage breaking changes.

**Comparison:**

```
Versioning Strategy	Description
Path-Based	/v1/resource for v1, /v2/resource for v2. Ensures clear separation between versions but requires URL changes in clients.
Query Parameter	https://api.example.com/resource?version=v2. Flexible but can lead to complex URLs and potential client-side implementation issues.
```

**Code:**

```
MERMAID
Copy
graph LR;
    A[API Gateway] --> B{Is Request for Version 1?}
    B -- Yes --> C[Route to Service V1]
    B -- No --> D{Is Versioned Endpoint Supported?}
    D -- Yes --> E[Route to Service V2]
    D -- No --> F[Error: Unsupported Version]
```

**Example input / output:**

```text
- **Input:** A request comes in with the URL `https://api.example.com/v2/resource`.
- **Output:** The API gateway routes the request to the version 2 service (`example-v2-service`).
```

**Data:**

```json
{
  "apiVersion": "v2",
  "serviceEndpoints": {
    "v1": "http://example-v1-service",
    "v2": "http://example-v2-service"
  }
}
```

**Flow:**

1. Yes
2. No
3. Client
4. Make API Request
5. API Gateway
6. Is Version Specified in URL?
7. Route to Correct Service
8. Use Default or Latest Version
9. Service v1/2

