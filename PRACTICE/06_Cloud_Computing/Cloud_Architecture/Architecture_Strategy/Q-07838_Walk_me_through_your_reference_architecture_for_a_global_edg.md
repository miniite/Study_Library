---
type: "practice"
difficulty: "Expert"
---

# Walk me through your reference architecture for a global edge-compute strategy: what runs at edge, what stays in region, and how do you keep them coherent?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Compute Placement Strategy](<../../../../06_Cloud_Computing/Cloud_Architecture/Architecture_Strategy/Compute_Placement_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a global edge-compute strategy, the goal is to balance latency with cost and reliability while ensuring coherence between edge and central cloud components. The architecture involves deploying compute resources at various layers—edge nodes, regional data centers, and centralized clouds—to serve diverse workloads efficiently.

The key triage of this question revolves around understanding your approach to managing the trade-offs in a multi-layered deployment model. Specifically, we'll discuss what runs at edge, what stays in region, how data flows between these layers, and measures to ensure coherence across different tiers.

TL;DR

This architecture deploys compute-intensive workloads closer to end-users for low latency, while keeping stateful or heavy processing tasks in regions for optimal resource utilization. We use a combination of edge gateways, edge computing nodes, and centralized cloud services to manage state synchronization and data flow efficiently.

#### Triaging the Question

The interviewer is probing your understanding of:

Workload Classification: Differentiating between compute-intensive, stateful, and stateless tasks.
Resource Utilization Trade-offs: Balancing resource allocation for edge vs. region.
Data Flow Management: Ensuring data consistency across edge and central cloud layers.
Fault Tolerance and Redundancy: Designing a robust architecture that can handle failures gracefully.

#### Assumptions
The application has varying levels of latency sensitivity.
There is a need for both real-time processing (edge) and batch processing (region).
Data security, compliance, and regulation are critical factors.
Step-by-Step Solution
Edge Layer

The edge layer typically includes:

Edge Gateways: Physical or virtual devices deployed at the network edge to manage local traffic.
Microservices: Lightweight services handling real-time data processing and low-latency requirements.
Stateless Workloads: Short-lived tasks that don't require persistent state.

#### Example
Use Case: Real-time video streaming, sensor data processing for IoT applications.

Regional Layer

The regional layer involves:

Regional Data Centers: Centralized infrastructure with more powerful compute resources for heavy lifting and data storage.
Batch Processing Services: Tasks requiring significant computational power or long-term data analysis.
Stateful Workloads: Long-running processes that require persistent state management.

#### Example
Use Case: Analyzing large datasets, training machine learning models, storing user profiles and preferences.

Centralized Cloud Layer

The centralized cloud layer focuses on:

Centralized Storage and Backup: Secure, scalable storage for critical data.
Global Synchronization Services: Ensuring coherence across edge and region via synchronization protocols.
Monitoring and Logging: Real-time monitoring of system health and performance metrics.

#### Example
Use Case: Global database replication, backup and recovery strategies, centralized logging for security audits.

Data Flow Management

To ensure data consistency:

Event-Driven Architecture (EDA): Use messaging queues like Kafka to facilitate real-time data exchange between edge and region.
API Gateway: Centralize API management at the regional layer to control access and enforce policies.

#### Cons
istency Models: Implement strong or eventual consistency models based on application requirements.
Coherence Strategy
State Synchronization: Use replication strategies like Active-Active, Active-Passive, or Shadowing for state synchronization.
Data Versioning: Maintain versioned data to handle conflicts and ensure backward compatibility.
Conflict Resolution Algorithms: Implement algorithms that resolve conflicts efficiently.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the architecture balances edge compute with regional and centralized cloud services while ensuring data consistency and coherence through robust data flow management and synchronization strategies.

#### Common mistakes
include:

Over-centralizing stateful workloads, leading to higher latency.
Underestimating the complexity of data flow management, resulting in inconsistent states.
Neglecting security and compliance across layers.

#### Follow-ups
might involve discussing specific technologies used (e.g., Kubernetes for edge orchestration), failure scenarios, or scaling considerations.

**Comparison:**

```
Layer	Component	Description
Edge	Edge Gateway	Manages local traffic and edge services
	Microservices	Lightweight, real-time processing
	Stateless Workloads	Short-lived tasks
Region	Regional Data Center	Centralized infrastructure for heavy processing
	Batch Processing	Tasks requiring significant computational power
	Stateful Workloads	Long-running processes with persistent state
Cloud	Centralized Storage	Secure, scalable storage for critical data
	Global Sync	Ensures coherence across edge and region
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Edge Gateway] --> B[Microservices];
    B --> C[Stateless Workloads];
    D[Regional Data Center] --> E[Batch Processing Services];
    E --> F[Stateful Workloads];
    G[Centralized Cloud Layer] --> H[Centralized Storage and Backup];
    H --> I[Global Synchronization Services];
    A --> J[Real-time Video Streaming];
    D --> K[Analyzing Large Datasets];
    G --> L[Global Database Replication];
```

**Example input / output:**

```text
Input:
- User request for real-time video streaming

Output:
1. Edge Gateway receives request and processes it locally.
2. Edge Gateway sends event to Kafka queue.
3. Regional Data Center consumes the event, performs processing.
4. Result is sent back to Edge via API Gateway.
5. Edge forwards result to user.
```

**Data:**

```json
{
  "layers": [
    {
      "name": "Edge",
      "components": ["Edge Gateway", "Microservices", "Stateless Workloads"]
    },
    {
      "name": "Region",
      "components": ["Regional Data Center", "Batch Processing Services", "Stateful Workloads"]
    },
    {
      "name": "Cloud",
      "components": ["Centralized Storage and Backup", "Global Synchronization Services"]
    }
  ],
  "dataFlow": [
    {
      "source": "Edge Gateway",
      "destination": "Regional Data Center",
      "type": "Event-Driven (EDA)"
    },
    {
      "source": "Regional Data Center",
      "destination": "Centralized Cloud Layer",
      "type": "API Gateway"
    }
  ]
}
```

**Flow:**

1. User Request
2. Edge Gateway
3. Microservices
4. Stateless Workloads
5. Local Processing
6. Kafka Queue
7. Regional Data Center
8. Batch Processing
9. Data Processing
10. Result
11. API Gateway
12. Edge Gateway
13. User

