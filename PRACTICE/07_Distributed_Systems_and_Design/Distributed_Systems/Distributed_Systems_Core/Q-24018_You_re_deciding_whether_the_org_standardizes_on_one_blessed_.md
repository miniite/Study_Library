---
type: "practice"
difficulty: "Expert"
---

# You're deciding whether the org standardizes on one blessed telemetry-agent (OTel Collector) everywhere or lets teams choose. Argue the paved-road-vs-autonomy trade-off for observability agents specifically (consistency of instrumentation, upgrade blast radius, operational load) and the governance you'd ship.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Federated vs Centralized Data Ownership](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Federated_vs_Centralized_Data_Ownership.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide whether an organization standardizes on one blessed telemetry-agent (OTel Collector) or allows teams to choose their own agents, we need to weigh the benefits of paved roads against those of autonomy in the context of observability agents specifically. The key trade-offs include consistency of instrumentation, upgrade blast radius, and operational load. Effective governance mechanisms must be established to support this decision.

First, let's understand what OTel Collector is: It’s an open-source telemetry aggregation system that standardizes data collection across multiple sources, protocols, and services. Its primary role is to unify telemetry from various sources into a single format, making it easier for teams to manage and analyze data.

**Comparison:**

```
Task1
Task2
Task3
Task4
Task5
Task6
Standardization Effort
Autonomous Agents Deployment
OTel Collector vs. Team Choice Implementation Timeline
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Decision] --> B1[(Standardize OTel Collector)]
    A --> B2[(Allow Teams Choice)]
    B1 --> C1[Consistency of Instrumentation]
    B1 --> C2[Upgrade Blast Radius]
    B1 --> C3[Operational Load]
    B2 --> D1[Flexibility & Innovation]
    B2 --> D2[Decentralized Governance]
    C1 --> E1[Ensures uniform data collection practices]
    C1 --> E2[Reduces learning curve for new team members]
    C2 --> F1[Minimizes impact on critical services during upgrades]
    C2 --> F2[Centralizes control over agent versions and configurations]
    C3 --> G1[Increases overhead due to multiple agents]
    C3 --> G2[Requires more human resources for management]
```

**Example input / output:**

| Team       | Agent Choice           | Pros                                         | Cons                              |
|------------|------------------------|----------------------------------------------|-----------------------------------|
| DevOps     | Jaeger Tracing Agent   | High traceability for complex applications    | Limited support for metrics & logs|
| Monitoring | Prometheus Exporter    | Rich monitoring capabilities                 | No full-fledged log collection     |
| Security   | OTel Collector         | Unified data model, easier governance        | Steeper learning curve            |

**Data:**

```yaml
# Example: Configuration Variations Across Teams
teams:
  - name: DevOpsTeam
    agent: jaeger-tracing-agent
  - name: MonitoringTeam
    agent: prometheus-exporter
  - name: SecurityTeam
    agent: otel-collector
```

**Flow:**

1. Start
2. Identify Business Needs
3. Assess Current ObservabilityLandscape
4. Determine StandardizationBenefits/Costs
5. Establish GovernanceFramework
6. Deploy OTel Collector or AllowChoice
7. Monitor and Adjust Policies

