---
type: "practice"
difficulty: "Expert"
---

# Your gateway is the enforcement point for API monetization (usage plans, metering, billing); design the metering architecture so it's accurate enough to bill on, survives gateway restarts without double-counting or dropping usage, and reconciles with the billing system.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Gateway Cost Metering and Budgets](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Load_Balancing_and_Zone_Aware_Routing/Gateway_Cost_Metering_and_Budgets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an API Gateway metering architecture that accurately bills on usage, ensures no double-counting or dropped events during gateway restarts, and seamlessly integrates with a billing system, we need to focus on key components such as event sourcing, idempotency, and reconciliation mechanisms. Let's break down the solution from first principles.

Firstly, the core idea is to ensure that every API request is recorded exactly once in a way that survives gateway restarts. This involves using an event sourcing pattern where each request generates a unique event that gets stored persistently. The architecture should also handle idempotency gracefully by ensuring that any duplicate requests do not result in multiple billing entries.

To achieve this, we'll design the metering system to have a three-tiered approach: logging, storage, and reconciliation.

**Comparison:**

| Component       | Functionality                                 |
|-----------------|----------------------------------------------|
| Event Source    | Stores unique events representing API usage   |
| Event Processor | Ensures idempotency and processes requests    |
| Storage Layer   | Maintains state for event replay             |

**Code:**

```
MERMAID
Copy
graph TD;
    A[API Request] --> B{Is Event Idempotent?};
    B -- Yes --> C[Log & Store Event];
    B -- No --> D[Rate Limit & Retry Later];
    C --> E[Event Sourced Storage Layer];
    F[Gateway Restart] --> G[Replay Events from Storage];
    G --> H[Update Usage Metrics];
    I[Billing System] --> J[Fetch Updated Metrics];
```

**Example input / output:**

| Input | Output |
|-------|--------|
| API Request (with unique id) | Event Stored in Persistent Storage, Usage Metrics Updated, Billing System Updated with Corrected Metrics |

**Data:**

```json
{
  "event": {
    "type": "API_USAGE",
    "timestamp": "2023-10-05T14:48:00Z",
    "api_id": "abc123",
    "user_id": "user456"
  }
}
```

**Flow:**

1. Yes
2. API Request
3. Is Event Idempotent?
4. Log & Store Event
5. Event Sourced Storage Layer
6. Gateway Restart
7. Replay Events from Storage
8. Update Usage Metrics
9. Billing System
10. Fetch Updated Metrics

