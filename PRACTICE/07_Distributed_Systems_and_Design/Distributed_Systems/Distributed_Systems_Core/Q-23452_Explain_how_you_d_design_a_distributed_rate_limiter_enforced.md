---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd design a distributed rate limiter enforced at 50 API gateway nodes using a shared token store versus local buckets with reconciliation. Which do you pick for a hard billing quota and why?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Rate Limiting](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Rate_Limiting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a distributed rate limiter in a scenario where we need to enforce a 50 API gateway nodes' rate limit, we have two primary approaches: using a shared token store or local buckets with periodic reconciliation. The choice between these depends on the specific requirements and constraints of the system, such as consistency, performance, availability, and scalability.

At a high level, the shared token store approach ensures global coherence but may introduce complexity in managing state across nodes, while the local bucket approach is simpler to implement and scales better within each node but risks eventual consistency issues. Let's break down both approaches from first principles.

**Comparison:**

```
Local Bucket	Shared Token Store
Simplicity, no cross-node state management	High consistency across all nodes
Potential for hotspots and bottlenecks at individual nodes	Lower performance impact on a single node
Scalability issues due to increased network latency	Easier to scale horizontally by adding more nodes
Risk of eventual inconsistency in quota enforcement	Ensures global fairness and quota enforcement
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[API Gateway Node] --> B(Decrement Local Bucket);
    B --> C{Is Token Available?};
    C -- Yes --> D[Allow Request];
    C -- No --> E[Reject Request];
    F[Token Store] --> G(Reconcile State Periodically);
```

**Example input / output:**

```

Given an API request, a node will first decrement its local bucket. If the local bucket still has tokens, the request is allowed; otherwise, it's rejected.

PYTHON
Copy
def handle_request(node_id):
    # Local bucket for node_id
    if node_id in local_buckets and local_buckets[node_id] > 0:
        local_buckets[node_id] -= 1
        return "Allow Request"
    else:
        return "Reject Request"
```

**Data:**

```yaml
# Example token store schema
tokens:
  - id: 1234567890abcdef
    quota: 50
    remaining: 49
```

**Flow:**

1. Yes
2. No
3. Local Bucket Decrement
4. Token Available?
5. Allow Request
6. Reject Request
7. Periodic Reconciliation
8. Global State Update

