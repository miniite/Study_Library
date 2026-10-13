---
type: "practice"
difficulty: "Expert"
---

# Design a graceful-degradation and dependency-tiering scheme for a fintech app where some flows (view balance) can serve stale/degraded data and others (transfer money) must fail closed. How do you encode this criticality in the architecture so degradation is deliberate rather than accidental?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Static Stability](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Static_Stability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a graceful-degradation and dependency-tiering scheme for a fintech app, where some flows can serve stale data and others must fail closed, we need to establish a clear separation between critical (high-availability) and non-critical operations, while ensuring that the system gracefully degrades in less critical paths without affecting critical functionality. This involves careful design of our architecture, service dependencies, and data consistency models.

First, let’s triage what the question is probing: the interviewer is looking for a method to balance availability with correctness, ensuring that some parts of the application can tolerate degradation while others cannot. The key ideas are encoding criticality in the system, implementing fail-fast mechanisms where necessary, and using robust state management techniques.

**Code:**

```python
# Example of a service definition with criticality encoded in metadata
class ServiceDefinition:
    def __init__(self, name, critical=False):
        self.name = name
        self.critical = critical

service_definitions = [
    ServiceDefinition("UserBalanceService", critical=True),
    ServiceDefinition("TransactionHistoryService"),
    ServiceDefinition("AccountService", critical=True)
]
```

**Example input / output:**

```text
Request:
- User requests account balance (via UserBalanceService)

Response:
- If the service is available, returns up-to-date balance.
- If degraded (e.g., network issues), returns last known good state or default value.

Request:
- User attempts to transfer money (via AccountService and TransactionHistoryService)

Response:
- Both services must be fully functional for the transaction to proceed.
- Any failure results in a rollback and an error message.
```

**Data:**

```json
{
  "services": [
    { "name": "UserBalanceService", "critical": true },
    { "name": "TransactionHistoryService", "critical": false },
    { "name": "AccountService", "critical": true }
  ]
}
```

