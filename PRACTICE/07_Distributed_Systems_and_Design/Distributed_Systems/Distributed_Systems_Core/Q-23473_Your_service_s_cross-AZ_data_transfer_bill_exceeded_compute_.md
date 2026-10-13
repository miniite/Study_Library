---
type: "practice"
difficulty: "Deep"
---

# Your service's cross-AZ data transfer bill exceeded compute cost. Explain the design decisions that create this and how you'd add AZ-affinity routing without sacrificing the AZ-failure resilience you built.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cross-AZ Traffic Costs](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Cross-AZ_Traffic_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a distributed system, ensuring both performance and availability is paramount. However, when cross-AZ (Availability Zone) data transfer costs exceed compute costs, it often indicates suboptimal design decisions around network traffic management and resource placement. To address this issue while maintaining resilience, we need to revisit our current architecture and implement AZ-affinity routing strategically.

Triaging the Problem

The core of the problem is that cross-AZ transfers are more expensive than intra-AZ ones due to higher latency and bandwidth costs. This suggests that data is being transferred unnecessarily between zones, possibly because of how services are deployed or how requests are routed. Our goal is to minimize cross-AZ traffic while ensuring that our system remains resilient to AZ failures.

#### Assumptions
Current Architecture: Services are distributed across multiple AZs with high availability and fault tolerance.
Network Costs: Cross-AZ transfers have higher costs than intra-AZ ones, primarily due to longer latencies and bandwidth limitations.
Service Deployment: Applications are deployed in a way that they can failover between different AZs.

#### First-Principles Solution

To reduce cross-AZ data transfer while maintaining resilience, we need to implement AZ-affinity routing without compromising on fault tolerance. Here’s the step-by-step approach:

Understand Service Dependencies: Analyze which services are heavily reliant on each other and determine if they can coexist within a single AZ.
Implement Load Balancers with AZ-Awareness: Use load balancers that are aware of AZs to route traffic more efficiently. This means directing requests to the closest service replicas, thereby reducing cross-AZ data transfer.
Microservices Architecture: Ensure microservices are designed for auto-scaling and AZ-aware routing. Each microservice should be configured to handle its own AZ failures gracefully.
Network Policies and Firewalls: Implement network policies that restrict unnecessary cross-AZ traffic while allowing essential services to communicate within the same AZ.
Detailed Steps

Dependent Services Co-location: Identify critical services that need high availability but do not require cross-az routing. Group these services together in a single or specific set of AZs.

#### Example
If we have a database service and a frontend application, they can be deployed within the same AZ to reduce cross-AZ traffic.

Load Balancer Configuration: Set up load balancers that are AZ-aware and route requests based on the closest replica available.

Yes
No
User Request
Is Service in Same AZ?
Send Request to Local Replica
Routing Table Lookup
Select Closest Replica
Request Sent to ClosestReplica

Network Policies: Define network policies that ensure only necessary traffic can cross AZ boundaries.

Service A
Service B in Same AZ
Allow Traffic
Service D in Different AZ
Block Traffic unless Essential

Microservices Design: Ensure that each microservice is designed to be self-contained and can operate independently within its own AZ.

Use service discovery mechanisms like Consul or Eureka to manage instances within the same AZ.

#### Closing Summary

By implementing an AZ-aware load balancing strategy, co-locating dependent services in a single AZ, and defining network policies that restrict unnecessary cross-AZ traffic, we can significantly reduce data transfer costs while maintaining high availability. This approach requires careful planning and design but is essential for optimizing cloud spend and ensuring efficient resource utilization.

#### Common Mistakes/Follow-ups
Overzealous Co-location: Avoid co-locating services just to save on network costs if it reduces overall resilience.
Inadequate Network Policies: Not properly defining network policies can lead to security risks or unnecessary traffic.
Ignoring Edge Cases: Failing to consider scenarios where a single AZ might still fail, such as during maintenance windows.

**Comparison:**

```
Service	Co-location Zone	Routing Table
A	AZ 1	Local
B	AZ 2, AZ 3	AZ 1
C	AZ 4, AZ 5	Block Non-Essential Traffic
```

**Code:**

```python
# Example of AZ-aware load balancer configuration
def route_request(user_request):
    closest_replica = find_closest_replica(user_request.service)
    if closest_replica in same_az_instances:
        return send_request_to(closest_replica)
    else:
        routing_table = get_routing_table()
        next_hop = routing_table.get_next_hop_for_service(user_request.service, user_request.origin_az)
        return send_request_to(next_hop)

def find_closest_replica(service):
    # Logic to find closest replica in the same AZ
    pass

def send_request_to(replica):
    # Send request to specified replica
    pass
```

**Example input / output:**

```text
Input:
- User Request: service-a request from az-1

Output:
- Route to instance-1 in the same AZ (az-1)
```

**Data:**

```json
{
  "same_az_instances": ["instance-1", "instance-2"],
  "routing_table": {
    "service-a": { "az-1": "instance-1", "az-2": "instance-3" },
    "service-b": { "az-1": "instance-4", "az-2": "instance-5" }
  }
}
```

**Flow:**

1. Yes
2. No
3. User Request
4. Is Service in Same AZ?
5. Send Request to Local Replica
6. Routing Table Lookup
7. Select Closest Replica
8. Request Sent to ClosestReplica

