---
type: "practice"
difficulty: "Deep"
---

# You must decide whether the gateway owns request idempotency state centrally or delegates it to each service; make the architectural case considering where the dedup store scales, what happens on gateway restart, and how you migrate services that currently trust the gateway to dedup.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Gateway Idempotency and Org Standards](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/Gateway_Idempotency_and_Org_Standards.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide whether the API Gateway should own request idempotency state centrally or delegate it to each service, we need to consider several factors, including where the deduplication store scales, what happens on gateway restarts, and how to migrate existing services that currently trust the gateway for deduplication. The core decision involves balancing consistency, availability, fault tolerance, and the complexity of distributed systems.

**Comparison:**

```
Gateway Ownership	Centralized State Management	Delegated State Management
Scalability	Requires a robust, scalable database or in-memory store.	Services must handle their own deduplication logic, which may be challenging if services are not designed for it.
Fault Tolerance	The gateway can continue functioning even if some services fail.	If the gateway fails, no service handles requests until it recovers.
Migration Complexity	Migrating existing services to a centralized system is complex and requires downtime.	Existing services may need to be updated or reconfigured, but this can often be done in stages without full downtime.
```

**Code:**

```python
# Example high-level API Gateway code snippet showing a central state management approach
class CentralDedupStore:
    def __init__(self):
        self.dedup_map = {}

    def is_request_idempotent(self, request_id):
        return self.dedup_map.get(request_id) is not None

    def mark_as_handled(self, request_id):
        self.dedup_map[request_id] = True
```

**Example input / output:**

```text
Example input and output for checking and marking a request as handled in the central dedup store:
- Input: `is_request_idempotent("abc123")` -> Output: `False` (request not seen before)
- Input: `mark_as_handled("abc123")` -> Output: N/A (state updated internally)
```

**Data:**

```json
{
  "requestId": "abc123",
  "status": "handled"
}
```

**Flow:**

1. Yes
2. No
3. Request
4. Is Request Idempotent?
5. Handle and Return
6. Forward to Service

