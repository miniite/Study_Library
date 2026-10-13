---
type: "practice"
difficulty: "Expert"
---

# A tradeoff point sits between a shared multi-tenant telemetry store with per-team quotas (cheap, dense) and per-team isolated stores (blast-radius containment, no noisy-neighbor cardinality contention). How do you scenario-test both against a data-isolation security scenario and an ingestion-contention scenario?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Quality Attribute Trade-off Scenarios](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Quality_Attribute_Trade-off_Scenarios.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When evaluating architectural trade-offs between a shared multi-tenant telemetry store with per-team quotas versus per-team isolated stores, we must consider key dimensions such as data isolation, ingestion contention, cost-effectiveness, and disaster resilience. ATAM involves breaking down these scenarios into manageable parts to understand the trade-offs clearly.

Firstly, let's define the two architectural approaches:

Shared Multi-Tenant Telemetry Store with Per-Team Quotas: This model is cheaper and can handle high densities of telemetry data efficiently due to shared infrastructure resources.
Per-Team Isolated Stores: Each team has its own isolated store, providing better security (blast-radius containment) and avoiding noisy-neighbor cardinality contention.

We need to test both approaches against two primary scenarios:

Data Isolation Security Scenario
Ingestion Contention Scenario

Now, let's dive into the first scenario.

**Comparison:**

```
Scenario	Shared Multi-Tenant Telemetry Store	Per-Team Isolated Stores
Data Compromise	High risk if one team's data is compromised.	Lower risk due to isolation of data stores.
Ingestion Contention	Quotas can lead to bottlenecks and delays in high-traffic times.	Each team manages their own ingestion, minimizing contention issues.
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Shared Multi-Tenant Telemetry Store] --> B[Security Risk - Data Compromise]
    C[Per-Team Isolated Stores] --> D[Security Strength - Blast-Radius Containment]

    B --> E[Minimized Exposure to Other Teams' Incidents]
    D --> F[Maximized Isolation and Protection Against External Threats]
```

**Example input / output:**

```text
For the shared multi-tenant telemetry store with per-team quotas:
- **Input**: Data ingestion request from `TeamX` of size 10GB.
- **Output**: Success if `TeamX` quota allows, otherwise failure and log.

For the per-team isolated stores:
- **Input**: Ingestion request from `TeamY`.
- **Output**: Success with no impact on other teams' data, as each team's store is isolated.
```

**Data:**

```json
{
  "shared_telemetry": {
    "teams": ["TeamA", "TeamB", "TeamC"],
    "quotas": {"TeamA": 10GB, "TeamB": 5GB, "TeamC": 8GB},
    "total_capacity": 30GB
  },
  "isolated_telemetry": {
    "teams": ["TeamA", "TeamB", "TeamC"],
    "stores": {"TeamA": "Store1", "TeamB": "Store2", "TeamC": "Store3"}
  }
}
```

**Flow:**

1. Ingestion Request
2. Check Team Quota
3. Quota Allowed
4. Data Ingested to Shared Store
5. Quota Exceeded
6. Deny Ingestion, Log Error
7. Ingestion Request
8. Identify Correct Store
9. Store Capacity Available
10. Data Ingested to Isolated Store
11. Capacity Not Available
12. Deny Ingestion, Notify Team

