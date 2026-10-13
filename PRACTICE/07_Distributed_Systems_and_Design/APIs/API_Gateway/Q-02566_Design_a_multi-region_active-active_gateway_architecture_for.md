---
type: "practice"
difficulty: "Expert"
---

# Design a multi-region active-active gateway architecture for a global payments platform with strict data residency; how do you handle regional config divergence, a single control plane vs regional control planes, and split-brain during a region partition?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Control Plane and Data Plane Design](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/Control_Plane_and_Data_Plane_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing an active-active multi-region API gateway architecture for a global payments platform with strict data residency requirements, we need to ensure seamless cross-regional operations while handling regional configuration divergence and maintaining high availability even in case of region partitions. The key challenges are managing the control plane consistency, ensuring data residency compliance, and preventing split-brain scenarios.

Firstly, let's break down the core requirements:

Strict Data Residency: Ensure that data from specific regions can only be processed within those regions.
Regional Configuration Divergence: Different regions may have different configurations due to regulatory or operational needs.
Single Control Plane vs Regional Control Planes: Decide whether a centralized control plane is more suitable or if regional control planes are needed for better resilience and compliance.
Split-Brain Prevention: Ensure that the system does not enter into an inconsistent state during region partitions.

To address these challenges, we can design the architecture as follows:

High-Level Design

The architecture will consist of multiple regions each with its own API gateway and backend services. A centralized control plane will manage global policies while regional control planes handle specific configurations.

Regional Components:
API Gateway: Acts as an entry point for client requests.
Backend Services: Handle business logic and data processing.
Database: Stores region-specific data.
Control Plane Design

We can have two options here: a single centralized control plane or multiple regional control planes. A hybrid approach is often the best balance between complexity and resilience.

Single Centralized Control Plane:
Global Policies: Define global policies in one central location.
Regional Configurations: Store region-specific configurations as overlays on top of the global policies.
Data Residency Compliance: Enforce data residency by checking against regional metadata before allowing a request to proceed.
Regional Control Planes
Each region has its own control plane that enforces regional policies and configurations.
The central control plane acts as an authoritative source, but each regional control plane can cache and override global policies locally if necessary.
Handling Data Residency

To ensure data residency, we need to implement the following steps:

Metadata Storage: Store metadata about which regions own specific data sets in a distributed database.
Request Validation: Validate all incoming requests against the metadata to ensure they are routed to the correct region’s API gateway.
Regional Configuration Divergence

To handle configuration divergence, we can use a combination of declarative policies and dynamic overrides:

Declarative Policies: Define global policies that act as a baseline.
Dynamic Overrides: Allow regional configurations to dynamically override certain aspects of these policies while ensuring they do not violate strict data residency rules.
Split-Brain Prevention

To prevent split-brain scenarios, we can implement the following mechanisms:

Eventual Consistency with Strong Leaders: Use distributed consensus algorithms like Raft or Paxos to ensure that one region acts as a leader during partitions.
Quorum-Based Routing: Implement quorum-based routing where requests require confirmation from multiple regions before being processed.

#### Example
Architecture
Yes
No
Global Control Plane
Is Region Online?
Regional API Gateway
Fallback to Global API Gateway
Backend Services
Handle Requests viaCentralized API Gateway
Local Database
Summary and Common Mistakes
Common Mistake 1: Ignoring data residency requirements, leading to non-compliance.
Common Mistake 2: Over-complicating the control plane design, making it harder to maintain and debug.
Common Mistake 3: Failing to implement robust split-brain prevention mechanisms, which can lead to inconsistent states.

By carefully designing a multi-region architecture that balances global policies with regional configurations, we ensure that the system is both resilient and compliant with strict data residency requirements.

**Comparison:**

```
Region	Allowed Data Types
us-west-1	Payments, Account Statements
eu-central-1	Payments
ap-southeast-2	Account Statements
```

**Code:**

```python
# Example of a simple request validation function in Python
def validate_request(request, metadata):
    region = request.metadata.region
    allowed_regions = metadata.allowed_regions[request.data.type]

    if region not in allowed_regions:
        raise ValueError("Request does not comply with data residency rules")

validate_request(request, metadata)
```

**Example input / output:**

```text
Input:
Request Type: Payment
Region: us-west-1

Output:
Validation Success: True (Region is allowed to handle this type of request)
```

**Data:**

```json
{
  "allowed_regions": {
    "payments": ["us-west-1", "eu-central-1"],
    "accounting": ["ap-southeast-2"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Client Request
4. Is Region Online?
5. Regional API Gateway
6. Fallback to Centralized APIGateway
7. Backend Services
8. Handle via Centralized APIGateway

