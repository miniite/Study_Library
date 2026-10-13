---
type: "practice"
difficulty: "Expert"
---

# You must design the org's incident-response for a system-of-systems where no single team understands the whole graph and the AIOps tool must page the right owner for a correlated multi-service incident. Explain the blast-radius-aware routing and ownership-mapping design that avoids paging everyone.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-tenant vs Dedicated Clusters](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Multi-tenant_vs_Dedicated_Clusters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an effective incident-response system for a system-of-systems where no single team understands the whole graph, we need to ensure that the AIOps tool can page the right owner for a correlated multi-service incident without paging everyone unnecessarily. The key concepts are blast-radius-aware routing and ownership-mapping design. We will start with a high-level overview and then dive into the details of how this can be implemented.

TL;DR

The solution involves creating an ownership graph that maps services to their responsible teams, understanding the blast radius of each service, and using these insights to determine which team should be paged for correlated incidents. This approach ensures efficient incident response by minimizing unnecessary paging while ensuring coverage across all relevant services.

#### Triaging the Question

The question is asking us to design a routing system that can efficiently handle multi-service incidents in a distributed system. The key points are:

Ownership Mapping: Identifying which team owns each service.
Blast Radius Awareness: Understanding how an incident might affect other services.
Efficient Paging: Ensuring the right team is paged to minimize disruption.

#### First-Principles Solution
Ownership Mapping
Service-Owning Teams: Create a graph where nodes represent services and edges represent ownership relationships between teams.
Correlation Engine: Implement an engine that can correlate incidents across different services based on predefined rules or heuristics (e.g., shared resources, service dependencies).
Blast Radius Awareness
Impact Assessment: Define a method to assess the potential impact of an incident in each service.
Blast Radius Graph: Build a graph where nodes represent services and edges indicate the likelihood of one service impacting another.
Efficient Paging Mechanism
Paging Rules Based on Ownership and Impact: Use the ownership mapping and blast radius assessment to determine which teams should be paged for a correlated incident.
Dynamic Routing Tables: Maintain dynamic routing tables that store the necessary information about ownership and impact, allowing quick lookups during an incident.

#### Example
Workflow
Incident Detection: An incident is detected in Service A.
Correlation Analysis: The correlation engine determines if this incident is correlated with incidents in other services (Service B and Service C).
Ownership and Impact Assessment: Based on the ownership graph, it identifies that Team X owns Service A, and based on the blast radius graph, it assesses that Services B and C might be affected.
Paging Decision: The system decides to page Team X for Service A and Teams Y and Z (owners of Services B and C) only if their services are significantly impacted.

#### Closing Summary

The design involves creating a comprehensive ownership mapping and blast radius assessment mechanism to ensure that the right teams are paged for correlated incidents. This approach minimizes unnecessary paging while ensuring all relevant services are covered.

#### Common Mistakes/Follow-Ups
Mistake: Failing to consider edge cases where multiple service impacts overlap, leading to redundant paging.

#### Follow-Up
How would you handle scenarios where the ownership mapping changes dynamically due to team restructuring?

**Comparison:**

| Service | Owning Team | Blast Radius |
|---------|-------------|--------------|
| ServiceA | TeamX       | High         |
| ServiceB | TeamY       | Low          |
| ServiceC | TeamZ       | Low          |

**Code:**

```python
# Example code snippet for dynamic routing table update

class RoutingTable:
    def __init__(self):
        self.ownership_graph = {}
        self.blast_radius_graph = {}

    def add_service(self, service_name, owning_team, impact_level):
        # Add service to ownership graph and blast radius graph
        pass

    def correlate_incidents(self, incident_details):
        # Correlate incidents based on predefined rules
        pass

    def decide_paging(self, incident_service, correlated_services):
        # Determine which teams should be paged
        pass
```

**Example input / output:**

```text
Incident detected in Service A
Correlation Engine:
  - Correlated with Service B and C based on shared resources
Ownership Mapping:
  - Team X owns Service A, Teams Y and Z own Services B and C respectively
Blast Radius Assessment:
  - ImpactLevel1: No significant impact
  - ImpactLevel2: Significant impact in both services

Paging Decision:
  - Page Team X for Service A
  - Page Teams Y and Z only if their services are significantly impacted (ImpactLevel2)
```

**Data:**

```json
{
  "ownership_graph": {
    "serviceA": ["teamX"],
    "serviceB": ["teamY"],
    "serviceC": ["teamZ"]
  },
  "blast_radius_graph": {
    "serviceA": {"impactLevel1": ["serviceB"], "impactLevel2": ["serviceC"]},
    "serviceB": {"impactLevel1": [], "impactLevel2": []},
    "serviceC": {"impactLevel1": [], "impactLevel2": []}
  }
}
```

**Flow:**

1. Yes
2. No
3. Incident in Service A
4. Correlation Engine
5. Correlate with Services
6. Determine Ownership andImpact
7. Page TeamX for ServiceA
8. (Evaluate) Page TeamsY and Z
9. Impact Level > Threshold?
10. Page Team Y/Z
11. No Additional Paging

