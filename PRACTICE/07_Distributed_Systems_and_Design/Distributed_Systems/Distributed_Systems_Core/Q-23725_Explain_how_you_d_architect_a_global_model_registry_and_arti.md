---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd architect a global model registry and artifact-distribution system so that promoting a model to production is a linearizable operation, but the 30GB weights propagate to hundreds of serving pods across regions without a thundering herd or a mixed-version window.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Mixed Version Fleets](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a global model registry and artifact-distribution system that ensures promoting a model to production is linearizable while efficiently distributing large weights to hundreds of serving pods across regions, we need to address several key aspects: linearizability, distributed coordination, state consistency, scalability, and reliability. The core idea is to leverage distributed systems principles, such as consensus algorithms for coordination, state-machine replication for linearization, and a combination of gossip-based protocols and push-pull mechanisms for artifact distribution.

#### Triaging the Question

The question is fundamentally asking how we can design a system that maintains a single source of truth for model versions in a globally distributed environment. This involves ensuring that any update to a model version is processed in an order-agnostic manner across all regions, while simultaneously distributing large artifacts (like 30GB weights) efficiently and without causing bottlenecks or mixed-version windows.

#### Assumptions

#### Cons
istent Hashing: Each region will have one or more serving pods.
Replicated State: The model registry will maintain a replicated state across multiple regions to ensure high availability.
Linearizable Operations: Any change in the model version should be visible and consistent across all regions.

#### First-Principles Solution
1. Centralized vs. Decentralized Coordination

To achieve linearizability, we can use a centralized coordination service like Apache ZooKeeper or Etcd to manage the order of updates to the model registry. However, this introduces a single point of failure and potential network latency. Alternatively, we can use decentralized consensus protocols such as Raft or Paxos, which are more robust but require careful implementation.

2. State-Machine Replication

Once an update is applied to the centralized coordination service, it triggers state-machine replication across all regions. Each region will have a replicated model registry that applies changes atomically and in order. This ensures linearizability by guaranteeing that any sequence of operations appears to occur as if they are executed one after another.

3. Artifact Distribution

For the actual artifact distribution, we can use a combination of gossip-based protocols and push-pull mechanisms:

Gossip Protocol: To propagate metadata (like checksums) about new models and their artifacts.
Push-Pull Mechanism: To ensure that each serving pod pulls down the necessary artifacts from nearby regions.
4. Region-Specific Serving Pods

Each region will have a cluster of serving pods, with each pod responsible for serving requests locally. The serving pods can be configured to fetch their weights and other artifacts from replicated state in their local region or nearby regions.

Detailed Solution

Centralized Coordination Service:

Use Apache ZooKeeper or Etcd as the centralized coordination service.
Define a raft of servers for high availability.
Implement linearizable operations using these services to update the model registry.

Replicated Model Registry:

Replicate the state across multiple regions using state-machine replication techniques.
Ensure that any change in the model version is applied atomically and in order across all replicas.

Artifact Distribution:

Use a gossip protocol (e.g., Kademlia) to propagate metadata about new models and their artifacts.
Implement a push-pull mechanism where serving pods pull down necessary artifacts from nearby regions based on the metadata received via gossip.

Serving Pods Configuration:

Each region will have multiple serving pods configured with load balancers.
Use a consistent hashing algorithm to map model requests to specific serving pods.
Serve requests locally and fetch weights and other artifacts as needed.

#### Closing Summary

To ensure that promoting a model to production is linearizable while efficiently distributing large artifacts, we can leverage distributed consensus protocols for coordination, replicated state-machine replicas for consistency, gossip-based protocols for metadata distribution, and push-pull mechanisms for artifact propagation. This design ensures high availability, reliability, and efficient operation across multiple regions.

#### Common Mistakes/Follow-Ups
Single Point of Failure: Over-relying on a centralized coordination service can introduce single points of failure.
Mixed-Version Windows: Failing to handle mixed-version windows properly can lead to inconsistencies in serving requests.
Latency Considerations: Gossip protocols and network latency should be carefully considered, especially for large artifacts.

**Comparison:**

| Region | Serving Pods | Weights URL |
|--------|--------------|-------------|
| us-west-2 | p1, p2 | https://us-west-2.weights.example.com/model_weights_v1 |
| us-east-1 | q1 | https://us-east-1.weights.example.com/model_weights_v1 |

**Code:**

```
MERMAID
Copy
graph LR
    A[Central Coordination Service] --> B[Replicated State]
    B --> C{Region-Specific Serving Pods}
    C --> D1(Serving Pod 1)
    C --> D2(Serving Pod 2)
    D1 --> E1[Pull Artifact from Nearby Region]
    D2 --> E2[Pull Artifact from Nearby Region]
    A --> F[Gossip Protocol for Metadata]
```

**Example input / output:**

```text
Input: Promote model version 1.0 to production in the us-west-2 region.
Output:
1. Update model registry in central coordination service.
2. Replicate state across all regions using state-machine replication.
3. Gossip protocol propagates metadata about new model version.
4. Serving pods pull down necessary artifacts from nearby regions based on metadata.
5. All serving pods in us-west-2 region now serve requests with updated model weights.
```

**Data:**

```yaml
model_registry:
  regions:
    - region: us-west-2
      serving_pods:
        - pod_id: p1
          weights_url: https://us-west-2.weights.example.com/model_weights_v1
        - pod_id: p2
          weights_url: https://us-west-2.weights.example.com/model_weights_v1
    - region: us-east-1
      serving_pods:
        - pod_id: q1
          weights_url: https://us-east-1.weights.example.com/model_weights_v1
```

**Flow:**

1. Central Coordination Service
2. Replicated State
3. Region-Specific Serving Pods
4. Serving Pod 1
5. Serving Pod 2
6. Pull Artifact from NearbyRegion
7. Pull Artifact from NearbyRegion
8. Gossip Protocol for Metadata

