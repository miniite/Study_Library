---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd set the org's policy on where model-serving state lives, so stateless-by-default holds but the genuinely stateful pieces (KV caches, session context, feature caches) are consciously placed, replicated, and owned rather than scattered accidentally.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Stateless by Default](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Stateless_by_Default/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In setting the organization's policy on where model-serving state lives, we aim to achieve a balance between ensuring statelessness by default while still allowing for conscious placement of genuinely stateful components such as KV caches, session context, and feature caches. This requires a strategic approach that encompasses both technical considerations and organizational practices.

Firstly, it’s crucial to understand what "stateless-by-default" means in the context of model serving. It implies that every instance of a model server should be able to start from a fresh state, without relying on external data or stateful components to function correctly. However, there may be certain use cases where maintaining state is necessary and beneficial, such as caching session context for user interactions.

To manage this effectively, we need to establish clear guidelines and technical solutions that differentiate between these two scenarios. The policy should be both prescriptive and flexible enough to accommodate various use cases without compromising on the overall architecture's robustness and maintainability.

**Comparison:**

```
Component Type	Stateful Dependencies	Stateless-by-Default?
Model Server	None	Yes
User Session Cache	KVCache	No
```

**Code:**

```python
# Example of a simplified policy enforcement in Python

def enforce_stateless_by_default(policy):
    if not is_stateful_component(policy.component_type):
        return True  # Stateless component, allow it to be stateless-by-default
    else:
        return False  # Stateful components need explicit approval and placement

def is_stateful_component(component_type):
    return component_type in ['KVCache', 'SessionContext', 'FeatureCache']
```

**Example input / output:**

```text
# Example input and output of the policy enforcement

policy = {
    "components": [
        {"name": "model-server", "type": "stateless"},
        {"name": "user-session-cache", "type": "stateful"}
    ]
}

for component in policy["components"]:
    if enforce_stateless_by_default(component):
        print(f"{component['name']} is stateless-by-default.")
    else:
        print(f"{component['name']} requires explicit placement and replication.")
```

**Data:**

```json
{
  "components": [
    {
      "name": "model-server",
      "type": "stateless",
      "stateful_dependencies": []
    },
    {
      "name": "user-session-cache",
      "type": "stateful",
      "stateful_dependencies": ["KVCache"]
    }
  ]
}
```

**Flow:**

1. Stateless-by-Default
2. Not Stateless-by-Default
3. Approved
4. Rejected
5. Stateless Component
6. Enforced Policy
7. Allowed to Start Fresh
8. Explicit Approval Needed
9. Placed and ReplicatedExplicitly
10. Rejected, Must be Stateless

