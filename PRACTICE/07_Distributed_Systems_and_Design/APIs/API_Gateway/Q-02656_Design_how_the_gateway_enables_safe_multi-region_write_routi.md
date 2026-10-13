---
type: "practice"
difficulty: "Expert"
---

# Design how the gateway enables safe multi-region write routing for a globally distributed datastore: how do you route writes to the right region, handle cross-region consistency for the client, and present a coherent latency and correctness story during regional failover?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Control Plane and Data Plane Design](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/Control_Plane_and_Data_Plane_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing an API Gateway to enable safe multi-region write routing for a globally distributed datastore, we need to address several critical aspects: ensuring writes go to the right region, maintaining cross-region consistency for clients, and delivering a robust latency and correctness story during regional failovers. The goal is to create a system that provides low-latency access while preserving data integrity.

#### Triaging the Question

The key challenges are:

Regional Routing: Ensuring writes are directed to the correct region.
Cross-Region Consistency: Maintaining consistency across regions, especially in scenarios like regional failovers.
Latency and Correctness: Providing a seamless experience for clients during regional outages while ensuring data correctness.

#### Assumptions
The API Gateway supports multiple regions (e.g., Region A, B, C).
Data is distributed across these regions to ensure low latency.
Clients may connect from any region but expect consistent access and write outcomes.

#### Solution
From First Principles
Step 1: Regional Routing

To route writes correctly:

Client Identification: The API Gateway must identify the client’s location or provide a method for clients to self-report their region (e.g., via headers).
Routing Strategy:
For non-critical data, allow writes to any region but ensure eventual consistency.
For critical data, use a consistent hashing mechanism to route writes based on the client's reported region.
Step 2: Cross-Region Consistency

Cross-region consistency can be achieved through various strategies:

Eventual Consistency: Use event sourcing or append-only logs to achieve eventual consistency. Clients are informed of write outcomes asynchronously.
Optimistic Concurrency Control (OCC): Implement OCC mechanisms within the API Gateway to detect and handle conflicts during writes.
Read Repair: Periodically read from other regions and update local copies if inconsistencies arise.
Step 3: Latency and Correctness During Regional Failover

During a regional failover, ensure:

Health Checks: Regularly perform health checks on services within each region to detect failures early.
Failover Mechanisms:
Use active-active or active-passive configurations where possible. For example, allow writes to multiple regions but prioritize local writes during outages.
Implement a failover strategy where the API Gateway automatically reroutes traffic to a healthy region.
Step 4: Client Experience and Error Handling
Transparent Failover: Ensure clients do not notice regional outages by maintaining consistent interfaces. Redirect or retry requests as needed.
Error Handling: Provide detailed error messages indicating if a write failed due to a transient issue, client-side error, or regional outage.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The API Gateway should be designed with robust multi-region support in mind, balancing low-latency access with data integrity. Key areas for optimization include efficient routing strategies, consistent cross-region handling, and seamless failover mechanisms. Common mistakes include overcomplicating the solution or neglecting client experience during outages.

#### Follow-up
questions might explore specific failure scenarios, trade-offs between consistency models (e.g., strong vs eventual), and how to manage write conflicts in a distributed system.

**Comparison:**

```
Strategy	Description
Eventual Consistency	Use event sourcing or logs for consistency.
OCC	Implement optimistic concurrency control for critical data.
Read Repair	Periodically check regions for inconsistencies.
```

**Code:**

```python
# Example code snippet for routing writes based on region
def route_write(region_header, data):
    client_region = get_client_region(region_header)
    if is_critical_data(data):
        # Use consistent hashing for critical data
        preferred_region = calculate_preferred_region(client_region)
        return preferred_region
    else:
        # Allow writes to any healthy region
        return health_check_regions()
```

**Example input / output:**

```text
Input:
- Client requests a write to the gateway with a 'X-Client-Region' header set to 'RegionA'.
- The client sends critical data.

Output:
- Gateway determines RegionB as the preferred region for writes.
- Write is routed to RegionB and logged in an append-only log.
```

**Data:**

```json
{
  "clientRegion": "RegionA",
  "data": {
    "type": "critical",
    "content": "important data"
  }
}
```

**Flow:**

1. Yes
2. No
3. Client Request
4. Is Critical Data?
5. Route to Preferred Region
6. Allow Writes to Any HealthyRegion
7. Write to Appropriate Regionand Log
8. Log Write for Recovery

