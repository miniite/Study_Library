---
type: "practice"
difficulty: "Expert"
---

# A dependency graph analysis reveals that 60% of your services transitively depend on one internal auth service. As the principal, is that a problem or a feature? Argue both sides, and if you'd de-risk it, how do you reduce the shared-fate coupling without shattering into inconsistent per-service auth?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Federated vs Centralized Data Ownership](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Federated_vs_Centralized_Data_Ownership.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When evaluating whether 60% of services transitively depending on a single internal auth service is a problem or a feature, we must consider both the potential risks and benefits from first principles. On one hand, centralized authentication can simplify security management and enforce consistent policies across the entire system. On the other hand, this high degree of dependency introduces significant risk—namely, a single point of failure that could cascade failures through all dependent services.

The key is to weigh these factors against the cost of alternatives and determine if we need to de-risk the current architecture. If we decide to reduce shared-fate coupling without shattering into inconsistent per-service auth, we can explore strategies like introducing a more resilient authentication service or leveraging existing infrastructure components.

**Comparison:**

```
Strategy	Pros	Cons
Centralized Auth	Simplifies security management, enforces consistency	Single point of failure, high shared-fate coupling
Decentralized Auth	Resilience against single-point failures	Complexity in managing multiple auth mechanisms
```

**Code:**

```python
# Example Python function demonstrating how an auth service might be called
def authenticate_user(user_id):
    return True  # Simulate successful authentication

# Example of calling the auth service from another service
def process_request(user_id):
    if not authenticate_user(user_id):
        raise Exception("Authentication failed")
    # Process request logic here
```

**Example input / output:**

```text
Input:
- ServiceA makes a request to process_request(123)

Output:
- Authenticate_user(123) is called, and if it fails, an exception is raised

Dry-run:
ServiceA -> ServiceB -> Auth (if ServiceB needs auth)
```

**Data:**

```json
{
  "services": [
    {"name": "ServiceA", "dependencies": ["Auth"]},
    {"name": "ServiceB", "dependencies": ["Auth"]},
    {"name": "ServiceC", "dependencies": ["ServiceB", "Auth"]},
    // ...
  ]
}
```

**Flow:**

1. ServiceA
2. Auth
3. ServiceB

