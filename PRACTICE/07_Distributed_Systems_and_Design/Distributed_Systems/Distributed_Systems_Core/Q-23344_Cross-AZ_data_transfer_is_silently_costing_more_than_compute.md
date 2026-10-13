---
type: "practice"
difficulty: "Expert"
---

# Cross-AZ data transfer is silently costing more than compute in your architecture. Explain the design decisions that create this bill (chatty services split across AZs, replication topology, misplaced caches), and redesign for AZ-affinity without sacrificing the AZ-failure resilience you built it for.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-AZ Traffic Costs](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Cross-AZ_Traffic_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of cross-AZ data transfer costs exceeding compute costs in a distributed system architecture, we need to revisit several key design decisions and their impacts. Specifically, the chatty services split across AZs, replication topology, and misplaced caches all contribute to higher cross-AZ data transfer bills. The goal is to redesign for AZ-affinity while maintaining the desired resilience.

At a high level, the root cause of the issue lies in the architecture's design choices that prioritize availability and resilience over cost optimization. Let’s break down each component and then propose a redesign strategy.

#### Triage

The question probes understanding of distributed systems, cost management, and trade-offs between different architectural decisions. The assumptions are:

The system is built with high availability as a primary goal.
Cross-AZ data transfer costs are unexpectedly high.
AZ-failure resilience must be maintained.

#### First-Principles Solution

Chatty Services Split Across AZs:

Design Decision: To ensure high availability, services were split across multiple Availability Zones (AZs).
Impact on Costs: Chatty services increase the frequency of cross-AZ data transfers, leading to higher transfer costs.
Optimization Strategy: Aggregate chatty services into fewer, more robust services within each AZ or use a microservices approach that minimizes inter-service communications.

Replication Topology:

Design Decision: A replication topology was chosen to ensure data availability and consistency across multiple AZs.
Impact on Costs: Replication introduces additional write operations and cross-AZ transfers, which can be costly.
Optimization Strategy: Optimize the replication strategy by reducing unnecessary writes or using less frequent but still resilient replication mechanisms.

Misplaced Caches:

Design Decision: Inefficiencies in cache placement led to repeated data fetches from remote AZs.
Impact on Costs: Cache misses increase cross-AZ transfers, driving up costs.
Optimization Strategy: Re-evaluate cache policies and placements to minimize cache misses and leverage local data access.
Redesign for AZ-Affinity

To redesign the architecture for AZ-affinity while maintaining resilience:

Service Aggregation:

Aggregate chatty services into fewer, more robust services within each AZ.

#### Example
If a service is frequently communicating with multiple other services, consider consolidating these interactions to reduce cross-AZ data transfer.

Replication Optimization:

Use multi-zone replication strategies like cross-region read replicas or warm standby instances that minimize unnecessary writes.
Optimize the write propagation strategy by using batched writes and asynchronous replication.

Cache Placement:

Re-evaluate cache policies to ensure data is cached locally where it’s frequently accessed.
Use local caching mechanisms such as in-memory caches within each AZ or distributed caches that span multiple services but are closer to their consumers.

#### Closing Summary

By revisiting the design decisions and optimizing for cost, we can achieve a more balanced architecture. The key steps involve aggregating chatty services, optimizing replication strategies, and ensuring efficient cache placement. These changes will help reduce cross-AZ data transfer costs while maintaining the required resilience.

#### Common Mistakes/Follow-Ups
Overlooking Service Aggregation: Failing to aggregate chatty services can lead to continued high cross-AZ transfer rates.
Under-Optimizing Replication: Not leveraging advanced replication strategies that reduce unnecessary writes and cross-AZ transfers.
Poor Cache Placement: Ignoring the impact of cache placement on data access patterns, leading to inefficient use of local storage.

**Comparison:**

```
AZ	Services	Replicas
AZ1	Service1, 4	R1
AZ2	Service2	R2
Cache Placement	Data Access
Local Cache in AZ1	80%
Remote Cache in AZ2	20%
```

**Code:**

```python
# Example code for optimizing service interactions
def aggregate_services():
    # Aggregate services based on interaction frequency
    aggregated_service = {
        "service1": [service2, service3],
        "service4": [service5]
    }
    return aggregated_service

def optimize_replication_strategy(replica_set):
    # Optimize write propagation and use batched writes
    for replica in replica_set:
        if is_write_batch_needed():
            perform_batch_write(replica)
```

**Example input / output:**

| Service | Interactions with Zone 1 (AZ) |
|---------|-------------------------------|
| Service1| 200                           |
| Service4| 150                           |
| Replica Set | Write Batch Needed? |
|-------------|---------------------|
| Zone1       | No                  |
| Zone2       | Yes                 |

**Data:**

```json
{
  "service_interactions": [
    {"service1": [200, 300], "service4": [150]},
    {"service2": [250]}
  ],
  "replica_set": [{"zone1": true, "zone2": false}, {"zone1": false, "zone2": true}]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Aggregate Services
8. Are services chatty?
9. Reduce Cross-AZ Transfers
10. Keep as is
11. Optimize Replication Strategy
12. Use Local Caches?
13. Minimize Remote Fetches
14. Use Asynchronous Replication?
15. Reduce Cross-AZ Transfers
16. Keep as is

